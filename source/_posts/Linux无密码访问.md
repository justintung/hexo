title: Linux无密码访问
date: 2016-05-14 18:21:36
tags:
- Linux
- 无密码访问
categories:
- Linux
---

####配置在192.168.56.101上无密码登录远程机器192.168.56.110

**192.168.56.101中运行**
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
b.将生成的密钥拷贝到要远程登录的机器(192.168.56.110)上
```shell
scp .ssh/id_rsa.pub 192.168.56.110:/home/nagios/
#提示如下：
nagios@192.168.56.110's password:
id_rsa.pub 100% 233 0.2KB/s 00:00
```
c.在要远程登录的机器(192.168.56.110)上配置公钥
```shell
#在192.168.56.101 ssh登录
ssh nagios@192.168.56.110
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

e.实现对本机的无密码访问
如，在192.168.56.110上访问localhost
```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

ssh localhost
```
