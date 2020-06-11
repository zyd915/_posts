---
title: 大数据集群搭建 (三)、HBase三节点分布式集群搭建
date: 2020-06-11 08:46:15
updated: 2020-06-11 08:46:18
tags: 
    - bigdata
    - hbase
categories: hbase
thumbnail: https://static.studytime.xin/article/DY3l0WUykno.jpg
toc: true
excerpt: HBase是建立在Hadoop文件系统之上的分布式面向列的数据库。它是一个开源项目，是横向扩展的。
---

### 下载HBase的压缩包
[http://archive.cloudera.com/cdh5/cdh/5/](http://archive.cloudera.com/cdh5/cdh/5/)

我们在这个网址下载我们使用的zk版本为[hbase-1.2.0-cdh5.14.2.tar.gz](http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.14.2.tar.gz)

### 解压HBase
node01执行解压命令，将HBase的压缩包解压到node01服务器的/opt/module/路径下去，然后准备进行安装。
```
cd /opt/software/
tar -zxvf hbase-1.2.0-cdh5.14.2.tar.gz.tar.gz  -C /opt/module/
```

### 修改node01配置文件

#### hbase-env.sh 配置
```
cd /opt/module/hbase-1.2.0-cdh5.14.2/conf/

vim hbase-env.sh

修改如下两项内容，值如下
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64
export HBASE_MANAGES_ZK=false
```

#### hbase-site.xml 配置
```
vim hbase-site.xml

<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://node01:9000/hbase</value>  
	</property>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
	<property>
		<name>hbase.master.port</name>
		<value>16000</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>node01,node02,node03</value>
	</property>
    <!-- 此属性可省略，默认值就是2181 -->
	<property>
		<name>hbase.zookeeper.property.clientPort</name>
		<value>2181</value>
	</property>
	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>/kkb/install/zookeeper-3.4.5-cdh5.14.2/zkdatas</value>
	</property>
    <!-- 此属性可省略，默认值就是/hbase -->
	<property>
		<name>zookeeper.znode.parent</name>
		<value>/hbase</value>
	</property>
</configuration>
```

#### regionservers
```
vim vim regionservers

# 指定HBase集群的从节点；原内容清空，添加如下三行
node01
node02
node03
```

#### cback-masters

创建back-masters配置文件，里边包含备份HMaster节点的主机名，每个机器独占一行，实现HMaster的高可用
```
vim backup-masters
将node02作为备份的HMaster节点，文件内容如下：
node02
```

### 创建软连接

因为HBase集群需要读取hadoop的core-site.xml、hdfs-site.xml的配置文件信息，所以我们三台机器都要执行以下命令，在相应的目录创建这两个配置文件的软连接。
```
ln -s /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop/core-site.xml  /opt/module/hbase-1.2.0-cdh5.14.2/conf/core-site.xml

ln -s /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop/hdfs-site.xml  /opt/module/hbase-1.2.0-cdh5.14.2/conf/hdfs-site.xml
```
执行完后，出现如下效果，以node01为例
```
[hadoop@node01 conf]$ ll
总用量 44
-rw-rw-r--. 1 hadoop hadoop    7 6月  10 23:34 backup-masters
lrwxrwxrwx. 1 hadoop hadoop   59 6月  10 23:35 core-site.xml -> /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop/core-site.xml
-rw-r--r--. 1 hadoop hadoop 1811 3月  28 2018 hadoop-metrics2-hbase.properties
-rw-r--r--. 1 hadoop hadoop 4603 3月  28 2018 hbase-env.cmd
-rw-r--r--. 1 hadoop hadoop 7535 6月  10 23:32 hbase-env.sh
-rw-r--r--. 1 hadoop hadoop 2257 3月  28 2018 hbase-policy.xml
-rw-r--r--. 1 hadoop hadoop 1812 6月  10 23:33 hbase-site.xml
lrwxrwxrwx. 1 hadoop hadoop   59 6月  10 23:35 hdfs-site.xml -> /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop/hdfs-site.xml
-rw-r--r--. 1 hadoop hadoop 4603 3月  28 2018 log4j.properties
-rw-r--r--. 1 hadoop hadoop   21 6月  10 23:33 regionservers
```

### 安装包分发
在node01服务器上执行以下命令：
```
scp -r /opt/module/hbase-1.2.0-cdh5.14.2/ node02:/opt/module/
scp -r /opt/module/hbase-1.2.0-cdh5.14.2/ node03:/opt/module/
```


### 添加HBase环境变量
分别在zookeeper集群每台服务器上，node01、node02、node03上修改配置~/.bash_profile文件。
```
vim ~/.bash_profile

export HBASE_HOME=/opt/module/hbase-1.2.0-cdh5.14.2
export PATH=$PATH:$HBASE_HOME/bin

三台节点，让新添环境变量生效（hadoop用户下执行）
source ~/.bash_profile
```

### HBase的启动与停止
启动HBase前需要预先启动HDFS及ZooKeeper集群。

在node01服务器上启动HBase集群：

```
start-hbase.sh

# 启动完后，jps查看HBase相关进程
[hadoop@node01 conf]$ xcall jps
--------- node01 ----------
4576 SecondaryNameNode
4946 NodeManager
4835 ResourceManager
4164 NameNode
42437 QuorumPeerMain
82649 Jps
60651 HRegionServer
4399 DataNode
--------- node02 ----------
4032 DataNode
4273 NodeManager
77632 Jps
57190 HRegionServer
40028 QuorumPeerMain
--------- node03 ----------
4818 DataNode
5076 NodeManager
41287 QuorumPeerMain
78573 Jps
```
可以看到，node01、node02上有进程HMaster、HRegionServer，node03上有进程HRegionServer



特殊说明：HBase启动的时候会产生一个警告，这是因为jdk7与jdk8的问题导致的，如果linux服务器安装jdk8就会产生。
```
[hadoop@node01 ~]$ start-hbase.sh
starting master, logging to /opt/module/hbase-1.2.0-cdh5.14.2/logs/hbase-hadoop-master-node01.out
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
node02: starting regionserver, logging to /opt/module/hbase-1.2.0-cdh5.14.2/bin/../logs/hbase-hadoop-regionserver-node02.out
node03: starting regionserver, logging to /opt/module/hbase-1.2.0-cdh5.14.2/bin/../logs/hbase-hadoop-regionserver-node03.out
node01: starting regionserver, logging to /opt/module/hbase-1.2.0-cdh5.14.2/bin/../logs/hbase-hadoop-regionserver-node01.out
node02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
node02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
node03: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
node03: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
node01: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
node01: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
node02: starting master, logging to /opt/module/hbase-1.2.0-cdh5.14.2/bin/../logs/hbase-hadoop-master-node02.out
node02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
node02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0

```

可以注释掉所有机器的hbase-env.sh当中的,“HBASE_MASTER_OPTS”和“HBASE_REGIONSERVER_OPTS”配置 来解决这个问题。不过警告不影响我们正常运行，可以不用解决.

### 单节点启动相关进程
```
#HMaster节点上启动HMaster命令
hbase-daemon.sh start master

#启动HRegionServer命令
hbase-daemon.sh start regionserver
```

### 访问WEB页面
[http://node01:60010](http://node01:60010)


### 停止HBase集群
在node01上运行。
```
stop-hbase.sh
```
若需要关闭虚拟机，则还需要关闭ZooKeeper、Hadoop集群

