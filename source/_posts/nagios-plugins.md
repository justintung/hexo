title: nagios plugins
date: 2016-05-14 16:21:36
tags:
- nagios
- 监控
- plugins
categories:
- 架构/软件
---
官方网址：[https://exchange.nagios.org/directory/Plugins](https://exchange.nagios.org/directory/Plugins)
1.check_website_content
地址：[https://exchange.nagios.org/directory/Plugins/Web-Servers/check_website_content-%28edouard-2Elamoine%29/details](https://exchange.nagios.org/directory/Plugins/Web-Servers/check_website_content-%28edouard-2Elamoine%29/details)

a.下载可执行的命令脚本check_website_content
将命令文件下载到服务端 /usr/local/nagios/libexec/,并修改相应的权限

b.定义command
在服务端文件/usr/local/nagios/etc/objects/commands.cfg末尾添加command
```shell
define command{
    command_name    check_website_content
    command_line    $USER1$/check_website_content -u $ARG1$ -c $ARG2$
}
```
c.添加服务
编辑文件 /usr/local/nagios/etc/objects/192.168.56.110.cfg
```shell
define service{
    use                             local-service
    host_name                       192.168.56.110
    service_description             test.itmotu.com
    check_command           check_website_content!test.itmotu.com/check_website_content.php!ok
}
服务用于检测test.itmotu.com/check_website_content.php是否正确输出ok文字
```
d.重启nagios
```shell
service nagios restart
```
check_website_content脚本如下：
```shell
#!/bin/bash

#STATE_OK=0
#STATE_WARNING=1
#STATE_CRITICAL=2
#STATE_UNKNOWN=3

type curl >/dev/null 2>&1 || { echo >&2 "This plugin require curl but it's not installed."; exit 3; }

while getopts u:c: option
do
case "${option}"
in
u) URL=${OPTARG};;
c) CONTENT=${OPTARG};;
esac
done

if [[ ! $URL || ! $CONTENT ]]; then
echo "Error: please specify URL to check and word content to search in. ex: ./check_website_content -u mywebsite.com/welcome.html -c specificwordinthepage"
exit 3
fi

WEB=$((curl $URL) 2>/dev/null )

if [[ $WEB == *$CONTENT* ]]
then
  echo "OK - $URL succesfully checked"
 exit 0
fi

echo "CRITICAL - $URL don't match with expected content"
exit 2
```