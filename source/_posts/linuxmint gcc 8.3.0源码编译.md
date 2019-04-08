title: linuxmint gcc 8.3.0源码编译
date: 2019-04-08 23:01:36
tags:
- gcc
categories:
- 安装
---
1.下载编译
```
wget  
tar zxvf gcc.tar.gz
cd gcc
mkdir objdir
cd objdir
../configure --prefix=/coding/env/gcc --disable-
make && make install
```
2.配置
```
sudo update-alternatives --install /usr/bin/gcc gcc /coding/env/gcc/bin/gcc 120

alternatives --config gcc
```