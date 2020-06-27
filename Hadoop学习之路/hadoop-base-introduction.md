---
title: Hadoop 发展背景和简介
permalink: hadoop-base-introduction
date: 2020-05-27 00:58:16
updated: 2020-05-27 00:58:17
tags: 
    - 大数据
    - hadoop
categories: hadoop
toc: true
excerpt: HADOOP最早起源于Nutch。Nutch的设计目标是构建一个大型的全网搜索引擎，包括网页抓取、索引、查询等功能，但随着抓取网页数量的增加，遇到了严重的可扩展性问题——如何解决数十亿网页的存储和索引问题。

---


### Hadoop产生的背景

HADOOP最早起源于Nutch。Nutch的设计目标是构建一个大型的全网搜索引擎，包括网页抓取、索引、查询等功能，但随着抓取网页数量的增加，遇到了严重的可扩展性问题——如何解决数十亿网页的存储和索引问题。

- 基于google开源三驾马车中的GFS的开源实现
- 2003年google发表GFS论文
- 2004年hdfs开源
- 2004年“谷歌MapReduce”论文
- 2005年Nutch开源版MapReduce
- 2008年 Hadoop项目成为apache开源顶级项目

### Hadoop是什么
Hadoop 是 apach 下的顶级开源项目，[官网地址http://hadoop.apache.org/](http://hadoop.apache.org/)

### Hadoop 的定义

狭义上指hadoop 的组件，hdfs、yarn、mapReduce等
广义上指以hadoop为核心的整个大数据处理体系
- hadoop
- hive
- kafka
- hbase
- zookeper
- ...

![](https://static.studytime.xin/article/20200528005708.png)


### Hadoop 版本发展
- 0.x系列版本
hadoop当中最早的一个开源版本，在此基础上演变而来的1.x以及2.x的版本

- 1.x版本系列
hadoop版本当中的第二代开源版本，主要修复0.x版本的一些bug等

- 2.x版本系列
架构产生重大变化，引入了yarn平台等许多新特性，也是现在生产环境当中使用最多的版本

- 3.x版本系列
在2.x版本的基础上，引入了一些hdfs的新特性等，且已经发型了稳定版本，未来公司的使用趋势

### Hadoop 生产环境版本选择
- 免费开源版本apache hadoop
- 免费开源版本hortonWorks hadoop 
- ClouderaManage hadoop  

针对版本的选择上，apache hadoop 开源，适合具有较强研发实力公司使用，在此基础上定制型开发同时，但存在版本兼容问题等，所以中小型公司不推荐使用。
hortonWorks hadoop  已经合并如 ClouderaManage hadoop，中小型公司推荐使用，做了版本兼容，以及一些其他优化，但不开源。


### Hadoop 模块组成
- HDFS：一个高可靠、高吞吐量的分布式文件系统
- MapReduce：一个分布式的离线并行计算框
- YARN：作业调度与集群资源管理的框架
- Common：支持其他模块的工具模块。

针对现有技术发展以及趋势，安装hadoop主要就是使用yarn进行资源调度，hdfs作为文件存储系统，MapReduce分布式计算已经被spark，flink等替代。

### Hadoop 的优缺点

#### 优点
- 可构建在廉价机器上
- 高容错性
- 适合批处理
- 适合大数据处理
- 流式文件访问，一次性写入，多次读取，保证数据一致性

#### 缺点
- 不适合低延迟数据访问
- 不适合小文件存取
- 不适合并发写入、文件随机修改，一个文件只能有一个写者 仅支持 append