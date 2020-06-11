---
title: Hbase学习之路（一）初识与扩展
date: 2020-06-11 19:33:22
updated: 2020-06-11 19:33:23
tags: 
    - bigdata
    - hbase
categories: hbase
toc: true
excerpt: Google 发表的**三驾马车（既谷歌文件系统 GFS、MapReduce 和 BigTable）论文**，被誉为计算机科学进入大数据时代的标志。
---
## 产生背景
Google 发表的**三驾马车（既谷歌文件系统 GFS、MapReduce 和 BigTable）论文**，被誉为计算机科学进入大数据时代的标志。
因早期 Hadoop 开发者，只实现了 Hadoop 文件系统和 Hadoop MapReduce，并未实现 BigTable，故而 BigTable 在Hadoop 大数据生态里面，相当一段时间里，一直是缺席的.
直到 PowerSet 公司推出 Hbase 项目，才算是真正实现了 BigTable 的开源版。PowerSet 早期是一家十分著名的创业公司，创业领域为下一代搜索引擎：自然搜索引擎。虽然在2008年发布了自己的正式产品，但结果不尽人意，后期因微软踏足搜索引擎领域而被收购。但是其在开发语义搜索引擎系统过程中，需要用到类似 Googe BigTable 的系统，而研发出来的 Hbase，确对整个大数据开源社区做出了很大贡献。

## 简介
Hbase 是 Googe BigTable 的开源实现，是 Apace Hadoop 大数据生态系统中的重要成员。是⼀个构建在 HDFS 上的分布式列存储系统。从逻辑上讲，HBase 将数据按照表、⾏和列进⾏存储，它是⼀个**分布式的、稀疏的、持久化存储**的多维度排序表。

## Hbase与BigTable
HBase 依赖于 HDFS 做底层的数据存储，BigTable 依赖 Google GFS 做数据存储
HBase 依赖于 MapReduce 做数据计算，BigTable 依赖 Google MapReduce 做数据计算
HBase 依赖于 ZooKeeper 做服务协调，BigTable 依赖 Google Chubby 做服务协调

## Hbase数据模型 
逻辑数据模型和物理数据存储。

### 逻辑数据模型：
逻辑数据模型是用户从**数据库所看到的模型**，他直接与 Hbase 数据建模相关。

### 物理数据存储：
物理数据模型是面向计算机物理表示的模型，描述了 **Hbase 是数据在存储介质（包括内存和磁盘）上的组织架构**。

## 逻辑数据模型（ HBase 表结构逻辑）
### 基本概述 
类似与数据库中的 database 和 table 逻辑概念，Hbase 分别将之称为 **namespace** 和 **table**,一个 namespace 中包含了一组 table。

Hbase 内置了两个缺省  namespace：
- hbase：系统内建表，包括namespace和meta表
- default：用户建表时未指定namespace的表都创建在此。

Hbase 表由一系列行构成，每行数据有一个 **rowkey**，以及若干 **column famil**y 构成，每个 column family 可包含**无限列**。
    
### 名词概念
#### rowkey
Hbase 表中数据以 rowkey 作为唯一标示的，类似于关系型数据库中的主键，每行数据有一个rowkey，为定位该行数据的索引。同一张表 rowkey 为全局有序的，rowkey 是没有数据类型的，以字节数组（byte[]）形式保存
针对 Hbase 的查询特性，rowkey 对 Hbase 而言查询性能影响很大，故而 rowKey 的设计就尤为重要，设计的时候要兼顾基于 rowkey 的单行查询也要兼容 rowkey 的范围扫描。

#### column family
列簇由多个列共同组成。每行数据都有相同的 column family。column family 属于 schema 的一部分，定义表时必须指定好。每个 column family 包含无数个动态列。为访问控制的基本单元。同一 column family 中的数据在物理上会存储在一个文件中。

#### column qualifier
内部列标示，Hbase 每列数据通过  column family:column qualifier 定位。column qualifier 不属于 schema 的一部分，可以动态指定，且每行数据可以有不同的qualifier。跟 rowkey 类似，column qualifier 也会没有数据类型的，以字节数组（byte[]）形式保存。

#### cell
通过 rowkey、column family、column qualifier 可以唯一定位一个 cell，内部保存了多个版本的数值，默认情况下，每个数据的版本号是写入时间戳。cell内的数据也是没有类型的，以数组形式保存。

#### timestamp
cell 内部数据是多版本的，Hbase 默认是将写入时间戳作为版本号。用户可根据业务需求自行设置版本号，默认为3个版本。读数据若没有指定版本号，则返回最新版本的数据，若存储的版本超过设置的存储最大版本号，则会自动清理。

### 模型特点
- 可扩展性强，支持数十亿行，上百万列，支持数十万个版本
- 可以存储非常稀疏的数据
- 支持点查，根据主键获取一行数据
- 支持扫描，快速获取某些行区间范围的数据，高效获取某几列的数据
- HBase 中支持的数据类型：byte[]（底层所有数据的存储都是字节数组）
- 主要用来存储结构化和半结构化的松散数据

### 物理数据存储
Hbase 是列簇式存储引擎，它以 column family 为单位存储数据，每个 column family 内部数据是以为  key value 形式存储。
在 Hbase 中，同一表中的数据的按照 rowkey 升序排列的，同一行中的不同列是按照 column qualifier 升序排列的，同一个 cell 中的数据是按照版本号降序排列的。

## 应用场景举例
⽹页库（ 360搜索—⽹络爬⾍）
商品库（淘宝搜索--历史账单查询）
交易信息（淘宝数据魔⽅）
云存储服务（⼩⽶）
监控信息（OpenTSDB）
 






