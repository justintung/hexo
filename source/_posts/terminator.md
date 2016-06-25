title: Terminator
date: 2016-05-7 12:20:36
tags:
- Terminator
- Terminal
- Linux
categories:
- Linux
---
1.说明
网址：[https://launchpad.net/terminator/](https://launchpad.net/terminator/)
Terminator是终端增强软件，最大的特点就是可以在一个屏幕下同时显示多个终端。
2.安装
```shell
sudo apt-get install terminator
```
3.窗口大小配置
```shell
sudo vi /usr/share/applications/terminator.desktop
```
修改Exec项为
```shell
Exec=terminator --geometry=1024*512
```
打开的窗口为居中显示
```shell
Exec=terminator --geometry=1024*512+40+20
```
1024*512：窗口大小，40：左顶点距离，20右顶点距离