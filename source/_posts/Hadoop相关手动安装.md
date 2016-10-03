---
title: Hadoop相关手动安装
date: 2016-09-09 23:07:26
tags:
- Hadoop
- 安装
categories:
- 架构/软件
---
官方文档：[http://hadoop.apache.org/docs/current/index.html](http://hadoop.apache.org/docs/current/index.html)
####一.hadoop
1.安装
```shell
yum install -y ssh ydsh
wget http://apache.fayea.com/hadoop/common/hadoop-3.0.0-alpha1/hadoop-3.0.0-alpha1.tar.gz
tar zxvf hadoop-3.0.0-alpha1.tar.gz -C /www/itmotu

cd /www/itmotu
ln -sf hadoop-3.0.0-alpha1 hadoop
```
hadoop默认以单个节点方式运行，这样便于调试；
伪分布式配置：[http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)
2.配置
vi etc/hadoop/core-site.xml
```shell
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9001</value>
    </property>
</configuration>
```
9000端口被php占用了，改用9001

vi etc/hadoop/hdfs-site.xml
```shell
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
3.本机无密码访问配置,即`ssh localhost`无需密码
```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
4.运行
a.格式化文件系统
```shell
bin/hdfs namenode -format
```
b.启动NameNode守护进程，DataNode守护进程
```shell
sbin/start-dfs.sh
```
5.namenode web界面
http://192.168.56.110:9870/
```shell
iptables -I INPUT -p tcp --dport 9870 -j ACCEPT
service iptables save
service iptables restart
```
6.创建HDFS目录，供MapReduce job使用
```shell
bin/hdfs dfs -mkdir /user
bin/hdfs dfs -mkdir /user/justin
bin/hdfs dfs -mkdir /user/justin/input
```
7.将input文件拷贝到HDFS分布式文件系统中
```shell
bin/hdfs dfs -put etc/hadoop/*.xml /user/justin/input
```
8.运行demo程序
```shell
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha1.jar grep /user/justin/input /user/justin/output 'dfs[a-z.]+'
```
output文件夹不能是已经存在的
9.将运行结果从分布式文件系统拷贝回本地
```shell
bin/hdfs dfs -get /user/justin/output /www/web/java/hadoop
#查看结果
cat /www/web/java/hadoop/output/*
```
或直接查看HDFS中的结果
```shell
bin/hdfs dfs -cat /user/justin/output/*
```
10.关闭守护进程
```shell
sbin/stop-dfs.sh
```
11.hadoop警告`WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable` 修改
```shell
#验证
bin/hadoop checknative -a

export HADOOP_HOME=/www/itmotu/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_ROOT_LOGGER=DEBUG,console
```
####二.HBase
文档:[http://hbase.apache.org/book.html](http://hbase.apache.org/book.html)
1.安装
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/stable/hbase-1.2.2-bin.tar.gz
tar zxvf hbase-1.2.2-bin.tar.gz -C /www/itmotu/

cd /www/itmotu/
ln -sf hbase-1.2.2 hbase

```
2.单节点启动
设置 hbase.rootdir， 默认 hbase.rootdir 是指向 /tmp/hbase-${user.name} 
vi conf/hbase-site.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <!--<value>file:///tmp/hbase-${user.name}</value>-->
    <value>hdfs://localhost:9001/hbase</value>
  </property>
  <property>
    <name>hbase.master.info.port</name>
    <value>60010</value>
  </property>
</configuration>
```
启动
```shell
./bin/start-hbase.sh

#启动后可以进入shell模式进行操作
./bin/hbase shell
#退出shell
exit
```
停止hbase
```shell
./bin/stop-hbase.sh
```
3.hbase web界面
http://192.168.56.110:60010/

