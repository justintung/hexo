title: RabbitMQ php demo
date: 2016-05-7 23:00:36
tags:
- 安装
- RabbitMQ
- php amqp使用
---
####1.producter.php
```php
<?php
//连接RabbitMQ
$conn_args = array( 'host'=>'127.0.0.1' , 'port'=> '5672', 'login'=>'guest' , 'password'=> 'guest','vhost' =>'/');
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

####2.consumer.php
```php
<?php
//连接RabbitMQ
$conn_args = array( 'host'=>'127.0.0.1' , 'port'=> '5672', 'login'=>'guest' , 'password'=> 'guest','vhost' =>'/');
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
####3.执行
```shell
php producter.php
php consumer.php
```
输出：
```shell
/mnt/web/php/test/rabbitmq/consumer.php:16:
array(2) {
  [0] =>
  string(12) "Hello World!"
  [1] =>
  string(6) "DIRECT"
}
```