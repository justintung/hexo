title: linuxmint gcc 8.3.0源码编译
date: 2019-04-08 23:01:36
tags:
- gcc
categories:
- 安装
---
1.下载编译
```
wget ftp://ftp.mirrorservice.org/sites/sourceware.org/pub/gcc/releases/gcc-8.3.0/gcc-8.3.0.tar.gz
tar zxvf gcc-8.3.0.tar.gz
cd gcc
./contrib/download_prerequisites
mkdir objdir
cd objdir
../configure --prefix=/coding/env/gcc/ --disable-multilib
make && make install
```
2.配置
```
sudo update-alternatives --install /usr/bin/gcc gcc /coding/env/gcc/bin/gcc 120

 update-alternatives --config gcc
```