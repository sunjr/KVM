#### 1、切换yum源
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```

#### 2、安装net-tools
```
yum install net-tools -y  //centos7默认不支持ifconfig，可以用ip命令替代
```


#### 3、查看是否支持虚拟化
```
    egrep -c '(vmx|svm)' /proc/cpuinfo   //命令结果大于0表示cpu支持虚拟化，如果不支持虚拟化，后面的操作无法进行
```
> 在vmware虚拟机中可以通过CPU设置中选择：虚拟化 Intel VT -x****来开始虚拟机的虚拟化

#### 4、安装KVM
```
yum install qemu-kvm -y        //安装KVM
yum install virt-manager libvirt libvirt-python virt-install bridge-utils -y     //安装虚拟化管理工具
```
> virt-manager：KVM的图形化管理界面
>
> libvirt: 是管理虚拟机和其他虚拟化功能，比如存储管理，网络管理的软件集合，它包括一个API库，一个守护程序（libvirtd）和一个命令行工具（virsh）
>
> libvirt-python：对libvirt进行封装，生成的Python接口
>
> virt-install：一个python写的脚本,是qemu-kvm工具的人性化实现，可以利用该工具在终端下创建KVM guest主机
>
> bridge-utils：Linux的网桥工具，KVM创建的虚拟机要获取外网IP的时候需要使用

#### 5、查看是否安装成功
```
[root@192 ~]# lsmod | grep kvm
kvm_intel             162153  0 
kvm                   525259  1 kvm_intel
//如若有上面的那些输出，表示安装成功
如果没有成功、则执行下面的命令刷新模块
modprobe kvm
modprobe kvm-intel
```
#### 6、关闭防火墙，libvirtd,设置libvirtd开机启动
```
systemctl stop firewalld.service        //停止firewall
systemctl disable firewalld.service     //禁止firewall开机启动
systemctl start libvirtd                //启动libvirtd
systemctl enable libvirtd               //设置libvirtd开机启动
```
#### 7、创建一个虚拟机安装盘
```
[root@192 ~]# mkdir /kvm
[root@192 ~]# cd /kvm
[root@192 kvm]# qemu-img create -f qcow2 centos-6.5.qcow2  10G
Formatting 'centos-6.5.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 lazy_refcounts=off 
[root@192 kvm]# chmod a+rw centos-6.5.qcow2     //修改文件权限
```
> 这里进行目录相关操作是因为我用root用户登录的，创建虚拟机的时候会出现权限问题
>

#### 8、创建一个虚拟机
```
virt-install --virt-type kvm --name centos-6.5 --ram 1024 \
--cdrom=CentOS-6.5-x86_64-minimal.iso \
--disk centos-6.5.qcow2 \
--network network=default \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--os-type=linux --os-variant=rhel6
```

#### 9、查看虚拟机状态
```
[root@192 kvm]# virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # list
 Id    Name                           State
----------------------------------------------------
 7     centos-6.6                     running
 
```

#### 10、使用vnc连接虚拟机
    * 因为只有一台KVM虚拟机，因此输入宿主机器IP即可连接到新创建的虚拟机