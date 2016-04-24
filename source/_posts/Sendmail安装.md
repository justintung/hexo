title: Sendmail安装
date: 2016-04-23 22:14:36
tags:
- sendmail
- 安装
---
###A. 安装相关软件包
centos默认安装了sendmail服务，但还需要安装相关软件包
```shell
yum -y install sendmail-cf
```
###B.配置
1.监听IP修改
```shell
vi /etc/mail/sendmail.mc
```
相应行修改如下：
from: 
```shell
DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl
```
to:	
```shell
DAEMON_OPTIONS(`Port=smtp,Addr=0.0.0.0, Name=MTA')dnl
```
重新生成sendmail.cf
```shell
 m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf
```
2.FQDN设置
```shell
vi  /etc/mail/local-host-names
```
追加内容如下：
```shell
knowstep.com
```
3.hosts修改
```shell
vi  /etc/hosts
```
相应行修改如下：
from: 
```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
```
to:	
```shell
127.0.0.1   knowstep.com localhost localhost.localdomain localhost4 localhost4.localdomain4
```
4.重启sendmail服务
```shell
service sendmail restart
```
5.测试