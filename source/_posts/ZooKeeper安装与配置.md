---
title: ZooKeeper安装与配置
date: 2016-10-03 13:28:32
tags:
- 大数据
- 工具
categories:
- 架构/软件
---
####一.ZooKeeper是什么
ZooKeeper是Hadoop的正式子项目，它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户
官方网址：[http://zookeeper.apache.org/](http://zookeeper.apache.org/)

####二. 单机安装、配置：
1. 下载zookeeper二进制安装包
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
```
2. 解压zookeeper安装包
```shell
tar zxvf zookeeper-3.4.9.tar.gz -C /usr/local/
cd /usr/local/
ln -sf zookeeper-3.4.9 zookeeper
yum install -y nc net-tools
```
3. 设置环境变量
```shell
vi /etc/bashrc
#设置添加内容
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:ZOOKEEPER_HOME/bin
#重新加载
source /etc/bashrc
```
4. 配置
配置文件存放在$ZOOKEEPER_HOME/conf/目录下，将zoo_sample.cfd文件名称改为zoo.cfg,  缺省的配置内容如下：
```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/tmp/zookeeper #快照数据保存位置，tmp目录数据重启后会消失
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```
配置说明：
tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

5. 启动zookeeper
当这些配置项配置好后，你现在就可以启动zookeeper了：
```shell
cd /usr/local/zookeeper/bin
./zkServer.sh start #启动 
jps #查看启动的服务名称
./zkServer.sh stop #关闭
```

#### 三.集群模式
参考：[http://blog.csdn.net/zhll3377/article/details/45041691](http://blog.csdn.net/zhll3377/article/details/45041691)
1,2,3步同单机模式

4.修改Zookeeper的配置文件
首先将/home/hadoop/zookeeper-3.4.6/conf/zoo_sample.cfg文件复制一份，并更名为zoo.cfg，如下所示：
```shell
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial   
# synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between   
# sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored.  
# do not use /tmp for storage, /tmp here is just   
# example sakes.  
dataDir=/home/hadoop/zk/data  
dataLogDir=/home/hadoop/zk/log  
# the port at which the clients will connect  
clientPort=2181  
# the maximum number of client connections.  
# increase this if you need to handle more clients  
#maxClientCnxns=60  
#  
# Be sure to read the maintenance section of the   
# administrator guide before turning on autopurge.  
#  
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance  
#  
# The number of snapshots to retain in dataDir  
#autopurge.snapRetainCount=3  
# Purge task interval in hours  
# Set to "0" to disable auto purge feature  
#autopurge.purgeInterval=1  
server.1=Master:3333:4444  
server.2=Slave1:3333:4444  
server.3=Slave2:3333:4444 
```
eg:
```shell
 initLimit=5 
 syncLimit=2 
 server.1=192.168.211.1:2888:3888 
 server.2=192.168.211.2:2888:3888
```
server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
 
根据dataDir和dataLogDir变量创建相应的目录。
 
5、创建myid文件
在dataDir目录下创建一个myid文件，然后分别在myid文件中存储（按照zoo.cfg文件的server.A中）A的数值，在不同机器上的该文件中填写相应的值。
 
6、启动Zookeeper
执行命令“zkServer.sh start”将会启动Zookeeper。在此大家需要注意，和在Master启动Hadoop不同，不同节点上的Zookeeper需要单独启动。而执行命令“zkServer.sh stop”将会停止Zookeeper。
 
开发人员可以使用命令“JPS”查看Zookeeper是否成功启动，以及执行命令“zkServer.sh status”查看Zookeeper集群状态，如下所示：
```shell
#192.168.1.224  
JMX enabled by default  
Using config: /home/hadoop/zookeeper-3.4.6/bin/../conf/zoo.cfg  
Mode: follower  
  
#192.168.1.225  
JMX enabled by default  
Using config: /home/hadoop/zookeeper-3.4.6/bin/../conf/zoo.cfg  
Mode: leader  
  
#192.168.1.226  
JMX enabled by default  
Using config: /home/hadoop/zookeeper-3.4.6/bin/../conf/zoo.cfg  
Mode: follower  
```
Zookeeper集群在启动的过程中，查阅zookeeper.out，会有如下异常：
```shell
java.net.ConnectException: Connection refused  
        at java.net.PlainSocketImpl.socketConnect(Native Method)  
        at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:339)  
        at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:200)  
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:182)  
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)  
        at java.net.Socket.connect(Socket.java:579)  
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:368)  
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.toSend(QuorumCnxManager.java:341)  
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.process(FastLeaderElection.java:449)  
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.run(FastLeaderElection.java:430)  
        at java.lang.Thread.run(Thread.java:745)  
```
上述异常可以忽略，因为集群环境中某些子节点还没有启动zookeeper。