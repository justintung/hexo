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
5.批量kill掉进程
```shell
ps aux|grep nginx|awk '{print $2}'|xargs kill -9
```
命令批量kill掉含有nginx字符的进程
ps后面的aux不要加`-`，不然会提示错误
另外应该排除命令本身，所以shell最好如下
```shell
ps aux|grep nginx|grep -v grep|awk '{print $2}'|xargs kill -9
```
6.dig命令
命令用于查询域名的解析情况
```shell
dig www.365coding.com
```
显示出路由轨迹
```shell
dig www.365coding.com +trace
```
查询域名在指定DNS下的解析情况：
```shell
dig www.365coding.com @114.114.114.114
```
windows下类似的命令为，
```shell
nslookup -qt=A www.365coding.com 114.114.114.114
```
7.清除dns缓存
```shell
aptitude install nscd
service nscd restart
```
windows下类似命令为
```shell
ipconfig /flushdns
```
8.htop
命令类似于top，但更加丰富
```shell
yum install -y htop
htop
```