---
title: virtual box
date: 2016-05-27 23:56:43
tags:
- 虚拟机
- virtual box
categories:
- 安装
---
目的：实现主机访问虚拟机，虚拟机可访问网络
环境：电信无线网络（未链接网线）
安装虚拟机时，自动添加的网卡（VirtualBox Host-Only Ethernet Adapter）：
![vbox网络连接](http://justintung.github.com/images/2016.5.28.vm1.png)
Virtual Box中对应虚拟机的网卡配置（2张网卡）：
![虚拟机网卡1](http://justintung.github.com/images/2016.5.28.vm2.png)
![虚拟机网卡2](http://justintung.github.com/images/2016.5.28.vm3.png)
共享文件夹配置：
![共享文件夹设置](http://justintung.github.com/images/2016.5.28.vm4.png)
虚拟机内查看所有网卡：
![虚拟机所有网卡](http://justintung.github.com/images/2016.5.28.vm5.png)
查看已经使用的网卡：
ifconfig
(如果发现eth1、eth2不存在，则使用`ifconfig eth1 up`来启用)
查看ifcfg-eth1、ifcfg-eth2的配置
切换目录：
![虚拟机网卡目录](http://justintung.github.com/images/2016.5.28.vm6.png)
ifcfg-eth1配置：
![网卡eth1](http://justintung.github.com/images/2016.5.28.vm7.png)
ifcfg-eth2配置：
![网卡eth2](http://justintung.github.com/images/2016.5.28.vm8.png)
重启网络：
```shell
service network restart
```
网络地址转换NAT 对应dhcp 网络配置，仅主机（Host-Only）适配器对应 static 网络配置

clone 一个虚拟机并重置网卡时：`ifconfig eth0 up` 会出现“eth0 unknow”错误，解决方法是：
将 /etc/udev/rules.d/70-persistent-net.rules 清空后重启
设置共享目录：
必须安装 VboxGuestAdditions
a.下载：
```shell
wget http://download.virtualbox.org/virtualbox/5.0.12/VBoxGuestAdditions_5.0.12.iso
```
b.virtual box中添加虚拟光盘
![虚拟介质管理器](http://justintung.github.com/images/2016.5.28.vm9.png)
c.挂载光驱：
在/etc/fstab中添加一行：
```shell
/dev/cdrom /mnt/cdrom auto users,exec,noauto,managed 0 0
```
执行挂载：
```shell
mount /dev/cdrom /mnt/cdrom
```

转到/mnt/cdrom将能看到光驱的内容，执行下面命令进行安装
```shell
./VboxLinuxAdditions.run
```
成功安装 VboxGuestAdditions 后，执行`reboot`

d.重启后执行挂载；
```shell
mkdir -p /mnt/web
mkdir -p /mnt/vhosts
mount -t vboxsf web /mnt/web
mount -t vboxsf vhosts /mnt/vhosts
```

将mount命令加入/etc/rc.d/rc.local，实现自动挂载
web,vhosts的应该与挂载文件夹时取的名称一致