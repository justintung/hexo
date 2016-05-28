---
title: LVS (ipvsadm)+keepalived安装
date: 2016-05-28 01:22:17
tags:
- LVS
- keepalived
- 负载均衡
- 服务器状态检测
categories:
- 架构/软件
---
![虚拟服务器结构示意图](http://justintung.github.com/images/2016.5.28.lvs1.png)

ipvs:LVS的核心是ipvs,ipvs通过ipvsadm来实现
Lvs客户端：负载均衡器/转发器后面的负责提供服务的真实机器

Ipvsadm安装
```shell
yum install -y libnl* libpopt* popt-static 
wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz
tar zxvf ipvsadm-1.26.tar.gz
cd ipvsadm-1.26
ln -sf /usr/src/kernels/2.6.32-573.el6.x86_64 /usr/src/linux

#(kernels 应该与当前使用内核版本一致，用uname -a 查看)
make && make install
```

Keepalived 安装
```shell
wget http://www.keepalived.org/software/keepalived-1.2.19.tar.gz
tar zxvf keepalived-1.2.19.tar.gz
cd keepalived-1.2.19
./configure --prefix=/www/itmotu/keepalived

make && make install
```

详细配置参考：
[http://www.jizhuomi.com/software/351.html](http://www.jizhuomi.com/software/351.html)

[http://blog.chinaunix.net/uid-25266990-id-3989321.html](http://blog.chinaunix.net/uid-25266990-id-3989321.html)