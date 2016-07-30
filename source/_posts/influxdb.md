---
title: InfluxDB系列安装
date: 2016-07-30 23:17:32
tags:
- 安装
- 工具
categories:
- 架构/软件
---
#### 一、安装
1.Telegraf
Telegraf （通过插件）负责时序数据收集
```shell
wget https://dl.influxdata.com/telegraf/releases/telegraf-0.13.1.x86_64.rpm
sudo yum localinstall telegraf-0.13.1.x86_64.rpm
```

2.InfluxDB
InfluxDB 负责时序数据存储。
```shell
wget https://dl.influxdata.com/kapacitor/releases/kapacitor-0.13.1.x86_64.rpm
sudo yum localinstall kapacitor-0.13.1.x86_64.rpm
```

3.Chronograf
Chronograf 负责数据可视化
```shell
wget https://dl.influxdata.com/chronograf/releases/chronograf-0.13.0-1.x86_64.rpm
sudo yum localinstall chronograf-0.13.0-1.x86_64.rpm
```

4.Kapacitor
Kapacitor负责监控和警告
```shell
wget https://dl.influxdata.com/kapacitor/releases/kapacitor-0.13.1.x86_64.rpm
sudo yum localinstall kapacitor-0.13.1.x86_64.rpm
```
![http://justintung.github.com/images/tick-stack-grid.jpg](http://justintung.github.com/images/tick-stack-grid.jpg)
#### 二、配置
1.telegraf
```shell
telegraf -sample-config -input-filter cpu:mem -output-filter influxdb > /etc/telegraf/telegraf.conf
```
命令收集cpu、mem数据到influxdb，input、output 以插件的方式存在，[input插件地址](https://docs.influxdata.com/telegraf/v0.13/inputs/)、[output插件地址](https://docs.influxdata.com/telegraf/v0.13/outputs/)
2.查看influxdb中收集的数据，InfluxQL
```shell
influxdb
```
进入influxdb的CLI交互模式，类似mysql

| 数据库 | 显示数据库 | 使用数据库 | 显示表格 |
|--------|--------|--------|--------|
|influxdb|show databases|use database_name|show measurements|
|mysql|show databases|show database_name|show tables|
由上面的配置可以看到show measurements显示含有2个measurement（类似mysql的表）
```shell
show field keys
```
显示所有measurement的field keys（类似mysql的表字段）
```shell
SELECT usage_idle FROM cpu WHERE cpu = 'cpu-total' LIMIT 5
```
查询数据
[influxdb的数据库操作](https://docs.influxdata.com/influxdb/v0.13/introduction/getting_started/)
3.chronograf
```shell
service chronograf start

#修改chronograf绑定的ip:端口
vi /opt/chronograf/config.toml

#重启chronograf
service chronograf restart

iptables -I INPUT -s 192.168.56.0/24 -p tcp --dport 10000 -j ACCEPT
service iptables save
service iptables restart
```
登录[http://192.168.56.102:10000/](http://192.168.56.102:10000/)建立自己的视图：
[https://docs.influxdata.com/chronograf/v0.13/introduction/getting_started/](https://docs.influxdata.com/chronograf/v0.13/introduction/getting_started/)