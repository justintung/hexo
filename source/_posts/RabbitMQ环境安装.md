title: RabbitMQ环境安装
date: 2016-05-2 23:44:36
tags:
- 安装
- RabbitMQ
---
####1.erlang安装
```shell
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
rpm -i rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
yum update
yum install -y erlang*
```
或
```shell
wget -O /etc/yum.repos.d/epel-erlang.repo http://repos.fedorapeople.org/repos/peter/erlang/epel-erlang.repo
yum install erlang
```

####2.rabbitmq-server 的安装
```shell
yum install -y librabbitmq-devel rabbitmq-server librabbitmq
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.1.5/sbin/
```
插件安装：
```shell
./rabbitmq-plugins enable rabbitmq_management
```
服务启动：
```shell
service rabbitmq-server start
iptables -I INPUT -p tcp --dport 15672 -j ACCPT
service iptables save
service iptables restart
```
访问：[http://192.168.56.110:15672/](http://192.168.56.110:15672/)
使用guest/guest登陆
####3.php amqp 扩展的安装
amqp基于rabbitmq-c，先安装rabbitmq-c
```shell
git clone https://github.com/alanxz/rabbitmq-c.git
cd rabbitmq-c
autoreconf -i
./configure
make
make install

ln -sf /usr/local/lib64/librabbitmq.* /usr/local/lib/
```
安装amqp扩展
```shell
wget http://pecl.php.net/get/amqp-1.7.0.tgz
tar zxvf amqp-1.7.0.tgz -C ./
cd amqp-1.7.0/
/www/itmotu/php-5.5.33/bin/phpize
./configure --with-php-config=/www/itmotu/php-5.5.33/bin/php-config
make && make install
```
