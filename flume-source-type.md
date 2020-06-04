---
title: Flume 学习之路（三）Flume的Source类型
date: 2020-05-09 22:29:58
updated: 2020-05-09 22:29:59
tags: 
    - 大数据
    - flume
categories: flume
toc: true
excerpt: Flume的Source类型介绍。
---

[官方文档介绍：http://flume.apache.org/FlumeUserGuide.html#flume-sources](官方文档介绍：http://flume.apache.org/FlumeUserGuide.html#flume-sources)

### Avro Source
内置 Avro Server，可接受 Avro 客户端发送的数据

官方文档描述：
![](https://static.studytime.xin/image/articles/spring-boot20190915125747.png)

![](https://static.studytime.xin/image/articles/spring-boot1228818-20180506141436097-274079212.png)

示例：
```
vim avro_source.properties

a1.sources = s1
a1.sinks = k1
a1.channels = c1

// 配置source
a1.sources.s1.channels = c1
a1.sources.s1.type = avro
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 6666

// 配置channels
a1.channels.c1.type = memory

// 配置sinks
a1.sinks.k1.channel = c1
a1.sinks.k1.type = logger

// 为sources和sinks绑定channels
a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1

```

启动 Flume NG:
```
bin/flume-ng agent -c conf/ -f conf/avro_source.properties -n a1 -Dflume.root.logger=INFO,console
```

开始输入测试数据:
```
vim  666.txt 

123
123
123
```

客户端输入：
```
bin/flume-ng avro-client -c conf/ -H bigdata -p 6666 -F 666.txt
```

### Thrift Source
内置 Thrift Server，可接受 Thrift 客户端发送的数据。ThriftSource 与Avro Source 基本一致。只要把source的类型改成thrift即可

### Exec Source

执行指定的shell，并从该命令标准输出中获取数据，但考虑到该flume agent不运行或者指令出错时，将无法收集到数据，故生产环境很少采用 。

```
vim exec_source.properties

// 配置文件
a1.sources= s1  
a1.sinks= k1  
a1.channels= c1  
   
// 配置sources
a1.sources.s1.type = exec  
a1.sources.s1.command = tail -F /tmp/test.log  
a1.sources.s1.channels = c1  
   
// 配置sinks 
a1.sinks.k1.type= logger  
a1.sinks.k1.channel= c1  
   
// 配置channel
a1.channels.c1.type= memory
```

启动 Flume NG:
```
bin/flume-ng agent --conf conf/ -f conf/exec_source.properties --name a1 -Dflume.root.logger=DEBUG,console 
```

生成测试数据：
```
cd /tmp/test.log

echo "hello world" >>  test.log
```

### Spooling Directory Source

监听一个文件夹下新产生的文件，并读取内容，发至 channel。使用该 Source 需要注意两点：第一个是拷贝到 spool 目录下的文件不可以再打开编辑，第二个是 spool 目录下不可包含相应的子目录。这个主要用途作为对日志的准实时监控。由于该Source可靠性和稳定性较好，被不少公司采用。

示例：

```
vim spooling_dir_source.properties  

a1.sources = s1  
a1.sinks = k1  
a1.channels = c1  
   
// Describe/configure the source  
a1.sources.s1.type =spooldir  
a1.sources.s1.spoolDir =/tmp/logs  
a1.sources.s1.fileHeader= true  
a1.sources.s1.channels =c1  
   
// Describe the sink  
a1.sinks.k1.type = logger  
a1.sinks.k1.channel = c1  
   
// Use a channel which buffers events inmemory  
a1.channels.c1.type = memory

```

启动 Flume NG:
```
bin/flume-ng agent --conf conf/ -f conf/spooling_dir_source.properties --name a1 -Dflume.root.logger=DEBUG,console 
```

生成测试数据
```
mv test.log logs
```

### KafKa Source

内置了Kafka Consumer，可从 KaFka Broker 中读取某个 topic的数据，写入Channel。

### Taildir Source

可以实时监控一个目录下文件的变化，并实时读取新增数据，记录断点，保证重启 Agent 后数据不丢失或被重复传输。

```
vim tail_dir_source.properties

a1.sources = r1
a1.channels = c1
a1.sinks = s1

a1.sources.r1.type = TAILDIR
// positionFile 检查点文件路径
a1.sources.r1.positionFile = /tmp/flume/taildir_position.json
// 指定filegroups，可以有多个，以空格分隔；（TailSource可以同时监控tail多个目录中的文件）
a1.sources.r1.filegroups = f1
// 指定监控的文件目录f1,匹配的文件信息
a1.sources.r1.filegroups.f1 = /tmp/baihe-flume/log_.*.log
a1.sources.r1.fileHeader = true
a1.sources.ri.maxBatchCount = 1000

//  sink1 配置
a1.sinks.s1.type = file_roll
a1.sinks.s1.sink.directory = /tmp/flumefiles
a1.sinks.s1.sink.rollInterval = 0

 
// fileChannel 配置
a1.channels.c1.type = file
// -->检测点文件所存储的目录
a1.channels.c1.checkpointDir = /tmp/flume/checkpoint/baihe/
// -->数据存储所在的目录设置
a1.channels.c1.dataDirs = /tmp/flume/data/baihe/
// -->隧道的最大容量
a1.channels.c1.capacity = 10000
// -->事务容量的最大值设置
a1.channels.c1.transactionCapacity = 200

a1.sources.r1.channels = c1
a1.sinks.s1.channel = c1
```

启动 Flume NG：
```
bin/flume-ng agent --conf conf/ -f conf/tail_dir_source.properties --name a1 -Dflume.root.logger=DEBUG,console 
```

生成测试数据：

采集目录下生成正则对应的文件信息，写入数据
```
cd /tmp/baihe-flume

echo "message1" >> log_20190915.log
echo "message12" >> log_20190915.log

// 查看sink文件
tail  -f /tmp/flumefiles/1568530522032-1
```

TailSource使用了RandomAccessFile来根据positionFile中保存的文件位置来读取文件的，在agent重启之后，亦会先从positionFile中找到上次读取的文件位置，保证内容不会重复发送


### Syslog

Syslog 分为 Tcp Source和 UDP Source两种，分别接受tcp和udp协议发过来的数据，写入Channel
  
### HTTP source

可接受HTTP协议发来的数据，宾写入Channel
 
### 如何选择Flume Source?
 