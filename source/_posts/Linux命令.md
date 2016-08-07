title: Linux命令
date: 2016-05-14 18:21:36
tags:
- Linux
- 命令
categories:
- Linux
---
1.find
svn 删除所有的 .svn文件
```shell
find ./ -name .svn -type d -exec rm -fr {} \;
```
查找文件内容中包含“某文字”
```shell
find ./ -type f|xargs grep "包含文字"
```
2.scp
说明：可以在 2个 linux 主机间复制文件
从 本地 复制到 远程 
```shell
#单文件
scp local_file remote_username@remote_ip:remote_folder

#目录
scp -r local_folder remote_username@remote_ip:remote_folder
```
从 远程 复制到 本地
```shell
#单文件
scp remote_username@remote_ip:remote_folder local_file

#目录
scp -r remote_username@remote_ip:remote_folder local_folder
```
3.rz、sz
说明：linux-window 接收、发送文件
```shell
#安装
yum install -y lrzsz
```
4.