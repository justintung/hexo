title: gcc版本更新
date: 2016-11-25 23:01:36
tags:
- gcc
categories:
- 安装
---
yum 系统的gcc版本过低
1.执行命令
```shell
$ sudo wget -O /etc/yum.repos.d/slc6-devtoolset.repo http://linuxsoft.cern.ch/cern/devtoolset/slc6-devtoolset.repo
```
2.安装
```shell
yum remove gcc

yum clean all
yum install gcc
```
3.查看版本
```shell
gcc -v
```

上面的gcc版本只到4.7, 4.8可以用下面的方式
```
yum install centos-release-scl
yum-config-manager --enable rhel-server-rhscl-7-rpms
yum install devtoolset-2-gcc  devtoolset-2-gcc-c++

source /opt/rh/devtoolset-2/enable
```