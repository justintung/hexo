title: svn客户端Subversion更新
date: 2016-08-17 12:20:36
tags:
- SVN
- Subversion
categories:
- 安装
---
yum 系统的svn版本过低，从源码编译subversion常常需要重新编译glibc，非常的慢；所以可以采用如下方式安装
1.创建yum资源
```shell
vi /etc/yum.repos.d/wandisco-svn.repo
```
内容如下：
```shell
[WandiscoSVN]
name=Wandisco SVN Repo
baseurl=http://opensource.wandisco.com/centos/$releasever/svn-1.9/RPMS/$basearch/
enabled=1
gpgcheck=0
```
2.安装
```shell
yum remove subversion*

yum clean all
yum install subversion
```
3.查看版本
```shell
svn --version
```

