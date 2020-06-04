---
title: 'Flume 学习之路（二）Flume 高级组件（Interceptor，Channel Selector 和 Sink Processor）'
date: 2020-05-09 22:27:30
updated: 2020-05-09 22:27:31
tags: 
    - 大数据
    - flume
categories: flume
toc: true
excerpt: 除了 Source、channel、Sink外，Flume Agent 还允许用户设置其他组件更灵活地控制数据流，包括 Interceptor，Channel Selector 和 Sink Processor。
---


除了 Source、channel、Sink外，Flume Agent 还允许用户设置其他组件更灵活地控制数据流，包括 Interceptor，Channel Selector 和 Sink Processor。

![](https://static.studytime.xin/image/articles/spring-boottimg.jpeg)


### Interceptor
Flume 中的拦截器（Interceptor），当 Source 读取 Event 发送到 Sink 的 Event 时候，在 Event header 中加入一些有用的信息，或者对 Event 的内容进行过滤，完成初步的数据清洗。

用户可配置多个 Interceptor，形成一个 Interceptor 链。
```
a1.sources.r1.interceptors=i1 i2  
a1.sources.r1.interceptors.i1.type=regex_filter  
a1.sources.r1.interceptors.i1.regex=\\{.*\\}  
a1.sources.r1.interceptors.i2.type=timestamp
```

这在实际业务场景中非常有用，Flume-ng 1.7 中目前提供了以下拦截器：

- Timestamp Interceptor：该 Interceptor 在每个 Event 头部插入时间戳，其中key是timestamp,value为当前时刻。

- Host Interceptor：该 Interceptor 在每个 Event 头部插入当前 Agent 所在机器的host或ip，其中key是host(也可自定义)。

```
vi host_agent.properties

a1.sinks = k1
a1.channels = c1
a1.sources.r1.channels = c1

a1.sources.r1.type = exec
a1.sources.r1.channels = c1
a1.sources.r1.command = tail -F /tmp/baihe/hehe.txt
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = host

参数为true时用IP192.168.8.71，参数为false时用主机名bigdata，默认为true
a1.sources.r1.interceptors.i1.useIP = false
a1.sources.r1.interceptors.i1.hostHeader = agentHost
 
a1.sinks.k1.type=hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = hdfs://bigdata:9000/user/baihe/flume/%y%m%d

a1.sinks.k1.hdfs.filePrefix = baihe_%{agentHost}
往生成的文件加后缀名.log
a1.sinks.k1.hdfs.fileSuffix = .log
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.writeFormat = Text
a1.sinks.k1.hdfs.rollInterval = 10
 
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
```

bin/flume-ng agent -c conf/ -f conf/host_agent.properties -n a1 -Dflume.root.logger=INFO,console

- Static Interceptor：静态拦截器，用于在events header中加入一组静态的key和value。
```
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = static
a1.sources.r1.interceptors.i1.key = static_key
a1.sources.r1.interceptors.i1.value = static_value

```

- UUID Interceptor：该 Interceptor 在每个 Event 头部插入一个128位的全局唯一标示，例如 b5755073-77a9-43c1-8fad-b7a586fc1b97

```
#type的参数不能写成uuid，得写具体，否则找不到类
a1.sources.r1.interceptors.i1.type = org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder
#如果UUID头已经存在,它应该保存
a1.sources.r1.interceptors.i1.preserveExisting = true
a1.sources.r1.interceptors.i1.prefix = UUID_
```

- Regex Filtering Interceptor：该 Interceptor 可根据正则表达式过滤或者保留符合要求的 Event
```

a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = regex_filter
a1.sources.r1.interceptors.i1.regex = ^bai1234.*
#该配置表示过滤掉不是以bai1234开头的events。如果excludeEvents设为true，则表示过滤掉以bai1234开头的events。
a1.sources.r1.interceptors.i1.excludeEvents = false
```

- Regex Extractor Interceptor：该 Interceptor 可根据正则表达式取出对应的值，并插入到头部
```
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = regex_extractor
a1.sources.r1.interceptors.i1.regex = cookieid is (.*?) and ip is (.*)
a1.sources.r1.interceptors.i1.serializers = s1 s2
a1.sources.r1.interceptors.i1.serializers.s1.name = cookieid
a1.sources.r1.interceptors.i1.serializers.s2.name = ip
```

该配置从原始events中抽取出cookieid和ip，加入到events header中。

### Channel Selector

Channel Selector 允许 Flume Source 选择一个或多个目标 Channel，并将当前 Event 写入这些 Channel。

Flume 提供了两种 Channel Selector 实现：
- Replicating Channel Selector：将每个 Event 指定多个 Channel，通过该 Selector，Flume 可将相同数据导入到多套系统中，一遍进行不同地处理。这是Flume 默认采用的 Channel Selector。

demo:
```
a1.sources = r1
a1.channels = c1 c2 c3
a1.sources.r1.selector.type = replicating
a1.sources.r1.channels = c1 c2 c3
# selector.optional是否可选，写入失败，则会被忽略。未设置的失败后，则导致事件失败
a1.sources.r1.selector.optional = c3
```
- Multiplexing Channel Selector：根据 Event 头部的属性值，将 Event写入对应的 Channel

```
a1.sources = r1
a1.channels = c1 c2 c3 c4
a1.sources.r1.selector.type = multiplexing
# 指定匹配的header值
a1.sources.r1.selector.header = state
# state 值为CZ 写入 c1 Channel
a1.sources.r1.selector.mapping.CZ = c1
# state US 写入  c2 c3 Channel
a1.sources.r1.selector.mapping.US = c2 c3
# 默认写入 c4
a1.sources.r1.selector.default = c4
```

### Sink Processor
Flume 允许将多个 Sink 组装在一起形成一个逻辑实体，成为 Sink Group。而 Sink Processor 则在 Sink Group 基础上提供负载均衡以及容错功能。当一个 Sink 挂掉了，可由另一个 Sink 接替。

demo:

```
a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = load_balance
```

Flume 提供了多种 Sink Processor 实现：
- Default Sink Processor：默认的 Sink Processor，仅仅接受一个 Sink，实现了最简单的 source - channel - sink，每个组件只有一个
- Failover Sink Processor：故障转移接收器，Sink Group 中每个 Sink 均被赋予一个优先级，Event 优先由高优先级的 Sink 发送，如果高优先级的 Sink 挂了，则次高优先级的 Sink 接替

```
a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = failover
# 代表优先级，值越大优先级越高
a1.sinkgroups.g1.processor.priority.k1 = 5
a1.sinkgroups.g1.processor.priority.k2 = 10
# 最大等待时长 （以毫秒为单位）
a1.sinkgroups.g1.processor.maxpenalty = 10000
```

- Load balancing Sink Processor：负载均衡接收处理器，Channel 中的 Event 通过某种负载均衡机制，交给 Sink Group 中的所有 Sink 发送，
- 目前 Flume支持两种负载均衡机制，分别是：round_robin（轮训），random（随机）。

demo:
```
a1.sinkgroups  =  g1 
a1.sinkgroups.g1.sinks  =  k1 k2 
# 组件类型名称需要是load_balance
a1.sinkgroups.g1.processor.type  =  load_balance 
# 失败的接收器是否退回
a1.sinkgroups.g1.processor.backoff  =  true
# 选择机制,必须是round_robin，random
a1.sinkgroups.g1.processor.selector  =  random
```



