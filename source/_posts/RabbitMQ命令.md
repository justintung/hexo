title: RabbitMQ命令
date: 2016-05-7 23:30:36
tags:
- RabbitMQ命令
- RabbitMQ
---
####为一个项目创建专门的vhost
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

用户创建后可以登陆：[http://192.168.56.110:15672/](http://192.168.56.110:15672/)
5.php demo
producter.php
```php
<?php
//连接RabbitMQ
$conn_args = array( 'host'=>'127.0.0.1' , 'port'=> '5672', 'login'=>'knowstep' , 'password'=> '3344520','vhost' =>'/knowstep');
$conn = new AMQPConnection($conn_args);
$conn->setTimeout(1);
$conn->connect();
//创建exchange名称和类型
$channel = new AMQPChannel($conn);
$ex = new AMQPExchange($channel);
$ex->setName('knowstep_exchange');
$ex->setType(AMQP_EX_TYPE_DIRECT);
$ex->setFlags(AMQP_DURABLE | AMQP_AUTODELETE);
$ex->declare();
//创建queue名称，使用exchange，绑定routingkey
$q = new AMQPQueue($channel);
$q->setName('knowstep_queue');
$q->setFlags(AMQP_DURABLE | AMQP_AUTODELETE);
$q->declare();
$q->bind('knowstep_exchange', 'knowstep_routingkey');
//消息发布
$channel->startTransaction();
$message = json_encode(array('Hello World!','DIRECT'));
$ex->publish($message, 'knowstep_routingkey',null,['delivery_mode'=>2]);
$channel->commitTransaction();
$conn->disconnect();
```
consumer.php
```php
<?php
//连接RabbitMQ
$conn_args = array( 'host'=>'127.0.0.1' , 'port'=> '5672', 'login'=>'knowstep' , 'password'=> '3344520','vhost' =>'/knowstep');
$conn = new AMQPConnection($conn_args);
$conn->connect();
//设置queue名称，使用exchange，绑定routingkey
$channel = new AMQPChannel($conn);
$q = new AMQPQueue($channel);
$q->setName('knowstep_queue');
$q->setFlags(AMQP_DURABLE | AMQP_AUTODELETE);
$q->declare();
$q->bind('knowstep_exchange', 'knowstep_routingkey'); 
//消息获取
$messages = $q->get(AMQP_AUTOACK) ;
if ($messages){
    var_dump(json_decode($messages->getBody(), true ));
}
$conn->disconnect();

```
####命令
```shell
stop [<pid_file>]
stop_app
start_app
wait <pid_file>
reset
force_reset
rotate_logs <suffix>

join_cluster <clusternode> [--ram]
cluster_status
change_cluster_node_type disc | ram
forget_cluster_node [--offline]
update_cluster_nodes clusternode
sync_queue queue
cancel_sync_queue queue

#用户相关
add_user <username> <password>
delete_user <username>
change_password <username> <newpassword>
clear_password <username>
set_user_tags <username> <tag> ...
list_users

#vhost相关
add_vhost <vhostpath>
delete_vhost <vhostpath>
list_vhosts [<vhostinfoitem> ...]
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
clear_permissions [-p <vhostpath>] <username>
list_permissions [-p <vhostpath>]
list_user_permissions <username>

set_parameter [-p <vhostpath>] <component_name> <name> <value>
clear_parameter [-p <vhostpath>] <component_name> <key>
list_parameters [-p <vhostpath>]

set_policy [-p <vhostpath>] <name> <pattern>  <definition> [<priority>] 
clear_policy [-p <vhostpath>] <name>
list_policies [-p <vhostpath>]

list_queues [-p <vhostpath>] [<queueinfoitem> ...]
list_exchanges [-p <vhostpath>] [<exchangeinfoitem> ...]
list_bindings [-p <vhostpath>] [<bindinginfoitem> ...]
list_connections [<connectioninfoitem> ...]
list_channels [<channelinfoitem> ...]
list_consumers [-p <vhostpath>]
status
environment
report
eval <expr>

close_connection <connectionpid> <explanation>
trace_on [-p <vhost>]
trace_off [-p <vhost>]
set_vm_memory_high_watermark <fraction>

```