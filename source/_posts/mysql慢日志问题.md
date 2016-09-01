---
title: mysql慢日志问题
date: 2016-09-01 14:46:52
tags:
- 慢日志
categories:
-  mysql
---
####1.开启慢日志
修改 /etc/my.cnf 添加如下语句：
```shell
slow_query_log                          = 1
long_query_time                         = 1
slow_query_log_file                     = slow.log
#log_queries_not_using_indexes           = ON
```
如果第4条开启的话，mysql会记录所有未使用索引的语句都会加入到慢日志文件中，不管该语句的运行时间
所以这个设置建议是要关闭掉的，不然慢日志异常的大

####2.修改慢日志配置
如果允许重启mysql的话，那就直接修改my.cnf文件
不允许的话，按如下方式操作：
a.登入mysql客户端命令行,查看当前配置
```shell
show variables like '%slow%';
```
b.修改slow_query_log值为0，即关闭慢日志
```shell
set global slow_query_log=off;
set global log_queries_not_using_indexes=off;
```
c.将slow.log转移或更改slow_query_log_file值为新的文件
d.修改slow_query_log值为1，即重新开启慢日志
```shell
set global slow_query_log=on;
```
