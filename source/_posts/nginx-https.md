---
title: nginx-https
date: 2016-10-5 15:36:14
tags:
- nginx
- https
categories:
- Nginx
---
####1.安装nginx、openssh

####2.生成证书
在nginx根目录下新建ssl文件夹(名字可以自己定)，并在命令窗口进入此目录，按照如下的几个命令，完成证书创建过程。
```shell
#此步用于生成私钥，会提示输入密码，密码后面步骤需要用到；itmotu.key为私钥的名字，文件名可自己定
openssl genrsa -des3 -out itmotu.key 1024
```
```shell
#此步用于生成csr证书，jason.key为上一步骤生成的私钥名，jason.csr为证书，证书文件名可自定
#在此步过程中，会交互式输入一系列的信息(所在国家、城市、组织等)，其中Common Name一项代表nginx服务访问用到的域名，我这里是本地测试，所以可以随意定一个jason.com，并在本地host文件中将此域名映射为127.0.0.1
openssl req -new -key itmotu.key -out itmotu.csr
```
```shell
#此步用于去除访问密码，如果不执行此步，在配置了ssl后，nginx启动会要求输入密码
#itmotu.key为需要密码的key，itmotu-np.key为去除访问密码的key文件
#操作过程中会要求输入密码，密码为生成key文件时的密码
openssl rsa -in itmotu.key -out itmotu-np.key
```
```shell
#此步用于生成crt证书
#itmotu.crt为第2步生成的csr证书名称，itmotu.crt为要生成的证书名称
openssl x509 -req -days 366 -in itmotu.csr -signkey itmotu-np.key -out itmotu.crt
```

经过以上几个步骤，证书生成完毕，ssl文件夹下的 itmotu.crt和 itmotu-np.key为我们后续要使用的文件。
注：在执行openssl命令时，可能会出现提示找不到openssl配置文件：
```shell
can't open config file: /etc/ssl/openssl.cnf
```
实际配置文件存在，但不在这个目录(可以在openssl安装目录找到)，我们可以直接在命令窗口设置下环境变量，使其指向正确的位置
```shell
#请根据你自己的安装目录调整
set OPENSSL_CONF=......\openssl.cfg
```
4.nginx配置ssl

打开nginx目录下conf\nginx.conf文件，找到HTTPS server的配置，将配置项前面的注释符号去掉

修改前配置内容如下：
```shell
# HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
```
修改后配置内容如下：
```shell
# HTTPS server

    server {
        listen       443 ssl;
        server_name  front;

        ssl_certificate      ./ssl/itmotu.crt;
        ssl_certificate_key  ./ssl/itmotu-np.key;

        #ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
```
5.配置重定向，http请求自动跳转到https

如果我们希望强制使用https，可以将http的请求重定向到https，只需要在http对应server配置中添加rewrite
```shell
server {
        listen       80;
        server_name  itmotu.com;
        rewrite ^(.*) https://$server_name$1 permanent;
        #省略其他配置......
}
```
