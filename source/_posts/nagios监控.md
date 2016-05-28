title: nagios监控
date: 2016-05-8 16:21:36
tags:
- nagios
- 监控
categories:
- 架构/软件
---
官方网址：[https://www.nagios.org/](https://www.nagios.org/)
####服务端
####1.添加用户、用户组
```shell
useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
```
####2.安装nagios core
```shell
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz
tar zxvf nagios-4.1.1.tar.gz
cd nagios-4.1.1/
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode

cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
/etc/init.d/nagios start
```
Nagios主程序只是提供一个运行框架，其具体监控是靠运行在其下的插件完成的
####3.安装nagios plugins
```shell
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar zxvf nagios-plugins-2.1.1.tar.gz
cd nagios-plugins-2.1.1/
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make && make install
```
####4.NRPE(Nagios Remote Plugin Executor)
由于Nagios只能监测自己所在的主机的一些本地情况,例如，cpu负载、内存使用、硬盘使用等等。如果想要监测被监控的服务器上的这些本地情况，就要用到NRPE。NRPE(Nagios Remote Plugin Executor)是Nagios的一个扩展，它被用于被监控的服务器上，向Nagios监控平台提供该服务器的一些本地的情况。NRPE可以称为 Nagios的Linux客户端。在Nagios服务器端只要安装NPRE监控插件就行。
下载地址：[https://sourceforge.net/projects/nagios/files/nrpe-2.x/](https://sourceforge.net/projects/nagios/files/nrpe-2.x/)
```shell
tar zxvf nrpe-2.15.tar.gz
cd nrpe-2.15
./configure
make all
make install-plugin
```
####5.开启端口5666
```shell
iptables -I INPUT -p tcp --dport 5666 -j ACCEPT
service iptables save
service iptables restart
```
5666端口为NRPE使用的端口
####6.关闭selinux防火墙
```shell
vi /etc/sysconfig/selinux
SELINUX=disabled
```
将selinux设置为disabled后重启
或使用
```shell
setenforce 0
```
无需重启
selinux防火墙会造成出现”unable to get process status“的错误
####7.nagios登陆账号密码设置
```shell
/usr/local/apache/bin/htpasswd -nb justin 3344520 > /usr/local/nagios/etc/htpasswd.users
chown nagios.nagios /usr/local/nagios/etc/htpasswd.users
```
未安装apache的，可以使用在线htpasswd生成工具，如(http://tool.oschina.net/htpasswd)[http://tool.oschina.net/htpasswd]
####8.nginx配置
nagios为cgi脚本，nginx默认不支持，需要另外配置
#####a.cgi配置
```shell
yum install -y fcgi fcgi-devel spawn-fcgi
git clone --depth=1 https://github.com/gnosek/fcgiwrap.git
cd fcgiwrap
autoreconf -i
./configure
make && make install

vi /etc/sysconfig/spawn-fcg
```
内容如下
```shell
#SOCKET=/var/run/php-fcgi.sock
#OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"
OPTIONS="-p 8081 -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/local/sbin/fcgiwrap"

```
启动spawn-fcgi
```shell
service spawn-fcgi start
```
#####b.nginx配置
```shell
server {
    listen       80;
    server_name  nagios.itmotu.com;
    location / {
    root   /usr/local/nagios/share;
    index  index.html index.htm index.php;
    }
    location ~ .*\.(php|php5)?$ {
    root /usr/local/nagios/share;
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
    }
    location /nagios {
        alias /usr/local/nagios/share;
    }
    location /cgi-bin/images {
        alias /usr/local/nagios/share/images;
    }
    location /cgi-bin/stylesheets {
        alias /usr/local/nagios/share/stylesheets;
    }
    location /cgi-bin {
        alias /usr/local/nagios/sbin;
    }
    location ~ .*\.(cgi|pl)?$ {
    root       /usr/local/nagios/sbin;
    auth_basic "Nagios Access";
    auth_basic_user_file /usr/local/nagios/etc/htpasswd.users;
    rewrite   ^/nagios/cgi-bin/(.*)\.cgi /$1.cgi break;
    fastcgi_pass  127.0.0.1:8081;
    fastcgi_index index.cgi;
    include fastcgi.conf;
    }
}
```

####客户端（被监控服务器）
安装基本类似于服务端
1.NRPE(Nagios Remote Plugin Executor)
监控服务器通过NRPE与被监控服务器通信
+安装
```shell
make install-plugin #客户端可以不安装
make install-daemon
make install-daemon-config
```

启动
```shell
/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```
将上面语句添加到/etc/rc.local可实现开机启动

检测是否正确启动
```shell
/usr/local/nagios/libexec/check_nrpe -H 127.0.0.1
```
输出NRPE 版本，则表示安装正确

2.配置监控项、允许监控本服务器的机器（即服务端服务器）
```shell
vi /usr/local/nagios/etc/nrpe.cfg
```
服务端IP设置
```shell
allowed_hosts=127.0.0.1,192.168.56.101
```
/usr/local/nagios/etc/nrpe.cfg文件中的的command项为，服务端可通过NRPE命令(服务端需要手动添加该命令的define,参考服务端配置)调用的监控命令(服务端没有nrpe.cfg文件)，如：
a.监控交换分区的使用情况，使用超过20％时为警告状态，超过10％时为严重状态
```shell
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
```
b.监控根分区磁盘使用情况
```shell
command[check_disk_root]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /
```
添加后需要重启NRPE
```shell
killall nrpe
/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```

####服务端其他配置
1.添加command nrpe
```shell
vi /usr/local/nagios/etc/objects/commands.cfg
```
```shell
# 'check_nrpe ' command definition
define command{
	command_name check_nrpe
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```
服务端使用check_nrpe命令,与客户端的nrpe demon通信，服务端check_nrpe可调用的命令,定义在客户端的nrpe.cfg文件中
2.
```shell
cp /usr/local/nagios/etc/objects/localhost.cfg  /usr/local/nagios/etc/objects/192.168.56.110.cfg
```
修改文件中的IP配置为192.168.56.110，并注释掉hostgroup定义，在各个“check_command”内容前面添加“check_nrpe!”，如下：
```shell
#define hostgroup{  
#        hostgroup_name  linux-servers  
#        alias           Linux Servers  
#        members         192.168.56.110  
#}
define service{
    use local-service         ; Name of service template to use
    host_name 192.168.56.110
    service_description PING
    check_command check_nrpe!check_ping!100.0,20%!500.0,60%
}

```
3.
```shell
vi /usr/local/nagios/etc/nagios.cfg
```
添加一行
```shell
cfg_file=/usr/local/nagios/etc/objects/192.168.56.110.cfg
```
4.重启服务
```shell
/etc/init.d/nagios restart
```
#### nagios auth用户设置
1.服务端、客户端都开启
/usr/local/nagios/etc/cgi.cfg
```shell
use_authentication=1
```
2.服务端cgi.cfg添加htpasswd生成的用户
```shell
default_user_name=justin
authorized_for_system_information=nagiosadmin,justin
authorized_for_configuration_information=nagiosadmin,justin
authorized_for_system_commands=nagiosadmin,justin
authorized_for_all_services=nagiosadmin,justin
authorized_for_all_hosts=nagiosadmin,justin
authorized_for_all_service_commands=nagiosadmin,justin
authorized_for_all_host_commands=nagiosadmin,justin
```
即原来有nagiosadmin的后面追加用户名，default_user_name要设置，不然登陆web后看到的是空白
3.重启nagios
```shell
service nagios restart
```


####服务启动
```shell
service spawn-fcgi start
/etc/init.d/nagios start
/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```
