---
title: Hive 系列文章（三）Hive高可用部署 HiveServer2高可用及Metastore高可用
permalink: hive-knowledge-install-ha
date: 2021-07-01 18:47:13
updated: 2021-07-01 18:47:43
tags:
    - 大数据
    - hive
categories: [大数据,hive]
keywords: [HiveServer2高可用, Metastore高可用, Hive高可用，如何实现hive高可用]
toc: true
thumbnail: https://static.studytime.xin//studytime/image/articles/F41ckt.jpg
excerpt: 当部署的 Metastore 宕机或 HiveServer2 服务宕机时，两个服务可能持续相当长的时间不可用，直到服务被重新拉起。为了避免这种服务中断情况，在真实生产环境中需要部署Hive Metastore 高可用及HiveServer2的高可用。
---
当部署的 Metastore 宕机或 HiveServer2 服务宕机时，两个服务可能持续相当长的时间不可用，直到服务被重新拉起。为了避免这种服务中断情况，在真实生产环境中需要部署Hive Metastore 高可用及HiveServer2的高可用。

那么怎样实现Hive高可用呢，下面分别从实现HiveServer2高可用和实现Metastore高可用两个方面讲解。

## HiveServer2高可用

Hive从0.14开始，使用Zookeeper实现了HiveServer2的HA功能（ZooKeeper Service Discovery），Client端可以通过指定一个nameSpace来连接HiveServer2，而不是指定某一个host和port，本文学习和研究HiveServer2的高可用配置。

