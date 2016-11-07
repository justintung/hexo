---
title: docker开发环境
date: 2016-11-06 21:49:26
tags:
- docker开发环境
- docker
categories:
- 架构/软件
---
脱离virtualbox，实现开发环境docker化
1.使用镜像
```shell
sudo docker pull busybox
sudo docker pull mysql:5.7
sudo docker pull mongo:3.2
sudo docker pull redis:3.2
sudo docker pull memcached:1.4
sudo docker pull rabbitmq:3.6-management
sudo docker pull php:7.0-fpm
sudo docker pull php:5.6-fpm
sudo docker pull nginx:1.11
```
2.给php镜像安装扩展,安装gd库，yii2需要使用
Dockerfile文件
```shell
FROM php:7.0-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```
编译，进入Dockerfile文件保存目录
```shell
docker build -t local-php:7.0-fpm .
```
同样的方式可以穿件local-php:5.6-fpm

2.运行镜像，写入一个文件中service.sh
```shell
#!/bin/bash 
sudo docker run -d -v /data/docker:/data --name webdata busybox > /dev/null 2>&1
#mysql
sudo docker run -d -v /data/docker/data/mysql:/var/lib/mysql -v /data/docker/conf/mysql:/etc/mysql/conf.d -p 8306:3306 -e MYSQL_ROOT_PASSWORD=3344520 --name mysql mysql:5.7 > /dev/null 2>&1
#mongo
sudo docker run -d -v /data/docker/data/mongo:/data/db -p 27017:27017 --name mongo  mongo:3.2 > /dev/null 2>&1
#redis
sudo docker run -d -v /data/docker/data/redis:/data -p 6379:6379 --name redis redis:3.2 redis-server --appendonly yes > /dev/null 2>&1
#memcached
sudo docker run -d --name memcached memcached:1.4 > /dev/null 2>&1
#rabbitmq
sudo docker run -d --hostname rabbitmq -p 15672:15672 -e RABBITMQ_DEFAULT_USER=justin -e RABBITMQ_DEFAULT_PASS=3344520 --name rabbitmq  rabbitmq:3.6-management > /dev/null 2>&1
#php7.0
sudo docker run -d -v /data/docker/codes/php:/var/www/html -p 9000:9000 --name php70 local-php:7.0-fpm > /dev/null 2>&1
#php5.6
sudo docker run -d -v /data/docker/codes/php:/var/www/html -p 9001:9000 --name php56 local-php:5.6-fpm > /dev/null 2>&1
#nginx
sudo docker run -d -v /data/docker/codes/php:/usr/share/nginx/html -v /data/docker/conf/nginx/nginx.conf:/etc/nginx/nginx.conf --link php70:php70 --link php56:php56 --link memcached:memcached --link mysql:mysql --link mongo:mongo --link redis:redis --link rabbitmq:rabbitmq -p 80:80 --name nginx nginx:1.11
```
3./data/docker/conf/nginx/nginx.conf
```shell
user  nginx;
worker_processes auto;

error_log stderr notice;
pid        /var/run/nginx.pid;


events {
  multi_accept on;
  use epoll;
  worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
                      
    log_format logstash '$http_host $server_addr $remote_addr [$time_local] "$request" '
                    '$request_body $status $body_bytes_sent "$http_referer" "$http_user_agent" '
                    '$request_time $upstream_response_time';
    #/var/log/nginx/access.log已被重定向到/dev/stdout，/var/log/nginx/error.log已被重定向到/dev/stderr
    access_log  /var/log/nginx/access.log  main; 
    
    sendfile        off;
    #sendfile     on;
    #tcp_nopush     on;

    gzip  on;
    keepalive_timeout  65;

    autoindex on;   #开启nginx目录浏览功能
    autoindex_exact_size off;   #文件大小从KB开始显示
    autoindex_localtime on;   #显示文件修改时间为服务器本地时间



    server {
        listen      80;
        server_name  test.knowstep.com;

        location / {
            root   /usr/share/nginx/html/test;
            index  index.php index.html index.htm;
        }
        location ~ \.php$ {
            root           /var/www/html/test;
            fastcgi_pass   php70:9000;
            
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        location ~ /\.(ht|svn|git) {
            deny all;
        }
    }
    server {
        listen      80;
        server_name  yii.knowstep.com;

        location / {
            root   /usr/share/nginx/html/advanced/frontend/web;
            index  index.php index.html index.htm;
        }
        location ~ \.php$ {
            root           /var/www/html/advanced/frontend/web;
            fastcgi_pass   php56:9000;
            
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        location ~ /\.(ht|svn|git) {
            deny all;
        }
    }
}

```
4./data/docker/conf/mysql目录下有一个my.cnf
```shell
[mysqld]
sql_mode=NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
5.可以开启一个容器运行redis-cli，操作对应的redis
```shell
sudo docker run -it --rm --link redis:redis redis:3.2 redis-cli -h redis -p 6379
```
6.最终
| 目录 | 容器目录 | 目录作用 | 原因 |
|--------|--------|--------|--------|
|/data/docker/data/mysql|/var/lib/mysql|mysql数据存储目录|mysql容器中使用了对应的-v|
|/data/docker/conf/mysql|/etc/mysql/conf.d|mysql配置文件目录|mysql容器中使用了对应的-v|
|/data/docker/data/mongo|/data/db|mongo数据存储目录|容器使用了-v|
|/data/docker/data/redis|/data|redis数据存储目录|容器使用了-v|
|/data/docker/codes/php|/var/www/html|php代码位置||
|/data/docker/codes/php|/usr/share/nginx/html||站点资源位置，静态资源nginx处理，php代码通过fastcgi处理|