---
title: Spark系列（三）:Spark集群安装部署
permalink: spark-knowledge-three
date: 2020-08-01 17:50:20
updated:2020-08-01 17:50:20
tags:
    - 大数据
    - spark
categories: spark
toc: true
excerpt: Spark集群安装部署
---
## 安装基础
- Java8安装成功
- zookeeper安装成功


### 下载安装包
- [spark官网下载链接](https://archive.apache.org/dist/spark/spark-2.3.3/spark-2.3.3-bin-hadoop2.7.tgz)
- [作者百度网盘链接: https://pan.baidu.com/s/1ytjRn231Gx3RFDSncrj5qQ  密码: 77tm](https://pan.baidu.com/s/1ytjRn231Gx3RFDSncrj5qQ)


### 上传安装包到服务器
```
cd /opt/software/

[hadoop@node01 software]$ ls
3.51.0.tar.gz                                          clickhouse-server-19.16.10.44-1.el7.x86_64.rpm         spark-2.3.3-bin-hadoop2.7.tgz
clickhouse-client-19.16.10.44-1.el7.x86_64.rpm         clickhouse-server-common-19.16.10.44-1.el7.x86_64.rpm  zookeeper-3.4.5-cdh5.14.2.tar.gz
clickhouse-common-static-19.16.10.44-1.el7.x86_64.rpm  hbase-1.2.0-cdh5.14.2.tar.gz
```

### 解压安装包到指定目录
```
tar -zxvf spark-2.3.3-bin-hadoop2.7.tgz -C /opt/module/
```

### 修改配置文件

#### 修改spark-env.sh
```
cd /opt/module/spark-2.3.3-bin-hadoop2.7/conf/

cp spark-env.sh.template spark-env.sh
vim spark-env.sh

#配置java的环境变量
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64
#配置zk相关信息
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER  -Dspark.deploy.zookeeper.url=node01:2181,node02:2181,node03:2181  -Dspark.deploy.zookeeper.dir=/spark"
```


#### 修改slave
```
mv slaves.template slaves

#指定spark集群的worker节点
node02
node03
```

### 分发安装目录到集群其他机器
```
scp -r /opt/module/spark-2.3.3-bin-hadoop2.7 node02:/opt/module
scp -r /opt/module/spark-2.3.3-bin-hadoop2.7 node03:/opt/module
```

### 修改集群spark环境变量，每台机器都执行
```
vim ~/.bash_profile

# spark
export SPARK_HOME=/opt/module/spark-2.3.3-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
source  ~/.bash_profile
```

### 启动集群
#### 启动zookeeper集群，每台机器执行此命令
```
zkServer.sh start
```

#### 启动spark集群
```
$SPARK_HOME/sbin/start-all.sh
```

#### 特殊说明：
#### 在哪里启动这个脚本，就在当前该机器启动一个Master进程，整个集群的worker进程的启动由slaves文件。
#### 后期可以在其他机器单独在启动master
```
$SPARK_HOME/sbin/start-master.sh
```

### 停止集群
#### 在处于active Master主节点执行，可以关闭集群
```
$SPARK_HOME/sbin/stop-all.sh
```

#### 在处于standBy Master主节点执行
```
$SPARK_HOME/sbin/stop-master.sh
```

### 查看进程
```
[hadoop@node01 software]$ xcall jps
--------- node01 ----------
24070 Jps
2601 JobHistoryServer
23530 QuorumPeerMain
23612 Master
--------- node02 ----------
7073 Worker
7011 QuorumPeerMain
7464 Jps
--------- node03 ----------
1394 Jps
1061 Worker
1002 QuorumPeerMain
```

### spark集群的web管理界面
当启动好spark集群之后，可以访问这样一个地址，[http://node01:8080](http://node01:8080)。

![](https://static.studytime.xin/article/20200802191751.png)
可以通过这个web界面观察到很多信息

- 整个spark集群的详细信息
- 整个spark集群总的资源信息
- 整个spark集群已经使用的资源信息
- 整个spark集群还剩的资源信息
- 整个spark集群正在运行的任务信息
- 整个spark集群已经完成的任务信息


### 高可用使用验证

#### 启动master，作为standby
```
ssh node02
$SPARK_HOME/sbin/start-master.sh
```
![](https://static.studytime.xin/article/20200802191855.png)

#### 手动杀死node01上面的Master进程，观察是否会自动进行切换
```
[hadoop@node01 ~]$ jps
24263 Jps
2601 JobHistoryServer
23530 QuorumPeerMain
23612 Master
[hadoop@node01 ~]$ kill -9 23612
```
**node01节点**，由于Master进程被杀死，所以界面无法访问
![](https://static.studytime.xin/article/20200802191936.png)

### **node02节点**，Master被干掉之后，node02节点上的Master成功篡位成功，成为ALIVE状态

![](https://static.studytime.xin/article/20200802192008.png)


## 执行Spark程序on standalone
```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://node01:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
examples/jars/spark-examples_2.11-2.3.3.jar \
10


####参数说明
--class：指定包含main方法的主类
--master：指定spark集群master地址
--executor-memory：指定任务在运行的时候需要的每一个executor内存大小
--total-executor-cores： 指定任务在运行的时候需要总的cpu核数
```

![](https://static.studytime.xin/article/20200714004927.png)
![](https://static.studytime.xin/article/20200714005002.png)


### spark shell启动以及使用
```
bin/spark-shell \
--master spark://node01:7077 \
--executor-memory 1G \
--total-executor-cores 2 \

# 参数说明：
--master spark://node01:7077 指定Master的地址
--executor-memory 500m:指定每个worker可用内存为1G
--total-executor-cores 2: 指定整个集群使用的cup核数为2个
```
特说说明：
如果启动spark shell时没有指定master地址，但是也可以正常启动spark shell和执行spark shell中的程序，其实是启动了spark的local模式，该模式仅在本机启动一个进程，没有与集群建立联系。
Spark Shell中已经默认将SparkContext类初始化为对象sc。用户代码如果需要用到，则直接应用sc即可
Spark Shell中已经默认将SparkSQl类初始化为对象spark。用户代码如果需要用到，则直接应用spark即可

### spark-shell --master local[N] 读取HDFS上文件进行单词统计
```
cd /home/hadoop

vim hello.txt

you,jump
i,jump
you,jump
i,jump
jump

hdfs dfs -mkdir -p /spark
hdfs dfs -put hello.txt /spark

# 进入spark shell

scala> sc.textFile("file:///home/hadoop/hello.txt").flatMap(x=>x.split(" ")).map(x=>(x,1)).reduceByKey((x,y)=>x+y).collect
res7: Array[(String, Int)] = Array((wq,1), (rt,1), (ew,1), ("",2), (ewr,1), (qwr,1), (qw,1), (e,1), (er,1), (qweqw,2))

scala> sc.textFile("hdfs://node01:8020/spark/hello.txt").flatMap(x=>x.split(" ")).map(x=>(x,1)).reduceByKey((x,y)=>x+y).collect
res6: Array[(String, Int)] = Array((jump,1), (you,jump,2), (i,jump,2))
```


### 扩展知识点
### 如何恢复到上一次活着master挂掉之前的状态?
在高可用模式下，整个spark集群就有很多个master，其中只有一个master被zk选举成活着的master，其他的多个master都处于standby，同时把整个spark集群的元数据信息通过zk中节点进行保存。<br />	后期如果活着的master挂掉。首先zk会感知到活着的master挂掉，下面开始在多个处于standby中的master进行选举，再次产生一个活着的master，这个活着的master会读取保存在zk节点中的spark集群元数据信息，恢复到上一次master的状态。整个过程在恢复的时候经历过了很多个不同的阶段，每个阶段都需要一定时间，最终恢复到上个活着的master的转态，整个恢复过程一般需要1-2分钟。


#### 在master的恢复阶段对任务的影响?
对已经运行的任务是没有任何影响，由于该任务正在运行，说明它已经拿到了计算资源，这个时候就不需要master。
对即将要提交的任务是有影响，由于该任务需要有计算资源，这个时候会找活着的master去申请计算资源，由于没有一个活着的master,该任务是获取不到计算资源，也就是任务无法运行。
