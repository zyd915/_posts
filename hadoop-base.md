---
title: Hadoop 基础
date: 2020-05-27 01:58:08
updated: 2020-05-27 01:58:10
tags: 
    - 大数据
    - hadoop
categories: hadoop
toc: true
excerpt: hdfs是大数据系统的基础，它提供了基本的存储功能，由于底层数据的分布式存储，上层任务也可以利用数据的本地性进行分布式计算。hdfs思想上很简单，就是namenode负责数据存储位置的记录，datanode负责数据的存储。使用者client会先访问namenode询问数据存在哪，然后去datanode存储；写流程也基本类似，会先在namenode上询问写到哪，然后把数据存储到对应的datanode上。所以namenode作为整个系统的灵魂，一旦它挂掉了，整个系统也就无法使用了。在运维中，针对namenode的高可用变得十分关键。
---

### HDFS的概念
Hadoop Distributed File System ，Hadoop分布式文件系统，主要用来解决海量数据的存储问题

### 文件级别的分布式系统

### 块级别的分布式系统


### 设计思想
- 分散均匀存储 dfs.blocksize = 128M
- 备份冗余存储 dfs.replication = 3

### 重要特性

HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M

DFS文件系统会给客户端提供一个统一的抽象目录树，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.dat

目录结构及文件分块信息(元数据)的管理由namenode节点承担


### HDFS 基本架构
![](https://static.studytime.xin/article/20200528010514.png)


### HDFS 的角色

HDFS也是按照Master和Slave的结构。分NameNode、SecondaryNameNode、DataNode这几个角色。

- 主节点 NameNode
HDFS集群管理者，负责管理文件系统原信息和NameNode各节点
管理元信息：维护整个文件系统目录树，各个文件的数据块信息等。
管理NameeNode： DataNode 周期性 NameNode汇报心跳，一旦NameNode发现某个DataNode出现故障，会在其他存货的DataNode上重构丢失的数据块

- 从节点 DataNode
存储实际的数据块，并周期性通过 心跳向NameNode汇报自己的状态信息。

- 客户端 Client
用户通过客户端与NamenOde和DataNode交互，完成HDFS关联，数据读写等操作。
文件的分块操作也是客户端完成，当HDFS写入文件时没客户端首先将文件切分成瞪大的数据块，之后从NameNode领取三个DataNode地址，并在他们之间建立数据流水线，将数据块流失写入这些节点中。

- secondaryNameNode
辅助namenode管理元数据信息，以及元数据信息的冷备份

- fsimage
元数据镜像文件（文件系统的目录树。）

- edits
元数据的操作日志（针对文件系统做的修改操作记录）

- 其他
namenode内存中存储的是=fsimage+edits。
SecondaryNameNode负责定时默认1小时，从namenode上，获取fsimage和edits来进行合并，然后再发送给namenode。减少namenode的工作量。
