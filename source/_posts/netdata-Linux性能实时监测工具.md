title: netdata  Linux 性能实时监测工具
date: 2016-04-24 22:14:36
tags:
- netdata
- 安装
- 性能监控
- centos
---
###1.centos安装相关软件包
```shell
yum install -y zlib-devel gcc make git autoconf autogen automake pkgconfig
```
其他系统查看[https://github.com/firehol/netdata/wiki/Installation](https://github.com/firehol/netdata/wiki/Installation)
###2.netdata安装
```shell
git clone https://github.com/firehol/netdata.git --depth=1
cd netdata
./netdata-installer.sh --install /www/itmotu
iptables -I INPUT -p tcp --dport 19999 -j ACCEPT
service iptables save
service iptables restart
```
###3.卸载
```shell
./netdata-uninstaller.sh
```
