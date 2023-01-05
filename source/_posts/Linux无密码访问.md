title: Linux无密码访问
date: 2016-05-14 18:21:36
tags:
- Linux
- 无密码访问
categories:
- Linux
---
免密ssh(A可以免密访问B):

1. 在A执行: ssh-keygen -t rsa
2. 在A执行: ssh-copy-id -i ~/.ssh/id_rsa.pub B服务器上的某用户名@B服务器ip
  这样就能在A 通过“B服务器上的某用户名”免密登录B; 