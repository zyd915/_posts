---
title: Kafka浅谈、kafka简介
permalink: kafka-info
date: 2020-11-04 08:10:27
updated: 2020-11-04 08:10:29
tags: 
    - kafka
categories: [大数据, Kafka]
toc: true
excerpt: Kafka起初由Linkedin公司开发的一个多分区、多副本、多订阅者，基于zookeeper协调的分布式消息系统，常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。目前kafk已经定位于一个分布式流失处理平台。它以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。
---

### 什么是kafka?
Kafka起初由Linkedin公司开发的一个多分区、多副本、多订阅者，基于zookeeper协调的分布式消息系统，常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。目前kafk已经定位于一个分布式流失处理平台。它以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。

### kafka 常用业务场景
- 消息中间件
生产者和消费者解耦，避免两者高度依赖
- 消息队列
缓存产生的数据，重放数据，以及流量削峰
- 发布订阅系统
消费者订阅主题，生产者生产的数据，所有订阅者都可以获取到数据
- 消息总线
所有数据流入kafka后，经kafka分流后，流入各个业务子系统

### kafka特点以及优缺点
- 高吞吐、低延迟
  kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒。


- 高伸缩性

   每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。


- 持久性、可靠性

  Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失。


- 容错性

   允许集群中的节点失败，某个节点宕机，Kafka 集群能够正常工作。


- 高并发
  支持数千个客户端同时读写。

### kafka 基本架构
一个典型的kafka体系架构包括若干个Producer、若干个Broker、若干个Consumer,一个一个zookeeper集群。其中Zookeeper用来负责集群元数据管理、控制器的选举等操作。
kafka采用了不同于其他消息队列的push-push架构，而是采用了pull-pull架构，既Producer将数据直接 push 给broker,而consumer从broker pull 拉取数据，此种架构又是主要体现在：
consumer可根据自己实际负载和数据需求情况，获取数据，避免采用push方式给consimer带来的较大压力。
consomer自己未付已读消息的offset而不是有zookeeper管理，大大缓解率broker眼里，更加轻量级。

#### kafka 专用术语

![](https://static.studytime.xin/article/2020/11/16045532304259.jpg)


#### broker
Kafka 集群包含一个或多个服务器，服务器节点称为broker。

broker存储topic的数据。如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的一个partition。

如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的一个partition，剩下的M个broker不存储该topic的partition数据。

如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个partition。在实际生产环境中，尽量避免这种情况的发生，这种情况容易导致Kafka集群数据不均衡。

#### Producer
生产者即数据的发布者，该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息追加到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition中，生产者也可以指定数据存储的partition。

#### Consumer
消费者可以从broker中读取数据。消费者可以消费多个topic中的数据。

### kafka 各组件详情

#### Topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

#### Partition
topic中的数据分割为一个或多个partition。每个topic至少有一个partition。每个partition中的数据使用多个segment文件存储。partition中的数据是有序的，不同partition间的数据丢失了数据的顺序。如果topic有多个partition，消费数据时就不能保证数据的顺序。在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

#### Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。 

#### Leader
每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition。

#### Follower
Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。




