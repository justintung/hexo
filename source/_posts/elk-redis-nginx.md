---
title: ELK + redis 收集 nginx 日志
date: 2016-06-25 23:49:26
tags:
- ELK
- Elasticsearch
- Logstach
- Kibana
categories:
- 架构/软件
---

####1.拉取镜像
```shell
sudo docker pull busybox
sudo docker pull elasticsearch:2.3
sudo docker pull logstash:2.3
sudo docker pull kibana:4.5
sudo docker pull redis:3.2
```
####2.运行容器
```shell
#!/bin/bash
sudo docker run -d -v /coding/docker:/data --name webdata busybox > /dev/null 2>&1
sudo docker run -d -v /coding/docker/data/redis:/data -p 6379:6379 --name redis redis:3.2 redis-server --appendonly yes > /dev/null 2>&1
sudo docker run -d -v /coding/docker/data/elasticsearch:/usr/share/elasticsearch/data --name elasticsearch elasticsearch:2.3 > /dev/null 2>&1
sudo docker run -d --volumes-from=webdata -v /coding/docker/conf/logstash:/config --name logstash-agent logstash:2.3 -f /config/logstash-agent.conf > /dev/null 2>&1
sudo docker run -d --volumes-from=webdata -v /coding/docker/conf/logstash:/config --name logstash-indexer logstash:2.3 -f /config/logstash-indexer.conf > /dev/null 2>&1
sudo docker run -d --link elasticsearch:elasticsearch -p 5601:5601 --name kibana  kibana:4.5 > /dev/null 2>&1
```
将运行命令放入一个shell脚本中，方便启动

##假设IP情况为：
| 服务 | IP |
|--------|--------|
|redis|172.17.0.3|
|elasticsearch|172.17.0.7|
|kibana|172.17.0.9|

####3.日志配置
在nginx.conf 设置日志格式
```shell
log_format logstash '$http_host $server_addr $remote_addr [$time_local] "$request" '
                    '$request_body $status $body_bytes_sent "$http_referer" "$http_user_agent" '
                    '$request_time $upstream_response_time';
```
虚拟机配置文件中开启日志access_log
```shell
server {
    listen      80;
    server_name  test.knowstep.com;

    root        /data/www/php/test;
    index       index.php index.html;

    include     /etc/nginx/conf.d/default-*.conf;
    include     /data/conf/nginx/conf.d/default-*.conf;

    # PHP backend is not in the default-*.conf file set,
    # as some vhost might not want to include it.
    include     /etc/nginx/conf.d/php-location.conf;

    # Import configuration files for status pages for Nginx and PHP-FPM
    include /etc/nginx/conf.d/stub-status.conf;
    include /etc/nginx/conf.d/fpm-status.conf;
    access_log /data/logs/test.knowstep.com.access.log logstash;
}
```

####4.logstash agent
负责收集nginx日志到redis
logstash-agent.conf
```shell
input { 
    file{
    	type=>"nginx_access"
    	path=>["/data/logs/test.knowstep.com.access.log"]
    }
} 
output { 
    redis{
    	host=>"172.17.0.3"
    	data_type=>"list"
    	key=>"logstash:redis"
    }
}
```
####5.logstath indexer
负责将redis中的日志写入elasticsearch
```shell
input {
        redis {
                host => "172.17.0.3"
                data_type => "list"
                key => "logstash:redis"
                type => "redis-input"
        }
}
filter {
    grok {
        match => [
            "message", "%{WORD:http_host} %{URIHOST:api_domain} %{IP:inner_ip} %{IP:lvs_ip} \[%{HTTPDATE:timestamp}\] \"%{WORD:http_verb} %{URIPATH:baseurl}(?:\?%{NOTSPACE:request}|) HTTP/%{NUMBER:http_version}\" (?:-|%{NOTSPACE:request}) %{NUMBER:http_status_code} (?:%{NUMBER:bytes_read}|-) %{QS:referrer} %{QS:agent} %{NUMBER:time_duration:float} (?:%{NUMBER:time_backend_response:float}|-)"
        ]
    }
    kv {
        prefix => "request."
        field_split => "&"
        source => "request"
    }
    urldecode {
        all_fields => true
    }
    date {
        match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
    }
}
output {
        elasticsearch {
                hosts => ["172.17.0.7:9200"]
                index => "logstash-%{+YYYY.MM.dd}"
        }
}
```
####3.访问
elasticsearch 接口地址：[http://172.17.0.7:9200/](http://172.17.0.7:9200/)
kibnana 访问地址：[http://172.17.0.9:5601/](http://172.17.0.9:5601/)