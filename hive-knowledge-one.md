---
title: hive 学习之路（一）初识
date: 2020-05-04 17:10:13
updated: 2020-05-04 17:10:43
tags:
    - 大数据
    - hive
categories: hive
toc: true
excerpt: Hive 是基于 Hadoop 的数据仓库解决方案。由于 Hadoop 本身在数据存储和计算方面有很好的可扩展性和高容错性，因此使用 Hive 构建的数据仓库也秉承了这些特性。
---

### Hive 是什么？

> 官方解释: Hive 是基于 Hadoop 的数据仓库解决方案。由于 Hadoop 本身在数据存储和计算方面有很好的可扩展性和高容错性，因此使用 Hive 构建的数据仓库也秉承了这些特性。

简单而言: 
- Hive 最初是由 Facebook 设计的，是基于 Hadoop 的一个数据仓库工具，
- 可以将结构化的数据文件映射为一张数据库表，并提供简单的类 SqL 查询语言(Hive SQL)。
- 底层数据存储在 HDFS 上，Hive 的本质是将 SQL 语句转换为 MapReduce 任务运行，
- 使不熟悉 MapReduce 的用户很方便地利用 HQL 处理和计算 HDFS 上的结构化的数据，适用于离线的批量数据计算。

### HQL 是什么？
Hive 提供的查询语言称为 HQL，为 Hive Query Language 的简写， 该语言标准跟标准 SQL 极为相似，但不等同于为标准 SQL。

###  为什么使用 Hive
- 更友好的接口：操作接口采用类 SQL 的语法，提供快速开发的能力
- 更低的学习成本：避免了写 MapReduce，减少开发人员的学习成本
- 更好的扩展性：可自由扩展集群规模而无需重启服务，还支持用户自定义函数

### Hive 特点

优点：
-  可扩展性，横向扩展，Hive 可以自由的扩展集群的规模，一般情况下不需要重启服务 
    *横向扩展*：通过分担压力的方式扩展集群的规模 
    *纵向扩展*：一台服务器cpu i7-6700k 4 核心 8 线程，8 核心 16 线程，内存64G => 128G
- 延展性，Hive 支持自定义函数，用户可以根据自己的需求来实现自己的函数
- 良好的容错性，可以保障即使有节点出现问题，SQL 语句仍可完成执行

缺点：
- Hive 不支持记录级别的增删改操作，但是用户可以通过查询生成新表或者将查询结果导入到文件中
- Hive 的查询延时很严重，因为 MapReduce Job 的启动过程消耗很长时间，所以不能 用在交互查询系统中。
- Hive 不是⼀个 OLAP(On-Line Analytical Processing) 系统，响应时间慢，无法实时更新数据
- Hive 不是⼀个 OLTP(On-line Transaction Processing) 系统，对事务的支持很弱

总结：
　　Hive 具有 SQL 数据库的外表，但应用场景完全不同，Hive 只适合用来做海量离线数据统计分析，也就是数据仓库。
  
