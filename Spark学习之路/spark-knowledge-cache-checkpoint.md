---
title: Spark RDD持久化缓存机制
permalink: spark-knowledge-cache-checkpoint
date: 2020-08-02 17:50:20
updated: 2020-08-02 17:50:20
tags:
    - 大数据
    - spark
categories: spark
toc: true
excerpt: RDD 持久化是 Spark 非常重要的特性之一。用户可显式将一个 RDD 持久化到内存或磁盘中，以便重用该RDD。RDD 持久化是一个分布式的过程，其内部的每个 Partition 各自缓存到所在的计算节点上。RDD 持久化存储能大大加快数据计算效率，尤其适合迭代式计算和交互式计算。
---

## RDD 持久化

### 什么事RDD持久化缓存机制
RDD 持久化是 Spark 非常重要的特性之一。用户可显式将一个 RDD 持久化到内存或磁盘中，以便重用该RDD。RDD 持久化是一个分布式的过程，其内部的每个 Partition 各自缓存到所在的计算节点上。RDD 持久化存储能大大加快数据计算效率，尤其适合迭代式计算和交互式计算。

### 如何对rdd设置缓存
Spark 提供了 persist 和 cache 两个持久化函数，其中 cache 将 RDD 持久化到内存中，而 persist 则支持多种存储级别。

persist RDD 存储级别：

| 持久化级别	 | 含义 |
| --- | --- |
| MEMORY_ONLY | 以非序列化的Java对象的方式持久化在JVM内存中。如果内存无法完全存储RDD所有的partition，那么那些没有持久化的partition就会在下一次需要使用它的时候，重新被计算。 |
| MEMORY_AND_DISK | 同上，但是当某些partition无法存储在内存中时，会持久化到磁盘中。下次需要使用这些partition时，需要从磁盘上读取。
 |
| MEMORY_ONLY_SER | 同MEMORY_ONLY，但是会使用Java序列化方式，将Java对象序列化后进行持久化。可以减少内存开销，但是需要进行反序列化，因此会加大CPU开销 |
| MEMORY_AND_DSK_SER | 同MEMORY_AND_DSK。但是使用序列化方式持久化Java对象。 |
| DISK_ONLY | 使用非序列化Java对象的方式持久化，完全存储到磁盘上。 |
| MEMORY_ONLY_2
MEMORY_AND_DISK_2 | 如果是尾部加了2的持久化级别，表示会将持久化数据复用一份，保存到其他节点，从而在数据丢失时，不需要再次计算，只需要使用备份数据即可。 |

是并不是这两个方法被调用时立即缓存，而是触发后面的action时，该RDD将会被缓存在计算节点的内存中，并供后面重用。
通过过查看源码发现cache最终也是调用了persist方法，默认的存储级别都是仅在内存存储一份，Spark的存储级别还有好多种，存储级别在object StorageLevel中定义的。


### 代码演示
```scala
val rdd1=sc.textFile("/words.txt")
val rdd2=rdd1.flatMap(_.split(" "))
val rdd3=rdd2.cache
rdd3.collect

val rdd4=rdd3.map((_,1))
val rdd5=rdd4.persist(缓存级别)
rdd5.collect
```

### 如何选择RDD持久化策略：
Spark 提供的多种持久化级别，主要是为了在 CPU 和内存消耗之间进行取舍。下面是一些通用的持久化级别的选择建议：

- 优先使用 MEMORY_ONLY，如果可以缓存所有数据的话，那么就使用这种策略。因为纯内存速度最快，而且没有序列化，不需要消耗CPU进行反序列化操作。
- 如果MEMORY_ONLY策略，无法存储的下所有数据的话，那么使用MEMORY_ONLY_SER，将数据进行序列化进行存储，纯内存操作还是非常快，只是要消耗CPU进行反序列化。
- 如果需要进行快速的失败恢复，那么就选择带后缀为_2的策略，进行数据的备份，这样在失败时，就不需要重新计算了。
- 能不使用DISK相关的策略，就不用使用，有的时候，从磁盘读取数据，还不如重新计算一次。


### 什么时候设置缓存
#### 某个rdd的数据后期被使用了多次

