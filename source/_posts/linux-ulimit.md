title: Linux ulimit
date: 2016-11-7 10:20:36
tags:
- ulimit
- Linux
categories:
- Linux
---
ulimit -a 用来显示当前的各种用户进程限制。
Linux对于每个用户，系统限制其最大进程数。为提高性能，可以根据设备资源情况，设置各linux 用户的最大进程数。

修改/etc/security/limits.conf文件。
```shell
*        soft    nproc 10240
*        hard    nproc 10240
*        soft    nofile 10240
*        hard    nofile 10240
```
注意：每行前面需要带有*号
直接在使用命令ulimit -n 10240，修改的是当前客户端的限制，对其他用户应用没有作用
修改/etc/security/limits.conf后，退出命令行，重新登录，对所有都有效