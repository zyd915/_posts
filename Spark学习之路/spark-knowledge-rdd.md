---
title: Spark之RDD
permalink: spark-knowledge-rdd
date: 2020-08-02 10:50:20
updated: 2020-08-02 10:50:23
tags:
    - 大数据
    - spark
categories: spark
toc: true
excerpt: RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象，代表一个不可变、可分区、里面的元素可并行计算的集合。
### RDD的概述
#### RDD是什么？

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象，代表一个不可变、可分区、里面的元素可并行计算的集合。

#### RDD的主要属性？

RDD 是Spark 中最基本的数据抽象，是一个逻辑概念，它可能并不对应次磁盘或内存中的物理数据，而仅仅是记录了RDD的由来, 父RDD是谁，以及怎样从父RDD计算而来。

spark 源码里面对 RDD 的描述：
```
Internally, each RDD is characterized by five main properties:
A list of partitions
A function for computing each split
A list of dependencies on other RDDs
Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
```

可以知道，每个 RDD 有以下五部分构成：

#### 一组分片（Partition）

数据分区列表，即数据集的基本组成单位。这里表示一个rdd有很多分区，每个分区内部包含了该rdd的部分数据，spark中任务是以task线程方式存在。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Core的数目。

#### 每个Partition的计算函数
Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。compute函数会对迭代器进行复合，不需要保存每次计算的结果。

#### RDD之间的依赖关系
RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算。

#### （可选的）对于key-value类型的RDD,则包含一个Partitioner，即RDD的分片函数。
当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。只有对于于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。

#### （可选的）计算每个Parition所倾向的节点位置。存储存取每个Partition的优先位置（preferred location）。
对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块的存储位置。

RDD是一个应用层面的逻辑概念。一个RDD多个分片。RDD就是一个元数据记录集，记录了RDD内存所有的关系数据。

### 基于spark的单词统计程序剖析rdd的五大属性

#### 需求
HDFS上有一个大小为300M的文件，通过spark实现文件单词统计，最后把结果数据保存到HDFS上

#### 代码
```scala
sc.textFile("/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).saveAsTextFile("/out")
```

#### RDD五大属性剖析图
![RDD五大属性剖析](https://static.studytime.xin/article/20200730234543.png)

### Spark编程接口
Spark 程序设计流程一般如下：
步骤一：实例化 sparkContent 对象。sparkContent 封装了程序运行的上下文环境，包括配置信息、数据库管理器、任务调度等。
步骤二：构造 RDD。可通过 sparkContent 提供的函数构造 RDD，常见的 RDD 构造方式分为：将 Scala集合转换为 RDD 和将 Hadoop 文件转换为 RDD。
步骤三：在 RDD 基础上，通过 Spark 提供的 transformation 算子完成数据处理步骤。
步骤四：通过 action 算子将最终 RDD 作为结果直接返回或者保存到文件中。

Spark 提供了两大类编程接口，分别为 RDD 操作符以及共享变量。
其中 RDD 操作符包括 transformation 和 action 以及 control API 三类；共享变量包括广播变量和累加器两种。


### 创建 sparkContent 对象，封装了 Spark 执行环境信息

#### 创建conf ,封装了spark配置信息

```scala
val sparkConf: SparkConf = new SparkConf().setAppName("WordCountOnSpark")
```

#### 创建 SparkContext，封装了调度器等信息
```scala
val sc = new SparkContext(sparkConf)
```

### 构建RDD
#### 通过已经存在的scala集合去构建
```
val rdd1=sc.parallelize(List(1,2,3,4,5))
val rdd2=sc.parallelize(Array("hadoop","hive","spark"))
val rdd3=sc.makeRDD(List(1,2,3,4))
```

#### 将文本文件转换为 RDD
```
sc.textFile(“/data”, 1) 
sc.textFile(“/data/file.txt”, 1) 
sc.textFile(“/data/*.txt”, 1) 
sc.textFile(“hdfs://bigdata:9000/data/”, 1) 
sc.sequenceFile(“/data”, 1) 
sc.wholeTextFiles(“/data”, 1)
```
