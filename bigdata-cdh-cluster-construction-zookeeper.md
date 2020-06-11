---
title: 大数据集群搭建 (二)、zookeeper三节点分布式集群搭建
date: 2020-06-10 13:20:50
updated: 2020-06-10 13:20:52
tags: 
    - bigdata
    - zookeeper
categories: zookeeper
thumbnail: https://static.studytime.xin/article/ASv0ABlBuAc.jpg
toc: true
excerpt: zookeeper集群搭建指的是ZooKeeper分布式模式安装。通常由2n+1台servers组成。这是因为为了保证Leader选举（基于Paxos算法的实现）能够得到多数的支持，所以ZooKeeper集群的数量一般为奇数。
---

zookeeper集群搭建指的是ZooKeeper分布式模式安装。通常由2n+1台servers组成。这是因为为了保证Leader选举（基于Paxos算法的实现）能够得到多数的支持，所以ZooKeeper集群的数量一般为奇数。

### 下载zookeeeper的压缩包
[http://archive.cloudera.com/cdh5/cdh/5/](http://archive.cloudera.com/cdh5/cdh/5/)

我们在这个网址下载我们使用的zk版本为[zookeeper-3.4.5-cdh5.14.2.tar.gz](http://archive.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.14.2.tar.gz)

### 解压zookeeeper
node01执行解压命令，将zookeeper的压缩包解压到node01服务器的/opt/module/路径下去，然后准备进行安装。
```
cd /opt/software/
tar -zxvf zookeeper-3.4.5-cdh5.14.2.tar.gz  -C /opt/module/
```

### 修改node01配置文件
```
cd /opt/module/zookeeper-3.4.5-cdh5.14.2/conf
cp zoo_sample.cfg zoo.cfg
mkdir -p /opt/module/zookeeper-3.4.5-cdh5.14.2/zkdatas

# 修改属性配置
vim  zoo.cfg

# 配置如下
dataDir=/opt/module/zookeeper-3.4.5-cdh5.14.2/zkdatas
autopurge.snapRetainCount=3
autopurge.purgeInterval=1

#文件末尾增加如下三行
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
```

### 安装包分发
在node01服务器上执行以下命令：
```
scp -r /opt/module/zookeeper-3.4.5-cdh5.14.2/ node02:/opt/module/
scp -r /opt/module/zookeeper-3.4.5-cdh5.14.2/ node03:/opt/module/
```

### 修改myid的值
分别在zookeeper集群每台服务器上，node01、node02、node03上修改myid值。
```
node01 server:
echo 1 >  opt/module/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid

node02 server:
echo 2 >  opt/module/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid

node03 server:
echo 3 >  opt/module/zookeeper-3.4.5-cdh5.14.2/zkdatas/myid
```

### 配置环境变量
分别在zookeeper集群每台服务器上，node01、node02、node03上修改配置~/.bash_profile文件。
```
vim ~/.bash_profile

export ZK_HOME=/opt/module/zookeeper-3.4.5-cdh5.14.2
export PATH=$PATH:$ZK_HOME/bin

三台节点，让新添环境变量生效（hadoop用户下执行）
source ~/.bash_profile
```

### 三台机器启动zookeeper服务
分别在zookeeper集群每台服务器上，node01、node02、node03上启动zookeeper。
```
zkServer.sh start
```

### 查看启动状态
分别在zookeeper集群每台服务器上运行。
```
zkServer.sh status

[hadoop@node01 software]$ zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.4.5-cdh5.14.2/bin/../conf/zoo.cfg
Mode: follower


[hadoop@node02 ~]$ zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.4.5-cdh5.14.2/bin/../conf/zoo.cfg
Mode: leader

[hadoop@node03 ~]$ zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.4.5-cdh5.14.2/bin/../conf/zoo.cfg
Mode: follower
```

zkServer的状态要么是follower，要么是leader。三个节点中，一个节点为leader，另外两个为follower

### 如何关闭zookeeper集群
分别在zookeeper集群每台服务器上运行。
```
zkServer.sh stop
```
### 特殊说明：
三台机器一定要保证时钟同步。
关闭虚拟机前，要在每个zookeeper服务器中使用`zkServer.sh stop`命令，关闭zookeeper服务器，否则，可能集群出问题。



