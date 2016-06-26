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
sudo docker pull elasticsearch:2.3
sudo docker pull logstash:2.3
sudo docker pull kibana:4.5
```
####2.运行容器
```shell
#!/bin/bash
sudo docker run -d -v /coding/docker/data/elasticsearch:/usr/share/elasticsearch/data --name elasticsearch elasticsearch:2.3 > /dev/null 2>&1
sudo docker run -itd -v /coding/docker/conf/logstash:/config --name logstash logstash:2.3 -f /config/logstash.conf > /dev/null 2>&1
sudo docker run -d --link elasticsearch:elasticsearch -p 5601:5601 --name kibana  kibana:4.5 > /dev/null 2>&1
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