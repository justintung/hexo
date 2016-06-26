---
title: docker
date: 2016-06-25 23:49:26
tags:
- docker
- 安装
categories:
- 架构/软件
---
官方文档：[https://docs.docker.com/engine/installation/linux/](https://docs.docker.com/engine/installation/linux/)
####1.Debian 中安装docker
```shell
sudo vi /etc/apt/sources.list.d/docker.list
#内容如下
deb https://apt.dockerproject.org/repo debian-jessie main

sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

sudo apt-get update
sudo apt-cache policy docker-engine
sudo apt-get install docker-engine
sudo service docker start
```
####2.常用镜像
```shell
sudo docker pull busybox
sudo docker pull mysql:5.7
sudo docker pull mongo:3.2
sudo docker pull redis:3.2
sudo docker pull rabbitmq:3.6-management
sudo docker pull million12/nginx-php:php-55
```
####3.运行容器
```shell
#!/bin/bash
#公共容器
sudo docker run -d -v /coding/docker:/data --name webdata busybox > /dev/null 2>&1
sudo docker run -d -v /coding/docker/data/mysql:/var/lib/mysql -p 8306:3306 -e MYSQL_ROOT_PASSWORD=3344520 --name mysql mysql:5.7 > /dev/null 2>&1
sudo docker run -d -v /coding/docker/data/mongo:/data/db -p 27017:27017 --name mongo  mongo:3.2 > /dev/null 2>&1
sudo docker run -d -v /coding/docker/data/redis:/data -p 6379:6379 --name redis redis:3.2 redis-server --appendonly yes > /dev/null 2>&1
sudo docker run -d --hostname rabbitmq -p 15672:15672 -e RABBITMQ_DEFAULT_USER=justin -e RABBITMQ_DEFAULT_PASS=3344520 --name rabbitmq  rabbitmq:3.6-management > /dev/null 2>&1

#项目容器
sudo docker run -itd -v /coding:/coding --volumes-from=webdata -p 8081:80 --name test.itmotu.com million12/nginx-php:php-55
```
将运行命令放入一个shell脚本中，方便启动

####4.docker 命令
| 功能 | 命令 |
|--------|--------|
|   查看容器信息     |   docker inspect 容器ID    |
|   查看容器日志信息     |   docker logs 容器ID     |
|   进入正在运行的容器     |   docker attach 容器ID     |
|   从容器内退出     |   ctrl+p+q     |