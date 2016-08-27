title: RabbitMQ用户角色及权限控制
date: 2016-05-7 23:44:36
tags:
- RabbitMQ用户角色
- RabbitMQ
- RabbitMQ权限控制
categories:
- 架构/软件
---
##1.用户角色
####A.RabbitMQ的用户角色分类：
none、management、policymaker、monitoring、administrator

####B.RabbitMQ各类角色描述：
none
不能访问 management plugin

management
用户可以通过AMQP做的任何事，外加：
列出自己可以通过AMQP登入的virtual hosts  
查看自己的virtual hosts中的queues, exchanges 和 bindings
查看和关闭自己的channels 和 connections
查看有关自己的virtual hosts的“全局”的统计信息，包含其他用户在这些virtual hosts中的活动。

policymaker 
management可以做的任何事，外加：
查看、创建和删除自己的virtual hosts所属的policies和parameters

monitoring  
management可以做的任何事，外加：
列出所有virtual hosts，包括他们不能登录的virtual hosts
查看其他用户的connections和channels
查看节点级别的数据如clustering和memory使用情况
查看真正的关于所有virtual hosts的全局的统计信息

administrator   
policymaker和monitoring可以做的任何事，外加:
创建和删除virtual hosts
查看、创建和删除users
查看创建和删除permissions
关闭其他用户的connections

####C.创建用户并设置角色：
```shell
#eg.1
rabbitmqctl add_user  username_admin  password
rabbitmqctl set_user_tags username_admin administrator

#eg.2
rabbitmqctl add_user  username_monitoring  password
rabbitmqctl set_user_tags username_monitoring monitoring

#eg.3
rabbitmqctl  add_user  username_proj  password
rabbitmqctl set_user_tags username_proj management
```
创建和赋角色完成后查看并确认：
```shell
rabbitmqctl list_users
```
##2.RabbitMQ 权限控制
默认virtual host："/"
默认用户：guest 
guest具有"/"上的全部权限，仅能有localhost访问RabbitMQ包括Plugin，建议删除或更改密码。可通过将配置文件中loopback_users置空来取消其本地访问的限制：
[{rabbit, [{loopback_users, []}]}]

用户仅能对其所能访问的virtual hosts中的资源进行操作。这里的资源指的是virtual hosts中的exchanges、queues等，操作包括对资源进行配置、写、读。配置权限可创建、删除、资源并修改资源的行为，写权限可向资源发送消息，读权限从资源获取消息。比如：
exchange和queue的declare与delete分别需要exchange和queue上的配置权限
exchange的bind与unbind需要exchange的读写权限
queue的bind与unbind需要queue写权限exchange的读权限
发消息(publish)需exchange的写权限
获取或清除(get、consume、purge)消息需queue的读权限

对何种资源具有配置、写、读的权限通过正则表达式来匹配，具体命令如下：
```shell
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
```
其中，conf、write、read的位置分别用正则表达式来匹配特定的资源，如'\^(amq\.gen.*|amq\.default)$'可以匹配server生成的和默认的exchange，'\^$'不匹配任何资源

需要注意的是RabbitMQ会缓存每个connection或channel的权限验证结果、因此权限发生变化后需要重连才能生效。


为用户赋权：
```shell
rabbitmqctl  set_permissions -p /vhost1  username_admin '.*' '.*' '.*'
```
该命令使用户username_admin具有/vhost1这个virtual host中所有资源的配置、写、读权限以便管理其中的资源

查看权限：
```shell
rabbitmqctl list_user_permissions username_admin

Listing permissions for user "username_admin" ...
/vhost1 .* .* .*
```
```shell
rabbitmqctl list_permissions -p /vhost1

Listing permissions in vhost "/vhost1" ...
username_admin .* .* .*
```