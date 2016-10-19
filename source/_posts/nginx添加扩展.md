---
title: nginx添加扩展
date: 2016-10-19 11:36:14
tags:
- nginx
- nginx扩展
categories:
- Nginx
---
####1.查找旧编译参数
```shell
cd /www/itmotu/nginx-1.9.12/sbin
./nginx -V

#得到输出
nginx version: nginx/1.9.12
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/www/itmotu/nginx-1.9.12 --with-http_stub_status_module --with-http_ssl_module
```

####2.下载扩展
```shell
cd /root/soft
svn checkout http://code.taobao.org/svn/nginx_concat_module/
```
####3.编译
```shell
./configure --user=www --group=www --prefix=/www/itmotu/nginx-1.9.12 --with-http_stub_status_module --with-http_ssl_module --add-module=/root/soft/nginx_concat_module/trunk/

make
```
此处注意，因为nginx已经安装，不需要make install
####4.拷贝nginx
因为nginx已经安装，不需要make install
```shell
service nginxd stop
cp /www/itmotu/nginx-1.9.12/sbin/nginx /www/itmotu/nginx-1.9.12/sbin/nginx.bak

cp objs/nginx /www/itmotu/nginx-1.9.12/sbin/
service nginxd start
```
####4.使用扩展
依据扩展的不同，进行配置