![](https://static.studytime.xin/article/20200803004106.png)

如上图所示的计算逻辑： 
- 当第一次使用rdd2做相应的算子操作得到rdd3的时候，就会从rdd1开始计算，先读取HDFS上的文件，然后对rdd1 做对应的算子操作得到rdd2,再由rdd2计算之后得到rdd3。同样为了计算得到rdd4，前面的逻辑会被重新计算。
- 默认情况下多次对一个rdd执行算子操作， rdd都会对这个rdd及之前的父rdd全部重新计算一次。 这种情况在实际开发代码的时候会经常遇到，但是我们一定要避免一个rdd重复计算多次，否则会导致性能急剧降低。   

总结：
可以把多次使用到的rdd，也就是公共rdd进行持久化，避免后续需要，再次重新计算，提升效率。



#### 为了获取得到一个rdd的结果数据，经过了大量的算子操作或者是计算逻辑比较复杂

![](https://static.studytime.xin/article/20200803004124.png)

```
val rdd2=rdd1.flatMap(函数).map(函数).reduceByKey(函数).xxx.xxx.xxx.xxx.xxx
```

### 清除缓存数据
#### 自动清除
一个application应用程序结束之后，对应的缓存数据也就自动清除
#### 手动清除
```
调用rdd的unpersist方法
```

## RDD的checkpoint机制
除了 cache 和 persist 之外，Spark 还提供了另外一种持久化：checkpoint, 它能将 RDD 写入分布式文件系统，提供类似于数据库快照的功能。

它是提供了一种相对而言更加可靠的数据持久化方式。它是把数据保存在分布式文件系统，
比如HDFS上。这里就是利用了HDFS高可用性，高容错性（多副本）来最大程度保证数据的安全性。

### 如何设置checkpoint
#### 在hdfs上设置一个checkpoint目录
```scala
sc.setCheckpointDir("hdfs://node01:8020/checkpoint") 
```

#### 对需要做checkpoint操作的rdd调用checkpoint方法
```scala
val rdd1=sc.textFile("/words.txt")
rdd1.checkpoint
val rdd2=rdd1.flatMap(_.split(" ")) 
```

#### 最后需要有一个action操作去触发任务的运行

```scala
rdd2.collect
```

## cache、persist、checkpoint三者区别
### cache和persis
* cache默认数据缓存在内存中
* persist可以把数据保存在内存或者磁盘中
* 后续要触发 cache 和 persist 持久化操作，需要有一个action操作
* 它不会开启其他新的任务，一个action操作就对应一个job 
* 它不会改变rdd的依赖关系，程序运行完成后对应的缓存数据就自动消失

### checkpoint
* 可以把数据持久化写入到hdfs上
* 后续要触发checkpoint持久化操作，需要有一个action操作，后续会开启新的job执行checkpoint操作
* 它会改变rdd的依赖关系，后续数据丢失了不能够在通过血统进行数据的恢复。
* 程序运行完成后对应的checkpoint数据就不会消失

```scala
sc.setCheckpointDir("/checkpoint")
val rdd1=sc.textFile("/words.txt")
val rdd2=rdd1.cache
rdd2.checkpoint
val rdd3=rdd2.flatMap(_.split(" "))
rdd3.collect
```
checkpoint操作要执行需要有一个action操作，一个action操作对应后续的一个job。该job执行完成之后，它会再次单独开启另外一个job来执行 rdd1.checkpoint操作。

对checkpoint在使用的时候进行优化，在调用checkpoint操作之前，可以先来做一个cache操作，缓存对应rdd的结果数据，后续就可以直接从cache中获取到rdd的数据写入到指定checkpoint目录中
   
   
### checkpoint 于 cache 和 persist去区别
- Spark 自动管理（创建和回收）cache 和 persist 持久化的数据，而checkpoint持久化的数据需要有由户自己管理
- checkpoint 会清除 RDD 的血统，避免血统过长导致序列化开销增大，而 cache 和 persist 不会清楚 RDD 的血统

代码实例：
```scala
sc.checkpoint("hdfs://spark/rdd");
val data = sc.testFile("hdfs://node01:8020/input");
val rdd = data.map(..).reduceByKey(...)

rdd.checkpoint
rdd.count()
```