### Hive 架构
![](https://static.studytime.xin/2020-05-04-091422.jpg)

从上图看出hive的架构由四部分组成：

#### 用户接口，包括 shell/CLI（Client Line Interfice）， Thrift协议（JDBC/ODBC），Web UI

- CLI，Shell 终端命令行（Command Line Interface），采用交互形式使用 Hive 命令行与 Hive 进行交互，最常用（学习，调试，生产）
- Thrift 是 Facebook 开发的一个软件框架，可以用来进行可扩展且跨语言的服务的开发， Hive 集成了该服务，能让不同的编程语言调用 Hive 的接口，
  JDBC/ODBC，是 Hive 的基于 JDBC 操作提供的客户端，用户（开发员，运维人员）通过 这连接至 Hive server 服务
- Web UI，通过浏览器访问 Hive

#### 底层的Driver： 驱动器Driver，编译器Compiler，优化器Optimizer，执行器Executor

Driver 组件完成 HQL 查询语句从词法分析，语法分析，编译，优化，以及生成逻辑执行计划的生成。生成的逻辑执行计划存储在 HDFS 中，它的输入是SQL语句，输出为一系列分布式执行程序（可以为Map Reduce、Tez、Spark等）

Hive 的核心是驱动引擎， 驱动引擎由四部分组成：
(1) 解释器：解释器的作用是将 HiveSQL 语句转换为抽象语法树（AST）
(2) 编译器：编译器是将语法树编译为逻辑执行计划
(3) 优化器：优化器是对逻辑执行计划进行优化
(4) 执行器：执行器是调用底层的运行框架执行逻辑执行计划

#### Metastore，元数据存储系统

- 元数据，通俗的讲，就是存储在 Hive 中的数据的描述信息。
- Hive 中的元数据通常包括：表的名字，表的列和分区及其属性，表的属性（内部表和 外部表），表的数据所在目录
- Metastore 默认存在自带的 Derby 数据库中。缺点就是不适合多用户操作，并且数据存 储目录不固定。数据库跟着 Hive 走，极度不方便管理，正式环境一般使用Mysql 代替
- Hive 和 MySQL 之间通过 MetaStore 服务交互

#### Hadoop

Hive 依赖于 Hadoop , 包括分布式文件系统 HDFS、分布式资源管理系统 Yarn 以及分布式计算引擎 MapReduce、Hive 中的数据表对应的数据存放在 HDFS 上，计算资源有 Yarn 分配，而计算任务则来自 MapReduce 引擎。

特殊说明：
根据 Metstore 的运行方式不同，可以将 Hive 分成三种部署模式：
- 嵌入式模式，Metastore 和数据库(Derby) 两个进程嵌入到 Drive 中，当 Drive 启动时会同时运行这两个进行，一般用于测试
- 本地魔术，Driver 和 Metastore 运行在本地，而数据库（比如Mysql）启动在一个共享点上
- 远程模式，Metastore 运行在一个单独节点上，有其他所有服务共享，使用 Beeline,JDBC/ODBC,Cli和Trift等方式访问Hive是，则是采用该模式，这是一种常用生产环境下的部署方式

总结: 
Hive 是Hadoop 生态系统中最早的 SQL 引擎，它的 Metastore 服务已经被越来越多的 SQL 引擎支持，已经成为大数据系统的元信息标准存储仓库。


### 执行流程
HiveQL 通过命令行或者客户端提交，经过 Compiler 编译器，运用 MetaStore 中的元数 据进行类型检测和语法分析，生成一个逻辑方案(Logical Plan)，然后通过的优化处理，产生 一个 MapReduce 任务。



### Hive 查询引擎
Hive 默认是构建在 MapReduce 计算引擎上，伴随新型计算引擎的出现，Hive也逐步支持更高效的 DAG 计算引擎，包括 Tez 和 Spark 等。

### Hive 支持的数据类型
Hive 的内置数据类型可以分为两大类：

- 基础数据类型，包括Boolean，字符串型，浮点型，整型
- 复杂数据类型，包括数组，map，struct。


| 分类 | 类型 | 描述 | 字面量示例 |
| --- | --- | --- | --- |
|BOOLEAN  |true/false  |TRUE  |  |
| TINYINT | 1字节的有符号整数 |-128~127  | 1Y |
| SMALLINT | 2个字节的有符号整数 | -32768~32767 | 1S |
| INT | 4个字节的带符号整数 | 1 |  |
| BIGINT | 8字节带符号整数 | 1L |  |
| FLOAT | 4字节单精度浮点数 | 1.0 |  |
| DOUBLE | 8字节双精度浮点数 | 1.0 |  |
| DEICIMAL | 任意精度的带符号小数 | 1.0 |  |
| STRING | 字符串，变长 | “a”,’b’ |  |
| VARCHAR | 变长字符串 | “a”,’b’ |  |
| CHAR | 固定长度字符串 | “a”,’b’ |  |
| BINARY  | 字节数组 |无法表示|  |
| DATE | 日期 | ‘2016-03-29’ |  |
| TIMESTAMP | 时间戳，纳秒精度 |122327493795  |  |

复杂类型

| 分类 | 类型 | 描述 | 字面量示例 |
| --- | --- | --- | --- |
| ARRAY |  |array(1,2) |  |
| MAP | key-value,key必须为原始类型，value可以任意类型 |map(‘a’,1,’b’,2)  |
| STRUCT | 字段集合,类型可以不同 |struct(‘1’,1,1.0),  |  |


### Hive 数据表多级概念
Hive 数据表是多层级的，Hive 中可以有多个数据库，每个数据库中可以存在多个数据表，每个数据表可以划分出多个分区或者数据同，每个分区内部也可以有多个数据桶。

![](https://static.studytime.xin/2020-05-04-091538.jpg)

- database：在 HDFS 中表现为${hive.metastore.warehouse.dir}目录下一个文件夹
- table：在 HDFS 中表现所属 database 目录下一个文件夹
- external table：与 table 类似，不过其数据存放位置可以指定任意 HDFS 目录路径
- partition：在 HDFS 中表现为 table 目录下的子目录
- bucket：在 HDFS 中表现为同一个表目录或者分区目录下根据某个字段的值进行 hash 散 列之后的多个文件
- view：与传统数据库类似，只读，基于基本表创建

### Hive 中的表分为内部表、外部表、分区表和 Bucket 表
内部表和外部表的区别：
- 删除内部表，删除表元数据和数据
- 删除外部表，删除元数据，不删除数据

内部表和外部表的使用选择：
- 大多数情况，他们的区别不明显，如果数据的所有处理都在 Hive 中进行，那么倾向于 选择内部表，但是如果 Hive 和其他工具要针对相同的数据集进行处理，外部表更合适。
- 使用外部表访问存储在 HDFS 上的初始数据，然后通过 Hive 转换数据并存到内部表中
- 使用外部表的场景是针对一个数据集有多个不同的 Schema
- 通过外部表和内部表的区别和使用选择的对比可以看出来，hive 其实仅仅只是对存储在 HDFS 上的数据提供了一种新的抽象。而不是管理存储在 HDFS 上的数据。所以不管创建内部 表还是外部表，都可以对 hive 表的数据存储目录中的数据进行增删操作。

 
















