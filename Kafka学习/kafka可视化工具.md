---
title: 超赞的kafka可视化客户端工具，让你嗨皮起来！
permalink: kafka-tool
date: 2021-10-28 00:56:45
updated: 2021-10-28 00:57:45
tags: tool
categories: tool
keywords: kafka可视化工具,kafka连接工具
toc: true
thumbnail: https://static.studytime.xin/article/20211030114718.png
excerpt: Kafka Tool提供了一个较为直观的UI可让用户快速查看Kafka集群中的对象以及存储在topic中的消息，提供了一些专门面向开发人员和管理员的功能
---

## 简介
Kafka Tool是一个用于管理和使用Apache Kafka®集群的GUI应用程序。 Kafka Tool提供了一个较为直观的UI可让用户快速查看Kafka集群中的对象以及存储在topic中的消息，提供了一些专门面向开发人员和管理员的功能，主要特性包括：

- 快速查看所有Kafka集群信息，包括其brokers, topics and consumers
- 查看分区中的消息内容并支持添加新消息
- 查看消费者偏移量，支持查看Apache Storm Kafka Spout消费者偏移量
- 以pretty-printed 格式显示JSON和XML消息
- 添加和删除topic以及其他管理功能
- 将单个消息从指定分区保存到本地硬盘驱动器
- 支持用户编写自己的插件以查看自定义数据格式
- 支持在Windows，Linux和Mac OS上运行

Kafka工具仅供个人使用免费。 未经购买许可证，不得将其用于任何非个人用途，包括商业，教育和非营利性工作。 在下载Kafka Tool后的30天内，出于评估目的非个人是允许使用的，在此之后，用户必须购买有效的许可证或删除该软件。


## 下载安装
[下载地址链接https://www.kafkatool.com/download.html](https://www.kafkatool.com/download.html)

![](https://static.studytime.xin/article/20211028202629.png)


## 配置
第一次打开软件时Kafka Tool会以弹窗的形式提示进行Kafka Cluster配置，需要输入Zookeeper或者Broker地址以找到相应集群。

当然除了基本属性设置外还支持安全性设置、高级设置及JAAS设置等。

![Kafka Tool官网](https://static.studytime.xin/article/20211028202916.png)


将公司Kafka集群的Cluster name, Kafka Cluster Version, Zookeeper Host等配置信息按照要求填写到相关设置项中.

![](https://static.studytime.xin/article/20211028203042.png)

![](https://static.studytime.xin/article/20211028203151.png)

## 使用

### 预览
Kafka Tool提供了集群Brokers，Topics，Consumers信息预览功能，查看指定Topic下消息时支持关键词过滤等。

![Kafka Brokers管理](https://static.studytime.xin/article/20211028204124.png)


![Kafka Topic管理](https://static.studytime.xin/article/20211028204307.png)

![Kafka onsumer管理](https://static.studytime.xin/article/20211028204508.png)


选中Topic，进入topic设置详情页，从页面中可以设置从Oldest还是Newest位置读取消息，同时可以设置读取数据的格式（String或者ByteArray）。由于测试Topic中数据是JSON格式，演示中将key和value都设置为String格式，设置完成后如果update没有变为灰色需要手动点击update更新配置。

![](https://static.studytime.xin/article/20211028204955.png)


可以通过”详情展示“ 按钮来查看详情，也可以选择读取位置。

![查看kafka消息](https://static.studytime.xin/article/20211028205034.png)


在详情展示页面，支持修改查看消息的格式类型，在Text、Hex、JSON与XML之间任意切换

![](https://static.studytime.xin/article/20211028205049.png)

### 分区管理，新增和删除
在 ”topic分区管理“中，点击 ”+“ 按钮，设置 ”topic“ 信息(名字,分区数,副本数)，即可创建topic。

![创建kafka分区](https://static.studytime.xin/article/20211028205855.png)

有创建当然也会有删除啦，在topic分区管理中，选择需要删除的分区，点击 ”×“ 按钮，弹出来的提示框中选择是，即可删除topic。

删除操作一定要慎重!请仔细确认!!!
删除操作一定要慎重!请仔细确认!!!
删除操作一定要慎重!请仔细确认!!!

### 为分区增加消息
在topic分区管理中，选择某个分区，可以为分区增加消息。点击 ”Data“ 下面的 ”+“ 按钮，弹出框选择 ”Add Multiple Message“，设置消息配置选项，在 ”Data“ 文本框按格式输入要添加消息，点击确定即可添加消息。

![kafka分区插入消息](https://static.studytime.xin/article/20211028205601.png)


### 使用Kafka Tool查看和管理Consumer
界面左侧点击Consumers可以看到该集群的所有消费者组，在下面列出来的消费者组中,随便点击一个，右侧会出现包含Properties和Offsets选项的界面。。

Properties包含如下内容:消费者组(组名)Id,消费者类型,偏移量存储位置。

![](https://static.studytime.xin/article/20211028210649.png)


Offsets包含如下信息:刷新、打印、编辑功能，可以获取到消费者组消费的topic信息,分区偏移量信息,获取消费端的偏移量,积压的偏移量,以及偏移量最后提交时间等。

![](https://static.studytime.xin/article/20211028210718.png)

当然这里也可以进行编辑，选择要进行编辑的消费者组，双击编辑按钮， 选择设置偏移量方式(从起始位置消费,从截止位置消费,或者从指定的偏移量开始消费)
，点击update完成设置。

![](https://static.studytime.xin/article/20211028210759.png)

![](https://static.studytime.xin/article/20211028211104.png)

以这样的方式，跳过偏移数据，从而达到消费端的偏移量和topic的截止消费量一致(不等的原因是,topic一直有数据推送)，，以提高消费端性能,减少资源占用。