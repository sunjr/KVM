#### 1、根据模板文件修改
    * 修改虚拟名称
    * 删除uuid
    * 指定disk
    * 删除mac
```
[root@192 libvirt]# cd /etc/libvirt/qemu            //被克隆的虚拟机的配置文件都在这里
[root@192 qemu]# ls
centos-6.5.xml  networks
[root@192 qemu]# cp centos-6.5.xml centos-6.5_copy.xml  //拷贝配置文件
。。。。。。进行上述操作
```
- 修改前文件内容
```
...
<name>centos-6.5</name>
...
<uuid>5fb1a625-0cfe-46cd-9ade-4e06ebdf3784</uuid>
...
<source file='/kvm/centos-6.5.qcow2'/>
...
<mac address='52:54:00:32:04:4a'/>
...
<source mode='bind' path='/var/lib/libvirt/qemu/channel/target/domain-centos-6.5/org.qemu.guest_agent.0'/>
...
```
- 修改后文件内容
```
...
<name>centos-6.5_copy</name>
...
<!-- <uuid>5fb1a625-0cfe-46cd-9ade-4e06ebdf3784</uuid> -->
...
<source file='/kvm/centos-6.5_copy.qcow2'/>
...
<!-- <mac address='52:54:00:32:04:4a'/> -->
...
<source mode='bind' path='/var/lib/libvirt/qemu/channel/target/domain-centos-6.5_copy/org.qemu.guest_agent.0'/>
...
```

#### 2、复制qcow2文件
    * 注意：
        - 删除被拷贝虚拟机的/etc/sysconfig/network-scripts/ifcfg-eth0中的HWADEE和UUID两行
        - 删除/etc/udev/rule.d/70-persistent-net.rules文件
    * 如果不删除会导致克隆过来的虚拟机网卡无法正常启动
```
[root@192 qemu]# cd /kvm/
[root@192 kvm]# cp centos-6.5.qcow2 centos-6.5_copy.qcow2           //复制qcow2文件
[root@192 kvm]# mv /etc/libvirt/qemu/centos-6.5_copy.xml ./         //把配置文件拷贝到当前目录
[root@192 kvm]# ls          
centos-6.5_copy.qcow2  centos-6.5_copy.xml  centos-6.5.qcow2  CentOS-6.5-x86_64-minimal.iso
```
#### 3、执行克隆命令
```
[root@192 kvm]# virsh define centos-6.5_copy.xml 
Domain centos-6.5_copy defined from centos-6.5_copy.xml
```
#### 4、查看克隆结果
```
[root@192 qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 12    centos-6.5                     running
 -     centos-6.5_copy                shut off
```
#### 5、启动虚拟机
```
virsh start centos-6.5_copy
```
#### 6、vnc连接虚拟机
```
[root@192 kvm]# netstat -aptn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      14840/qemu-kvm      
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN      15253/qemu-kvm      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      2744/rpcbind        
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      12507/dnsmasq       
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1123/sshd           
tcp        0      0 192.168.30.128:22       192.168.30.1:50225      ESTABLISHED 2233/sshd: root@pts 
tcp        0      0 192.168.30.128:5900     192.168.30.1:51325      ESTABLISHED 14840/qemu-kvm      
tcp        0      0 192.168.30.128:22       192.168.30.1:58006      ESTABLISHED 13208/sshd: root@pt 
tcp        0     52 192.168.30.128:22       192.168.30.1:54747      ESTABLISHED 14453/sshd: root@pt 
tcp        0      0 192.168.30.128:22       192.168.30.1:58173      ESTABLISHED 12580/sshd: root@no 
tcp        0      0 192.168.30.128:22       192.168.30.1:53799      ESTABLISHED 15143/sshd: root@pt 
tcp6       0      0 :::111                  :::*                    LISTEN      2744/rpcbind        
tcp6       0      0 :::22                   :::*                    LISTEN      1123/sshd   
```
> 由上面的端口占用可以看出：两个kvm的进程分别占用了5900、5901两个端口，因此连接第一台虚拟机可以指定端口5900(默认端口，可以省略)，第二台则需要指定为5901