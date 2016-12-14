#### Spark编程之三：穿透linux,实现Window与linux快速文件共享

-----------------------------------------

##### 背景说明

通过VM VitualBox共享文件功能，实现linux和window文件共享，从而实现windows开发Spark应用快速部署到Spark集群。


##### 软件要求

升级VM VitualBox到V5.10版本[^VitualBox]

##### Vitualbox 共享文件夹功能配置

- 挂载　VBoxGuestAdditions.iso[^ISO]　到虚拟光驱，如下图示

　![installAdditons](https://raw.githubusercontent.com/huangbaixun/imgs/master/3.1.1.png)
　
- 配置共享文件夹vmshare[^vmshare]，如下图示

　![installAdditons](https://raw.githubusercontent.com/huangbaixun/imgs/master/3.1.2.png)
　
- 启动Node1
- 挂载iso
```shell
   sudo mkdir /media/cdrom
   sudo mount /dev/cdrom /media/cdrom
```
- 安装依赖组件

```shell
  yum update
  #yum remove kernel-headers
  yum install gcc kernel-devel make
  yum install bzip2
```


- 安装VBox插件,并重启，注：Could not find the X.Org or XFree86 Window System错误不影响使用

```shell
# sudo /media/cdrom/VBoxLinuxAdditions.run 
Verifying archive integrity... All good.
Uncompressing VirtualBox 5.1.10 Guest Additions for Linux...........
VirtualBox Guest Additions installer
Removing installed version 5.1.10 of VirtualBox Guest Additions...
Copying additional installer modules ...
Installing additional modules ...
vboxadd.sh: Building Guest Additions kernel modules.
vboxadd.sh: You should restart your guest to make sure the new modules are actually used.
vboxadd.sh: Starting the VirtualBox Guest Additions.

Could not find the X.Org or XFree86 Window System, skipping.
# reboot 
```

- 挂载windows共享文件夹
```shell
mkdir /mnt/vmshare
mount -t vboxsf vmshare /mnt/vmshare/
```
- 测试windows向linux同步文件，在window环境中向vmshare放入一个测试i文件readme.txt,并在linux 下验证
```shell
ls /mnt/vmshare/
readme.txt
```

#####  参考资料

- [shared_folder_centos_virtualbox.txt](https://gist.github.com/larsar/1687725)
- [VirtualBox Shared Folders with CentOS Server Guest -dep](http://toolkt.com/site/virtualbox-shared-folders-with-centos-server-guest/)

[^VitualBox]: 之前的分布式集群安装指南使用的是V5.1.4版本
[^ISO]: 默认路径：C:\Program Files\Oracle\VirtualBox\VBoxGuestAdditions.iso
[^vmshare]: 在D盘创建一个空的文件夹vmshare