---
title: Kafka浅谈、kafka集群部署与安装
permalink: kafka-install
date: 2020-11-02 08:10:27
updated: 2020-11-02 08:10:28
tags: 
    - kafka
categories: [大数据,Kafka]
toc: true
excerpt: kafka集群部署与安装.
---

## 下载安装包（http://kafka.apache.org）
```
kafka_2.11-1.1.0.tgz
```
## 规划安装目录
```
/opt/module
```

## 上传安装包到node01服务器，并解压
```
cd /opt/software
tar -zxf kafka_2.11-1.1.0.tgz -C /opt/module/
```

## 修改配置文件
### 在node01上修改

#### node01执行以下命令进行修改配置文件
```
# 进入到kafka安装目录下有一个config目录，进行修改配置文件
cd  /kkb/install/kafka_2.11-1.1.0/config

vim server.properties
  
# 指定kafka对应的broker id ，唯一
broker.id=0
# 指定数据存放的目录
log.dirs=/opt/module/kafka_2.11-1.1.0/logs
# 指定zk地址
zookeeper.connect=node01:2181,node02:2181,node03:2181
# 指定是否可以删除topic ,默认是false 表示不可以删除delete.topic.enable=true
#指定broker主机名
host.name=node01
```
#### node01执行以下命令分发kafka安装目录到其他节点
```shell
cd /opt/module/
scp -r kafka_2.11-1.1.0/ node02:$PWD
scp -r kafka_2.11-1.1.0/ node03:$PWD
```

#### node02执行以下命令进行修改配置
```
cd /opt/module/kafka_2.11-1.1.0/config/
vi server.properties
    
#指定kafka对应的broker id ，唯一
broker.id=1
#指定数据存放的目录
log.dirs=/kkb/install/kafka_2.11-1.1.0/logs
#指定zk地址
zookeeper.connect=node01:2181,node02:2181,node03:2181
#指定是否可以删除topic ,默认是false 表示不可以删除
delete.topic.enable=true
#指定broker主机名
host.name=node02
```
#### node03执行以下命令进行修改配置
```
cd /opt/module/kafka_2.11-1.1.0/config/
vi server.properties
    
#指定kafka对应的broker id ，唯一
broker.id=2
#指定数据存放的目录
log.dirs=/kkb/install/kafka_2.11-1.1.0/logs
#指定zk地址
zookeeper.connect=node01:2181,node02:2181,node03:2181
#指定是否可以删除topic ,默认是false 表示不可以删除
delete.topic.enable=true
#指定broker主机名
host.name=node03
```
## kafka集群启动和停止
### 启动
#### 先启动zk集群
#### 然后在所有节点执行脚本
```shell
  cd /opt/software/kafka_2.11-1.1.0/
  nohup bin/kafka-server-start.sh config/server.properties 2>&1 & 
```
#### 一键启动kafka start_kafka.sh
```shell
#!/bin/sh
for host in node01 node02 node03
do
        ssh $host "source /etc/profile;nohup /opt/module/kafka_2.11-1.1.0/bin/kafka-server-start.sh /opt/module/kafka_2.11-1.1.0/config/server.properties >/dev/null 2>&1 &"
        echo "$host kafka is running"
    
done
```

#### 停止 所有节点执行关闭kafka脚本

```
cd /opt/module/kafka_2.11-1.1.0/
bin/kafka-server-stop.sh 
```
#### 一键停止kafka stop_kafka.sh
```shell
#!/bin/sh
for host in node01 node02 node03
do
  ssh $host "source /etc/profile;nohup /opt/module/kafka_2.11-1.1.0/bin/kafka-server-stop.sh &" 
  echo "$host kafka is stopping"
done
```

#### 一键启动和停止脚本 kafkaCluster.sh 

 ```shell
  #!/bin/sh
  case $1 in 
  "start"){
  for host in node01 node02 node03 
  do
    ssh $host "source /etc/profile; nohup /opt/module/kafka_2.11-1.1.0/bin/kafka-server-start.sh /opt/module/kafka_2.11-1.1.0/config/server.properties > /dev/null 2>&1 &"   
    echo "$host kafka is running..."  
  done  
  };;
  
  "stop"){
  for host in node01 node02 node03 
  do
    ssh $host "source /etc/profile; nohup /opt/module/kafka_2.11-1.1.0/bin/kafka-server-stop.sh >/dev/null  2>&1 &"   
    echo "$host kafka is stopping..."  
  done
  };;
  esac
 ```
  
##### 启动

 ```
  sh kafkaCluster.sh start
```

##### 停止

  ```shell
  sh kafkaCluster.sh stop
```



##  kafka的命令行的管理使用

### 创建topic kafka-topics.sh
node01执行以下命令创建topic
```shell
    cd /opt/module/kafka_2.11-1.1.0/
    bin/kafka-topics.sh --create --partitions 3 --replication-factor 2 --topic test --zookeeper node01:2181,node02:2181,node03:2181
    ```

### 查询所有的topic kafka-topics.sh
```shell
    cd /opt/module/kafka_2.11-1.1.0/
    bin/kafka-topics.sh --list --zookeeper node01:2181,node02:2181,node03:2181 
    ```
### 查看topic的描述信息 kafka-topics.sh
```shell
    cd /kkb/install/kafka_2.11-1.1.0/
    bin/kafka-topics.sh --describe --topic test --zookeeper node01:2181,node02:2181,node03:2181  
 ```

### 删除topic kafka-topics.sh
```shell
    cd /opt/module/kafka_2.11-1.1.0/
    bin/kafka-topics.sh --delete --topic test --zookeeper node01:2181,node02:2181,node03:2181 
```

### node01模拟生产者写入数据到topic中
node01执行以下命令，模拟生产者写入数据到kafka当中去
```shell
  cd /opt/module/kafka_2.11-1.1.0/
  bin/kafka-console-producer.sh --broker-list node01:9092,node02:9092,node03:9092 --topic test 
  ```
### node01模拟消费者拉取topic中的数据
node02执行以下命令，模拟消费者消费kafka当中的数据
```shell
  cd /opt/module/kafka_2.11-1.1.0/
  bin/kafka-console-consumer.sh --zookeeper node01:2181,node02:2181,node03:2181 --topic test --from-beginning
  它会把消息的偏移量保存在zk上
  
  或者
  
  cd /opt/module/kafka_2.11-1.1.0/
  bin/kafka-console-consumer.sh --bootstrap-server node01:9092,node02:9092,node03:9092 --topic test --from-beginning
  它会把消息的偏移量保存在kafka集群内置的topic中
  ```
### 任意kafka服务器执行以下命令可以增加topic分区数

```
cd /opt/module/kafka_2.11-1.1.0

bin/kafka-topics.sh --zookeeper zkhost:port --alter --topic topicName --partitions 8
```