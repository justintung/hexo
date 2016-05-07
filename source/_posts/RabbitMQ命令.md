title: RabbitMQ命令
date: 2016-05-7 23:30:36
tags:
- RabbitMQ命令
- RabbitMQ
---
1.添加vhost
```shell
rabbitmqctl add_vhost /knowstep
```
创建vhost /knowstep
2.创建用户
```shell
rabbitmqctl add_user knowstep 3344520
```
创建用户knowstep，密码为3344520
3.设置用户角色
```shell
rabbitmqctl set_user_tags knowstep management
```
设置用户knowstep为management角色
4.设置用户权限
```shell
rabbitmqctl set_permissions -p /knowstep knowstep '.*' '.*' '.*'
```
设置用户knowstep对vhost /knowstep 拥有所有权限