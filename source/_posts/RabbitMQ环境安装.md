title: RabbitMQ环境安装
date: 2016-05-2 23:44:36
tags:
- 安装
- RabbitMQ
categories:
- 架构/软件
---
####1.erlang安装
```shell
wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
yum install erlang
```

####2.rabbitmq-server 的安装
```shell
yum install -y librabbitmq-devel rabbitmq-server librabbitmq

#或者

rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
yum install rabbitmq-server-3.6.1-1.noarch.rpm

cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.1/sbin/
```
插件安装：
```shell
./rabbitmq-plugins enable rabbitmq_management
```
服务启动：
```shell
service rabbitmq-server start
iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
service iptables save
service iptables restart
```
访问：[http://192.168.56.110:15672/](http://192.168.56.110:15672/)
使用guest/guest登陆，出于安全因素的考虑，guest用户只能通过localhost登陆使用，并建议修改guest用户的密码以及新建其他账号管理使用rabbitmq(该功能是在3.3.0版本引入的)
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
