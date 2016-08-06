---
title: Linux 常用的命令
date: 2016-08-05 23:17:32
tags:
- Linux 命令
categories:
- Linux
---
1.iftop
功能：查看实时的网络流量，监控TCP/IP连接
```shell
#安装
yum install -y iftop
```
运行iftop，将得到流量相关的交互界面
```shell
iftop -i eth1
```
**界面参数说明**：
中间的<= =>这两个左右箭头，表示的是流量的方向。
TX：发送流量
RX：接收流量
TOTAL：总流量
Cumm：运行iftop到目前时间的总流量
peak：流量峰值
rates：分别表示过去 2s 10s 40s 的平均流量

**常用的参数**：
-i 设定监测的网卡，如：# iftop -i eth1
-B 以bytes为单位显示流量(默认是bits)，如：# iftop -B
-N 使端口信息默认直接都显示端口号，如: # iftop -N
-F 显示特定网段的进出流量，如# iftop -F 10.10.1.0/24或# iftop -F 10.10.1.0/255.255.255.0
-h 显示参数信息
**
进入iftop画面后的一些操作命令(注意大小写)**：
按q退出监控。
按P切换暂停/继续显示
按l打开屏幕过滤功能，输入要过滤的字符，比如ip,按回车后，屏幕就只显示这个IP相关的流量信息;

2.uptime
功能：平均负载
```shell
uptime

#运行结果
23:38:36 up  3:17,  2 users,  load average: 0.93, 0.74, 0.95
```
当前时间 23:38:36
系统已运行的时间 3:17
当前在线用户 2 users
平均负载：0.93, 0.74, 0.95，最近1分钟、5分钟、15分钟系统的负载

3.iostat
功能：iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
```shell
#安装 含有 iostat、mpstat、sar
yum install -y sysstat
```
4.mpstat
功能：报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中

5.sar
```shell
sar -u 1 5
```
输出CPU使用情况的统计信息，每秒输出一次，一共输出100次
```shell
sar -b 1 5
```
显示I/O和传送速率的统计信息
```shell
sar -c
```
每秒钟创建的进程数
6.top
功能：查看负载


















