---
title: Flume 学习之路（六）Flume 的监控
date: 2020-05-09 22:33:48
updated: 2020-05-09  22:33:49
tags: 
    - 大数据
    - flume
categories: flume
toc: true
excerpt: 使用 Flume 实时收集日志的过程中，尽管有事务机制保证数据不丢失，但仍然需要时刻关注 Source、Channel、Sink 之间的消息传输是否正常。比如，SouceChannel 传输了多少消息，ChannelSink 又传输了多少，两处的消息量是否偏差过大等等。
---


使用 Flume 实时收集日志的过程中，尽管有事务机制保证数据不丢失，但仍然需要时刻关注 Source、Channel、Sink 之间的消息传输是否正常。
比如，SouceChannel 传输了多少消息，ChannelSink 又传输了多少，两处的消息量是否偏差过大等等。

Flume 为我们提供了 Monitor 的机制：[http://flume.apache.org/FlumeUserGuide.html#monitoring](http://flume.apache.org/FlumeUserGuide.html#monitoring) 通过Reporting 的方式，把过程中的Counter都打印出来。

### 类型
- JMX Reporting
- Ganglia Reporting
- JSON Reporting
- Custom Reporting

### Ganglia Reporting
Flume 可以报告它的 metrics 到 ganglia3，只要你在启动 Flume agent 的时候设置一些参数即可，也可以把这些参数设置在 flume-env.sh 配置文件中。需要设置的参数如下，这些参数的前缀如下Flume.monitoring：

-Dflume.monitoring.type：类型必须是ganglia
-Dflume.monitoring.pollFrequency： 默认值是60秒，flume向ganglia报告metrics的时间间隔
-Dflume.monitoring.isGanglia3： 默认是false，ganglia server的版本在3以上，flume 发送的是ganglia3.1的数据格式

启动flume Agent:
```
$ bin/flume-ng agent --conf-file example.conf --name a1 -Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=com.example:1234,com.example2:5455
```

### JSON Reporting

Flume 也可以报告 JSON 格式的 report，为了开启 JSON report，在 Flume 机器上启动了一个 web server。需要在客户端启动时设置以下参数：
type    该组件的名称，这里设置为http
port    该服务监听的端口，默认是41414

启动flume Agent:
`flume-ng agent --conf-file example.conf --name a1 -Dflume.monitoring.type=http -Dflume.monitoring.port=34545`
然后通过http://<hostname>:<port>/metrics来查看值



