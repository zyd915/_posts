---
title: 大数据面试题整理、hadoop
permalink: hadoop-question
date: 2020-11-11 13:20:50
updated: 2020-11-11 13:20:53
tags: 
    - bigdata
    - hadoop
categories: hadoop
toc: true
excerpt: 大数据面试习题整理
---

## hadoo存储机制（简述读写流程）
## hadoop简述读流程
## hadoop 简述写流程
## NameNode 的 HA高可用
## HDFS 在上传文件的时候，如果其中一个块突然损坏了怎么办？
DataNode保存数据块时，会产生一个校验码，当存储数据块时，发现校验码不一致，则认为该块已损坏，NameNode会通过其他节点上的正常副本重构受损的数据块。
## NameNode 的作用
HDFS集群管理者，负责管理文件系统元数据和NameNode各节点。存储所有dataNode的元数据信息可以理解为文件、block、datanode的关系。dataNode节点会定时发送心跳包给NameNode，汇报自己的状态，当dataNode出现故障时，nameNode会在其他节点重构该异常nameNode的数据。
## NameNode 在启动的时候会做哪些操作？
NameNode启动时，会加载镜像文件以及edit.log文件重构元数据信息，没有则生成对应的镜像文件和edit.log文件
## 如何实现 hadoop 的安全机制
引入secondName节点，定义与nameNode同步，同步nameNode元数据信息创建检查点。
## 评述 hadoop 运行原理
### 如何杀死hadoop的一个job，上线新节点，下线节点的命令？


