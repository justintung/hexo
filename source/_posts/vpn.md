---
title: vpn
date: 2016-06-08 20:46:52
tags:
- vpn
categories:
- 安装
---
#### 1.安装PPP，PPTP
```shell
yum install -y ppp
rpm -ivh http://static.ucloud.cn/pptpd-1.3.4-2.el6.x86_64.rpm
```
#### 2.编辑pptp.conf，在最后加入以下两行代码
```shell
# vim /etc/pptpd.conf

localip 10.8.0.1
remoteip 10.8.0.10-100
```
localip为vpn服务器的本地地址，remoteip为分配给vpn客户端使用的ip地址段

#### 3.编辑options.pptpd，在最后加入以下两行代码
```shell
# vim /etc/ppp/options.pptpd

ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

#### 4.编辑chap-secrets，account为pptp登录帐号，password为登录密码，其他默认
```shell

# vim /etc/ppp/chap-secrets
# client        server      secret            IP addresses
  justin       pptpd       3344520          *
```
"*"表示ip由vpn服务器分配

#### 5.编辑sysctl.conf，开启网络转发功能
```shell
# vim /etc/sysctl.conf

net.ipv4.ip_forward = 1
# sysctl -p
```

#### 6.配置NAT
```shell
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
```

#### 7.启动PPTP服务
```shell
service pptpd start
```

#### 8.设置为开机启动
```shell
chkconfig pptpd on
chkconfig iptables on
```