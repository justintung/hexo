---
title: varnish安装
date: 2016-05-28 01:02:52
tags:
- varnish
- 安装
categories:
- 架构/软件
---
```shell
yum install -y python-docutils pcre-devel readline readline-devel libtool

ln -sf /usr/local/lib/libjemalloc* /usr/local/lib64/

wget https://repo.varnish-cache.org/source/varnish-4.1.2.tar.gz
tar zxvf varnish-4.1.2.tar.gz
cd varnish-4.1.2
./autogen.sh
./configure --prefix=/www/itmotu/varnish-4.1.2 --enable-dependency-tracking --enable-debugging-symbols --enable-developer-warnings

make && make install
```