![xjxBOb](https://static.studytime.xin//studytime/image/articles/xjxBOb.jpg)


假设现在准备在node1和node3上分别启用两个HiveServer2的实例，并通过zookeeper完成HA的配置。

### node01节点修改hive-site.xml配置
```
<property>
    <name>hive.server2.support.dynamic.service.discovery</name>
    <value>true</value>
</property>
 
<property>
    <name>hive.server2.zookeeper.namespace</name>
    <value>hiveserver2_zk</value>
</property>

<property>
    <name>hive.zookeeper.quorum</name>
    <value> node01:2181,node02:2181,node03:2181</value>
</property>
 
<property>
    <name>hive.zookeeper.client.port</name>
    <value>2181</value>
</property>
```

### node03节点同步配置及信息修改
将安装好的hive文件夹同步到node03节点上，修改node03上的hive-site.xml配置如下：
```
 <property>
 	<name>hive.server2.thrift.bind.host</name>
  	<value>node03</value>
  </property>
```

### 重启服务
分别重启node01,node03节点上的的hiveServer2和metaStore服务
```
nohup hive --service hiveserver2 >> /opt/module/apache-hive-2.1.1-bin/hiveserver.log 2>&1 &
nohup hive --service metastore >> /opt/module/apache-hive-2.1.1-bin/metastore.log 2>&1 &

```

### 在Zookeeper中检查配置
修改完配置后，可通过zookeeper_client命令进行查看，hiveserver2在zookeeper中是否注册成功
```
[zk: localhost:2181(CONNECTED) 1] ls /hiveserver2_zk
[serverUri=0.0.0.0:10001;version=2.1.1-cdh6.3.2;sequence=0000000000]
```

### beeline连接测试
```
beeline> !connect jdbc:hive2://node01:2181,node02:2181,node03:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2_zk
```

JDBC连接的URL格式和参数含义说明：
```
jdbc:hive2://<zookeeper quorum>/<dbName>;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2

// 参数含义
<zookeeper quorum> 为Zookeeper的集群链接串，如node1:2181,node2:2181,node3:2181
<dbName> 为Hive数据库，默认为default
serviceDiscoveryMode=zooKeeper 指定模式为zooKeeper
zooKeeperNamespace=hiveserver2 指定ZK中的nameSpace，即参数hive.server2.zookeeper.namespace所定义
```

## Metastore高可用

### 原理说明

常规连接原理：
![常规连接](https://static.studytime.xin//studytime/image/articles/fHbzph.jpg)

高可用原理：
![高可用原理](https://static.studytime.xin//studytime/image/articles/NIw5fp.jpg)

### 修改节点配置
修改node01、node03节点hive配置文件hive-site.xml
```
 <property>
    <name>hive.metastore.uris</name>
    <value>thrift://node01:9083,thrift://node03:9083</value>
  </property>
```

### 重启服务
分别重启node01,node02节点上的的hiveServer2和metaStore服务
```
nohup hive --service hiveserver2 >> /opt/module/apache-hive-2.1.1-bin/hiveserver.log 2>&1 &

nohup hive --service metastore >> /opt/module/apache-hive-2.1.1-bin/metastore.log 2>&1 &
```

## 测试验证

### 验证HiveServer2是否是高可用
在node03节点上，杀掉占用10000端口的进程，即杀掉node03的hiveServer2进程
```
[root@node03 logs]$ netstat -ntpl |grep 10000

(Not all processes could be identified, non-owned process info

 will not be shown, you would have to be root to see it all.)

tcp6    0   0 :::10000        :::*          LISTEN   87776/java
```

#### 在Zookeeper中检查配置
```
[zk: localhost:2181(CONNECTED) 1] ls /hiveserver2_zk
[serverUri=0.0.0.0:10001;version=2.1.1-cdh6.3.2;sequence=0000000000]
```


#### beeline测试连接
```
[root@node01 ~]# beeline
WARNING: Use "yarn jar" to launch YARN applications.
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/jars/log4j-slf4j-impl-2.8.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/jars/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Beeline version 2.1.1-cdh6.3.2 by Apache Hive
beeline>
beeline> !connect jdbc:hive2://node01:2181,node02:2181,node03:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2_zk
Connecting to jdbc:hive2://node01:2181,node02:2181,node03:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2_zk
Enter username for jdbc:hive2://node01:2181,node02:2181,node03:2181/: hive
Enter password for jdbc:hive2://node01:2181,node02:2181,node03:2181/:
21/07/01 09:31:18 [main]: INFO jdbc.HiveConnection: Connected to 0.0.0.0:10001
Connected to: Apache Hive (version 2.1.1-cdh6.3.2)
Driver: Hive JDBC (version 2.1.1-cdh6.3.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://node01:2181,node02:2181,node0> show tables;
INFO  : Compiling command(queryId=hive_20210701093131_c1958b66-3e2a-443d-8562-22f00f4bb463): show tables
INFO  : Semantic Analysis Completed
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:tab_name, type:string, comment:from deserializer)], properties:null)
INFO  : Completed compiling command(queryId=hive_20210701093131_c1958b66-3e2a-443d-8562-22f00f4bb463); Time taken: 1.164 seconds
INFO  : Executing command(queryId=hive_20210701093131_c1958b66-3e2a-443d-8562-22f00f4bb463): show tables
INFO  : Starting task [Stage-0:DDL] in serial mode
INFO  : Completed executing command(queryId=hive_20210701093131_c1958b66-3e2a-443d-8562-22f00f4bb463); Time taken: 0.046 seconds
INFO  : OK
+-----------+
| tab_name  |
+-----------+
| score4    |
| stu       |
+-----------+
2 rows selected (1.728 seconds)
```


### 验证mestastore是否是高可用

#### node03节点杀死mestastore进程

```
[root@node01 ~]# ps -ef | grep metastore
hive     19802 19786  2 09:23 ?        00:00:33 /usr/java/jdk1.8.0_181-cloudera/bin/java -Dproc_jar -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Stack=true -Xms576716800 -Xmx576716800 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/hive_hive-HIVEMETASTORE-2c60fdac6f3da589eafb946bedf8838a_pid19802.hprof -XX:OnOutOfMemoryError=/opt/cloudera/cm-agent/service/common/killparent.sh -Dlog4j.configurationFile=hive-log4j2.properties -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hive/bin/../conf/parquet-logging.properties -Dyarn.log.dir=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/libexec/../../hadoop-yarn -Dyarn.root.logger=INFO,console -Djava.library.path=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/lib/native -Dhadoop.log.dir=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hadoop -Dhadoop.id.str=hive -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /opt/cloudera/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554/lib/hive/lib/hive-service-2.1.1-cdh6.3.2.jar org.apache.hadoop.hive.metastore.HiveMetaStore -p 9083
root     27765 27722  0 09:49 pts/2    00:00:00 grep --color=auto metastore

kill -9 19802
```

#### 执行查询语句
```
0: jdbc:hive2://node01:2181,node02:2181,node0> select * from stu limit 1;
INFO  : Compiling command(queryId=hive_20210701093806_b5920638-19be-42fb-921f-b81206a1f35f): select * from stu limit 1
INFO  : Semantic Analysis Completed
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:stu.id, type:int, comment:null), FieldSchema(name:stu.name, type:string, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hive_20210701093806_b5920638-19be-42fb-921f-b81206a1f35f); Time taken: 0.366 seconds
INFO  : Executing command(queryId=hive_20210701093806_b5920638-19be-42fb-921f-b81206a1f35f): select * from stu limit 1
INFO  : Completed executing command(queryId=hive_20210701093806_b5920638-19be-42fb-921f-b81206a1f35f); Time taken: 0.001 seconds
INFO  : OK
+---------+-----------+
| stu.id  | stu.name  |
+---------+-----------+
| 1       | zhangsan  |
+---------+-----------+
1 row selected (0.546 seconds)
```

至此， HiveServer2及Metastore的多实例高可用Ha配置完成，能解决生产中的很多问题，比如：并发、负载均衡、单点故障、安全等等，故而强烈建议在生产环境中使用该模式来提供Hive服务。
