---
title: docker ELK
date: 2016-06-25 23:49:26
tags:
- docker
- ELK
- Elasticsearch
- Logstach
- Kibana
- 安装
categories:
- 架构/软件
---

####1.拉取镜像
```shell
sudo docker pull elasticsearch:5.0
sudo docker pull logstash:5.0
sudo docker pull kibana:5.0
```
####2.运行容器
```shell
#!/bin/bash
sudo docker run -d -v /coding/docker/data/elasticsearch:/usr/share/elasticsearch/data --name elasticsearch elasticsearch:5.0 > /dev/null 2>&1
sudo docker run -itd -v /coding/docker/conf/logstash:/config --name logstash logstash:5.0 -f /config/logstash.conf > /dev/null 2>&1
sudo docker run -d --link elasticsearch:elasticsearch -p 5601:5601 --name kibana  kibana:5.0 > /dev/null 2>&1
```
将运行命令放入一个shell脚本中，方便启动

logstash.conf demo
```shell
input { 
    stdin { } 
} 
output { 
    stdout { } 
}
```
####3.访问
elasticsearch 接口地址：[http://elasticsearch容器IP:9200/](http://elasticsearch容器IP:9200/)
kibnana 访问地址：[http://kibnana容器IP:5601/](http://kibnana容器IP:5601/)

####4.坑
```shell
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
解决方法：切换到root用户修改配置 /etc/sysctl.conf
添加下面配置：
```shell
vm.max_map_count=655360
```
并执行命令：
```shell
sudo sysctl -p
```
然后，重新启动elasticsearch，即可启动成功。