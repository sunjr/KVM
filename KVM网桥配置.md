## KVM的虚拟机通过网桥使用外网

#### 1、配置网桥
* 在/etc/sysconfig/network-scripts新建ifcfg-br0，内容如下
```
TYPE=Bridge                     //表示这是一个网桥设备
BOOTPROTO=static                //静态获取IP
DEVICE=br0                      //网桥的名称
ONBOOT=yes                      //是否开机启动
IPADDR0=192.168.30.128          //IP地址
PREFIX0=24                      //子网掩码
GATEWAY0=192.168.30.2           //网关
DNS1=192.168.30.2               //dns
```
* 修改ifcfg-eno16777736内容（原来网卡的配置文件）
```
TYPE=Ethernet               //设备的类型
BOOTPROTO=dhcp              //使用dhcp获取IP
DEFROUTE=yes                
PEERDNS=yes                 
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
NAME=eno16777736            //名称
DEVICE=eno16777736          //名称
ONBOOT=yes                  //开始启动
BRIDGE=br0                  //桥接网卡
```
* 重启network服务
```
[root@192 network-scripts]# systemctl restart network       //重启network
[root@192 network-scripts]# brctl show                      //查看网桥信息
bridge name	bridge id		STP enabled	interfaces
br0		8000.000c29953153	no		eno16777736
virbr0		8000.5254007957fd	yes		virbr0-nic
```
#### 2、创建KVM虚拟机
* 创建虚拟磁盘
```
[root@192 kvm]# qemu-img create -f qcow2 centos-6.5.qcow2  10G
```
* 创建虚拟机(注意network参数)
```
virt-install --virt-type kvm --name centos-6.5 --ram 1024 \
--cdrom=CentOS-6.5-x86_64-minimal.iso \
--disk centos-6.5.qcow2 \
--network bridge=br0 \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--os-type=linux --os-variant=rhel6
```


