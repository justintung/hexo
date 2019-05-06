title: linux 24小时制修改
date: 2019-05-06 17:01:36
tags:
- linux
categories:
- 配置
---
1.执行命令
```shell```
tzselect
```
选择 Asia -> China -> Beijing Time

2.更新
```shell```
rm /etc/localtime
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```