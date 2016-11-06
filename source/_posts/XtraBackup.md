---
title: XtraBackup
date: 2016-10-19 11:36:14
tags:
- XtraBackup
- mysql备份
categories:
- Mysql
---
####1.库安装
```shell
yum install -y libev numactl perl perl-DBD-MySQL
```
2.下载XtraBackup
官方地址：[https://www.percona.com/downloads/XtraBackup/](https://www.percona.com/downloads/XtraBackup/)
```shell
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.4/binary/redhat/6/x86_64/percona-xtrabackup-24-2.4.4-1.el6.x86_64.rpm
```
3.配置
```shell
#1.备份地址
mkdir -p /www/data/backup

#2.创建专门的备份用户
备份数据库的用户需要具有相应权限，如果要使用一个最小权限的用户进行备份，则可基于如下命令创建此类用户：

mysql> CREATE USER ‘bkpuser'@'localhost’ IDENTIFIED BY ‘newpasswd’; 
mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM ‘bkpuser’; 
mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO ‘bkpuser’@’localhost’; 
mysql> FLUSH PRIVILEGES;

#3 sql开启 log-bin
在/etc/my.cnf文件[mysqld]下修改
```shell
[mysqld]
server-id=1
log-bin=mysql-bin
```
4.全站备份
```shell
#1全备份
innobackupex --host 127.0.0.1 --user=root --password=3344520 /www/data/backup/

#2，一致性处理（2016-11-04_03-38-21为上一步生成的备份目录，）
innobackupex --apply-log /www/data/backup/2016-11-04_03-38-21/

#3增量备份二进制文件mysql-bin.000002和154，应该查看/www/data/backup/2016-11-04_03-38-21/目录下的xtrabackup_binlog_info文件
mysqlbinlog --start-position=154 /www/data/mysql-5.7.11/mysql-bin.000002 > `date +%F`.sql

```
在步骤1后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据文件仍处理不一致状态。“准备”的主要作用正是通过回滚未提交的事务及同步已经提交的事务至数据文件也使得数据文件处于一致性状态。所以需要进行步骤2。

5.还原
假设mysql数据存储目录为/www/data/mysql-5.7.11/
```shell
#1.删除错误数据(为了测试)
cd /www/data/mysql-5.7.11/
rm -rf *

#2.还原（先关闭mysql）
innobackupex --datadir=/www/data/mysql-5.7.11/ --copy-back /www/data/backup/2016-11-04_03-38-21/

#3使用log-bin文件还原，只是为了举例旧方法
mysql> set sql_log_bin=0;
mysql> SOURCE /www/data/mysql-5.7.11/2016-11-04_03-38-21/2016-11-04.sql
mysql> set sql_log_bin=1; 

#4.修改还原数据的权限
chown mysql:mysql -R *

#5.重启mysql
service mysql restart
```
6.使用innobackupex进行增量备份
前面我们进行增量备份时，使用的还是老方法：备份二进制日志。其实xtrabackup还支持进行增量备份
```shell
#fulldata_dir为完全备份数据所在目录，incdata_dir为增量数据所在目录
innobackupex --incremental incdata_dir --incremental-basedir=fulldata_dir
```
步骤
```shell
#1.进行全量备份，同上

#2.增量备份
innobackupex  --host 127.0.0.1 --user=root --password=3344520  --incremental /www/data/backup/inc --incremental-basedir=/www/data/backup/2016-11-04_03-38-21/

#3.整理,INCREMENTAL-DIR-n为增量备份数据
innobackupex --apply-log --redo-only /www/data/backup/2016-11-04_03-38-21/
innobackupex --apply-log --redo-only /www/data/backup/2016-11-04_03-38-21/ --incremental-dir=INCREMENTAL-DIR-1
innobackupex --apply-log --redo-only /www/data/backup/2016-11-04_03-38-21/ --incremental-dir=INCREMENTAL-DIR-2
...

#4.还原
innobackupex --datadir=/www/data/mysql-5.7.11/ --copy-back /www/data/backup/2016-11-04_03-38-21/

#5.修改还原数据的权限
chown mysql:mysql -R *

#6.重启mysql
service mysql restart
```