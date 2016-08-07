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
4.shutdown
说明：shutdown命令用来系统关机命令。shutdown指令可以关闭所有程序，并依用户的需要，进行重新开机或关机的动作。
**参数**：
```html
-c：当执行“shutdown -h 11:50”指令时，只要按+键就可以中断关机的指令；
-f：重新启动时不执行fsck；
-F：重新启动时执行fsck；
-h：将系统关机；
-k：只是送出信息给所有用户，但不会实际关机；
-n：不调用init程序进行关机，而由shutdown自己进行；
-r：shutdown之后重新启动；
-t<秒数>：送出警告信息和删除信息之间要延迟多少秒。
```
指定现在立即关机：
```shell
shutdown -h now
```
 指定5分钟后关机，同时送出警告信息给登入用户：
```shell
shutdown +5 "System will shutdown after 5 minutes"
```