> help 打印帮助信息
>
> list 列出当前运行中的虚拟机 list --all列出所有的虚拟机
>
> shutdown 关闭指定的虚拟机(虚拟机必须安装acpid程序)，否则无法关闭虚拟机，shutdown后面可以是虚拟机的ID或者名称
>
> start 启动指定的虚拟机 start $vm_name
>
> destroy 强制关闭虚拟机电源
>
> reboot 重启虚拟机
>
> undefine 移除虚拟机但不删除，删除虚拟机需要在进行此操作之后手动删除虚拟机文件
>
> quit/exit 退出命令
>
> net-dumpxml default 查看默认网络
>
> domiflist domain(vm_name) 列出虚拟机的所有网口,例如:domiflist centos-6.5
>
> domblklist domin(vm_name) 列出虚拟机所有的块设备,例如:domblklist centos-6.5