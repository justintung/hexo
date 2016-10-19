---
title: hhvm
date: 2016-05-27 23:49:26
tags:
- hhvm
- 进程监控
categories:
- 架构/软件
---
####安装
```shell
yum -y install http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm

cd /etc/yum.repos.d
wget http://www.hop5.in/yum/el6/hop5.repo

yum clean all
yum install hhvm
```
