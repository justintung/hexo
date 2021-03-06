title: SVN
date: 2016-05-7 12:20:36
tags:
- SVN
categories:
- 安装
---
1.安装
```shell
yum install -y subversion
```
2.创建svn仓库地址
```shell
mkdir -p /www/svnrepos
```
3.配置各个仓库共用的passwd
```shell
vi /www/svnrepos/passwd

#内容如下
justin=3344520
dongdong=3344520
……
```
格式为：账号=密码
4.新建仓库test
```shell
svnadmin create /www/svnrepos/test
```
5.设置仓库test的用户权限
```shell
vi /www/svnrepos/test/conf/svnserve.conf

#内容如下
anon-access = none
auth-access = write
password-db = /www/svnrepos/passwd
authz-db = authz
realm = /www/svnrepos/test
```
用户目录权限
```shell
vi /www/svnrepos/test/conf/authz

#添加下面内容
[/]
justin=rw
[/html]
dongdong=rw
```
6.服务启动
```shell
/usr/bin/svnserve -d -r /www/svnrepos
```
或修改`/etc/init.d/svnserve`文件，如下：
```shell
#args="--daemon --pid-file=${pidfile} $OPTIONS"
args="--daemon --pid-file=${pidfile} -r /www/svnrepos/"
```
7.svn自动部署
```shell
cd /www/svnrepos/test/hooks/
cp post-commit.tmpl post-commit
chmod a+x post-commit
vi post-commit

#修改内容如下
REPOS="$1"
REV="$2"

BASEPATH=/www/web/test
WEBPATH="$BASEPATH/"
export LANG=zh_CN.UTF-8

for dir in `svnlook dirs-changed /www/svnrepos/test/`
do
    svn update -N $WEBPATH/$dir --username justin --password 3344520 --no-auth-cache
done
#mailer.py commit "$REPOS" "$REV" /path/to/mailer.conf
```
重启服务
```shell
service svnserve restart

#或
killall svnserve
/usr/bin/svnserve -d -r /www/svnrepos
```
8.提交时强制输入注释
```shell
REPOS="$1"
TXN="$2"

SVNLOOK=/usr/bin/svnlook
LOGMSG=$($SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c)
if [ "$LOGMSG" -lt 2 ]; then
    echo -e "\n 提交文件时必须添加注释，提交中止." 1>&2
    exit 1
fi

exit 0
```