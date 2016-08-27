---
title: docker redis交互
date: 2016-08-27 12:49:26
tags:
- docker
- redis
- redis-cli
- 安装
categories:
- 架构/软件
---

####1.拉取redis镜像
```shell
sudo docker pull redis:3.2
```
####2.运行 redis-server
```shell
#!/bin/bash
sudo docker run -d -v /data/docker/data/redis:/data -p 6379:6379 --name redis redis:3.2 redis-server --appendonly yes > /dev/null 2>&1
```
容器名字固定为redis
####3.运行 redis-cli
容器名字随机，退出立刻删除容器
```shell
sudo docker run -it --rm --link redis:redis redis:3.2 redis-cli -h redis -p 6379
```
运行后进入redis-cli的交互模式
容器名字不随机无法通过上面命令启动多个redis-cli进行交互
注意：--rm 参数，以及--link中使用的redis-server容器的名称