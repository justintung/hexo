---
title: docker自定义网络
date: 2016-11-06 21:49:26
tags:
- docker自定义网络
- docker
categories:
- 架构/软件
---
脱离virtualbox，实现开发环境docker化
```
#!/bin/bash
sudo docker run -d -v /data/docker:/data --name webdata busybox > /dev/null 2>&1
#network
sudo docker network create --subnet=172.18.0.0/16 --gateway=172.18.0.1 standalone_subnet > /dev/null 2>&1
#mysql
sudo docker run -d -v /data/docker/data/mysql:/var/lib/mysql -v /data/docker/conf/mysql:/etc/mysql/conf.d -p 8306:3306 -e MYSQL_ROOT_PASSWORD=3344520 --network standalone_subnet --ip 172.18.0.2 --name mysql mysql:5.7 > /dev/null 2>&1
#redis
sudo docker run -d -v /data/docker/data/redis:/data -p 6379:6379 --network standalone_subnet --ip 172.18.0.3 --name redis redis:latest redis-server --appendonly yes > /dev/null 2>&1
#mongo
sudo docker run -d -v /data/docker/data/mongo:/data/db -p 27017:27017 --network standalone_subnet --ip 172.18.0.4 --name mongo  mongo:latest > /dev/null 2>&1

#memcached
#sudo docker run -d -p 11211:11211 --network standalone_subnet --ip 172.18.0.5 --name memcached memcached:latest > /dev/null 2>&1
#rabbitmq
#sudo docker run -d --hostname rabbitmq -p 15672:15672 -e RABBITMQ_DEFAULT_USER=justin -e RABBITMQ_DEFAULT_PASS=3344520 --network standalone_subnet --ip 172.18.0.6 --name rabbitmq  rabbitmq:latest > /dev/null 2>&1
#postgres
#sudo docker run -d -v /data/docker/data/postgres:/var/lib/postgresql -p 5432:5432 -e POSTGRES_PASSWORD=3344520 --network standalone_subnet --ip 172.18.0.7  --name postgres postgres:latest > /dev/null 2>&1

#php7.3
sudo docker run -d -v /data/docker/codes/php:/var/www/html -p 9000:9000 --network standalone_subnet --ip 172.18.0.8  --name php73 php:7.3-fpm > /dev/null 2>&1
#php5.6
sudo docker run -d -v /data/docker/codes/php:/var/www/html -p 9001:9000 --network standalone_subnet --ip 172.18.0.9  --name php56 php:5.6-fpm > /dev/null 2>&1

#nginx
sudo docker run -d -v /data/docker/codes:/usr/share/nginx/html  -v /data/coding/config/vhosts:/etc/nginx/vhosts -v /data/docker/conf/nginx/nginx.conf:/etc/nginx/nginx.conf --link php73:php73 --link php56:php56 --link mysql:mysql --link mongo:mongo --link redis:redis -p 80:80 --network standalone_subnet --ip 172.18.0.10  --privileged=true --name nginx nginx:latest > /dev/null 2>&1
```
nginx.conf
```
#user  nobody;
worker_processes  auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
worker_rlimit_nofile 102400;

events
{
    use epoll;
    worker_connections 102400;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    server_tokens off;
    log_format  main  '$host - $remote_addr - $remote_port $remote_user [$time_local] $request '
        '"$status" $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"'
        '"$gzip_ratio"';
    log_format logstash_json '{"@timestamp": "$time_iso8601", '
                              '"@fields": {'
                                '"host": "$host", '
                                '"remote_addr": "$remote_addr", '
                                '"remote_port": "$remote_port", '
                                '"remote_user": "$remote_user", '
                                '"scheme": "$scheme", '
                                '"request": "$request", '
                                '"request_uri": "$request_uri", '
                                '"uri": "$uri", '
                                '"request_method": "$request_method", '
                                '"server_protocol": "$server_protocol", '
                                '"request_completion": "$request_completion", '
                                '"status": "$status", '
                                '"body_bytes_sent": "$body_bytes_sent", '
                                '"bytes_sent": "$bytes_sent", '
                                '"request_time": "$request_time", '
                                '"http_user_agent": "$http_user_agent", '
                                '"http_referrer": "$http_referer", '
                                '"http_x_forwarded_for": "$http_x_forwarded_for", '
                                '"proxy_host": "$proxy_host", '
                                '"proxy_port": "$proxy_port"}}';
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;


    keepalive_timeout 75 20;
    request_pool_size       4k;
    connection_pool_size    256;

    client_header_timeout  3m;
    client_body_timeout    3m;
    send_timeout           3m;
    client_header_buffer_size    32k;
    large_client_header_buffers    4 32k;

    output_buffers   4 32k;
    postpone_output  1460;
    client_max_body_size       20m;
    client_body_buffer_size    256k;
    client_body_temp_path    /dev/shm/client_body_temp;
    proxy_temp_path            /dev/shm/proxy_temp;
    fastcgi_temp_path          /dev/shm/fastcgi_temp;
    server_names_hash_bucket_size 128;

    #gzip
    gzip on;
    gzip_http_version 1.1;
    gzip_comp_level 4;
    gzip_proxied any;
    gzip_types text/plain text/css application/x-javascript application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_min_length 1100;
    gzip_buffers 4 8k;

    etag off;
    access_log off;

    include vhosts/*.conf;
}
```
nginx vhost
```
server {
    charset utf-8;
    client_max_body_size 128M;

    listen 80;
    server_name test.qq.com;
    root /usr/share/nginx/html/php/test_qq;
    index index.php index.html ;


    location ~ \.php$ {
      root /var/www/html/test_qq;
      fastcgi_pass    php73:9000;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```