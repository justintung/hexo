title: Golang 安装
date: 2016-04-23 14:30:17
tags:
- Golang
- 安装
---
A. 下载源码
```
wget http://www.golangtc.com/static/go/go1.4.2.src.tar.gz
```
B. 安装需要重新
```
yum install -y mercurial git bison
```
C. 解压
```
tar zxvf go1.4.2.src.tar.gz -C /www/itmotu/
mv go go1.4.2
```
D. 编译
 ```
cd /www/itmotu/go1.4.2/src
./all.bash
ln -sf /www/itmotu/go1.4.2/ /www/itmotu/go
```
从已经编译好的包安装
```
tar zxvf go1.5.2.linux-amd64.tar.gz -C /www/itmotu/go
vi /etc/profile
```
添加下面内容：
```
#export JAVA_HOME=/usr/local/java/jdk1.8.0_51
#export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export GOROOT=/www/itmotu/go
export GOPATH=/www/web/go
export GOARCH=amd64
export GOOS=linux
export GOBIN=$GOPATH/bin
export GOTOOLS=$GOROOT/pkg/tool

export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$GOBIN:$GOTOOLS:$GOROOT/bin
#alias docker='sudo docker'
```
执行：
source /etc/profile  
建议添加在 /etc/rc.d/rc.local

