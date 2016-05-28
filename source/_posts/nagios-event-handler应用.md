title: nagios event handler应用
date: 2016-05-14 18:21:36
tags:
- nagios
- 监控
- event handler
categories:
- 架构/软件
---
nagios的事件处理（Event Handlers）可以在监控的主机或服务的状态发生变化时触发脚本或系统命令来对故障进行处理，事件处理在告警发出前发生。
事件处理会在下面情况下触发：
1). 主机或服务处于一个软态(SOFT)故障状态时；
2). 主机或服务处于一个硬态(HARD)故障状态时；
3). 主机或服务从软态或硬态的故障状态中初始恢复时;
事件处理命令可以通过各种语言脚本来完成，脚本中需要处理下面的参数：
对服务的：$SERVICESTATE$、$SERVICESTATETYPE$和$SERVICEATTEMPT$；
对主机的：$HOSTSTATE$、$HOSTSTATETYPE$和$HOSTATTEMPT$；
脚本须检测这些作为命令行参数传入的值，并采取必要动作来处理这些值。

事件处理命令通常是与运行于本机上的Nagios程序的权限是相同的（下面例子中Nagios服务是以nagios用户运行的）。这可能会有问题，如果你想写成一个用于系统服务重启的命令，它需要有root权限才能执行一系列命令与任务。你或许会尝使用sudo命令来实现它。

本例通过Nagios检测远程机器上的MySQL服务，当服务出现问题时通过Nagios的事件处理逻辑来重启远程机器上的MySQL服务。

####配置在Nagios服务器（192.168.56.101）上无密码登录远程机器（MySQL服务运行在上面-192.168.56.110）

**服务端中运行**
a.生成密钥
```shell
su - nagios
ssh-keygen -t rsa

#下面一直回车，不要设置密码
Generating public/private rsa key pair.
Enter file in which to save the key (/home/nagios/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/nagios/.ssh/id_rsa.
Your public key has been saved in /home/nagios/.ssh/id_rsa.pub.
The key fingerprint is:
d2:82:61:12:53:f9:53:75:77:8d:32:c0:ca:c8:20:60 nagios@nagios.itech.com
```
b.将生成的密钥拷贝到要远程登录的机器上
```shell
scp .ssh/id_rsa.pub 192.168.56.110:/home/nagios/
#提示如下：
nagios@192.168.56.110's password:
id_rsa.pub 100% 233 0.2KB/s 00:00
```
c.在要远程登录的机器上配置公钥
```shell
#提示如下：
nagios@192.168.56.110's password:
Last login: Sat Nov 29 22:30:55 2008 from 192.168.56.101

cat id_rsa.pub >> .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys

#退出
exit
```

d.测试无密码登录
```shell
ssh nagios@192.168.56.110

Last login: Sat Nov 29 22:35:27 2008 from 192.168.56.101
```

####在客户端配置sudo
使nagios用户可以以root身份运行/usr/local/nagios/libexec/eventhandlers/restart-mysql脚本
以root用户登陆
```shell
visudo

#添加
nagios ALL=(root) NOPASSWD:/usr/local/nagios/libexec/eventhandlers/restart-mysql
```

3.在客户端编写MySQL重启脚本
vi /usr/local/nagios/libexec/eventhandlers/restart-mysql
```shell
#!/bin/sh
#
# Event handler script for restarting the MySQL server on the remote machine
#
# Note: This script will only restart the MySQL server if the service is
# retried 2 times (in a "soft" state) or if the web service somehow
# manages to fall into a "hard" error state.
#
#
# What state is the MySQL service in?
case "$1" in
    OK)
     ;;
    WARNING)
     ;;
    UNKNOWN)
     ;;
    CRITICAL)
     	# Is this a "soft" or a "hard" state?
		case "$2" in
             SOFT)
             # What check attempt are we on? We don't want to restart the MySQL server on the first
             # check, because it may just be a fluke!
                 case "$3" in
                     2)
                     echo -n "Restarting MySQL service..."
                     /sbin/service mysqld restart
                     ;;
                 esac
             ;;
             HARD)
                 echo -n "Restarting MySQL service..."
                 /sbin/service mysqld restart
                 ;;
        esac
		;;
    esac
exit 0
```
上面的脚本只会在MySQL处于软状态，且第二次检查出现故障时或者进入硬状态时重启MySQL。
4.在服务端，配置Nagios服务器上的配置文件
```shell
vi /usr/local/nagios/etc/nagios.cfg

#使得
enable_event_handlers=1
```
在命令配置文件中定义重启MySQL的命令
```shell
vi /usr/local/nagios/etc/objects/commands.cfg
#添加
define command{
    command_name restart-mysql
    command_line /usr/bin/ssh nagios@$HOSTADDRESS$ "sudo /usr/local/nagios/libexec/eventhandlers/restart-mysql $SERVICESTATE$ $SERVICESTATETYPE$ $SERVICEATTEMPT$"
 }
``` 
配置主机监控文件
```shell
vi /usr/local/nagios/etc/objects/192.168.56.110.cfg
# 省略主机定义和其他服务定义
define service{
    use generic-service ; Name of service template to use
    host_name MySQL
    service_description MySQL
    check_command check_nrpe!check_mysql
    notifications_enabled 1
    event_handler_enabled 1
    event_handler restart-mysql
}
```
这个脚本理论上在服务转入硬态故障之前可以重启MySQL服务以修复故障，这里包含了首次重启没有成功的情况。须注意的是事件处理将只是第一次进入硬态紧急状态时才会被触发，这将阻止Nagios在服务一直处于硬态故障状态时反复地重启MySQL服务。
