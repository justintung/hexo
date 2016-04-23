title: JDK安装
date: 2016-04-23 13:14:36
tags:
- JDK
- 安装
---
A. 下载源码  
```
wget http://download.oracle.com/otn-pub/java/jdk/8u51-b16/jdk-8u51-linux-x64.tar.gz?AuthParam=1437533458_2510c1c12dfef1e6892ce4a2266d90a0
mv jdk-8u51-linux-x64.tar.gz?AuthParam=1437533458_2510c1c12dfef1e6892ce4a2266d90a0 jdk-8u51-linux-x64.tar.gz
```
B.安装
```
mkdir -p /usr/local/java/
tar zxvf jdk-8u51-linux-x64.tar.gz -C /usr/local/java/
```

要配置JAVA_HOME、CLASSPATH

CDN下载: [https://www.reucon.com/cdn/java/](https://www.reucon.com/cdn/java/)  
