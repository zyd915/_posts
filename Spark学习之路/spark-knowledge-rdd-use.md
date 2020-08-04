---
title: Spark的算子Transformation和Action
permalink: spark-knowledge-rdd-use
date: 2020-08-02 12:50:20
updated: 2020-08-02 12:50:23
tags:
    - 大数据
    - spark
categories: spark
toc: true
excerpt: Spark的算子Transformation和Action
---

### RDD的算子分类
####  transformation（转换）
根据已经存在的rdd转换生成一个新的rdd, 它是延迟加载，它不会立即执行

#### action (动作)
它会真正触发任务的运行，将rdd的计算的结果数据返回给Driver端，或者是保存结果数据到外部存储介质中

### RDD transformation
transformation API 是惰性的，调用这些API比不会触发实际的分布式数据计算，而仅仅是将相关信息记录下来，直到action API才会开始数据计算。

Spark 提供了大量的 transformation API，下面列举了一些常用的API:

| API | 功能 |
| --- | --- |
| map(func) | 将 RDD 中的元素，通过 func 函数逐一映射成另外一个值，形成一个新的 RDD |
| filter(func) | 将 RDD 中使用 func 函数返回 true 的元素过滤出来，形成一个新的 RDD |
| flatMap(func | 类似于map，但每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素） |
| mapPartitions(func) | 类似于 map，但独立地在 RDD 的每一个分片上运行，因此在类型为T的 RDD 上运行时，func的函数类型必须是Iterator[T] => Iterator[U]|
| sample(withReplacement, fraction, seed) | 数据采样函数。根据fraction指定的比例对数据进行采样，可以选择是否使用随机数进行替换，seed 用于指定随机数生成器种子 |
| union(otherDataset) | 求两个 RDD (目标 RDD 与指定 RDD)的并集，并以 RDD 形式返回 |
| intersection(otherDataset) | 求两个 RDD (目标 RDD 与指定 RDD )的交集，并以 RDD 形式返回 |
| distinct([numTasks])) | 对目标 RDD 进行去重后返回一个新的 RDD |
| groupByKey([numTasks]) | 针对 key/value 类型的 RDD，将 key 相同的 value 聚集在一起。默认任务并发度与父 RDD 相同，可显示设置 [numTasks]大小 |
| reduceByKey(func, [numTasks]) | 针对 key/value 类型的 RDD，将 key 相同的 value 聚集在一起，将对每组value，按照函数 func 规约，产生新的 RDD  |
| aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | 与 reduceByKey 类似，但目标 key/value 的类型与最终产生的 RDD 可能不同 |
| sortByKey([ascending], [numTasks]) | 针对 key/value 类型的 RDD，按照 key 进行排序，若 ascending 为 true，则为升序，反之为降序|
| join(otherDataset, [numTasks]) |  针对 key/value 类型的 RDD，对 (K,V) 类型的 RDD 和(K,W)类型的RDD上调用，按照 key 进行等值连接，返回一个相同key对应的所有元素在一起的(K,(V,W))的RDD  相当于内连接（求交集） |
| cogroup(otherDataset, [numTasks]) | 分组函数，对(K,V)类型的RDD和(K,W)类型的RD按照key进行分组，产生新的 (K,(Iterable<V>,Iterable<W>)) 类型的RDD|
| cartesian(otherDataset) | 求两个 RDD 的笛卡尔积 |
| coalesce(numPartitions)    | 重新分区, 缩减分区数，用于大数据集过滤后，提高小数据集的执行效率 |
| repartition(numPartitions) | 重新分区，将目标 RDD 的 partition 数量重新调整为 numPartitions， 少变多 |
| glom() | 将RDD中每个partition中元素转换为数组,并生 成新的rdd2 |
| mapValues() | 针对于(K,V)形式的类型只对V进行操作 |
| cache | RDD缓存，可以避免重复计算从而减少时间，cache 内部调用了 persist 算子，cache 默认就一个缓存级别 MEMORY-ONLY |
| persist | persist 可以选择缓存级别 |


### RDD action
transformation 算子具有惰性执行的特性，他仅仅是记录一些原信息，知道遇到action算子才会触发相关transformation 算子的执行，

Spark 提供了大量的 action API，下面列举了一些常用的API:

| API | 功能 |
| --- | --- |
| reduce(func) | 将RDD中元素前两个传给输入函数，产生一个新的return值，新产生的return值与RDD中下一个元素（第三个元素）组成两个元素，再被传给输入函数，直到最后只有一个值为止 |
| collect() | 将 RDD 以数组的形式返回给 Driver,通过将计算后的较小结果集返回 |
| count() | 计算 RDD 中的元素个数 |
| first() | 返回 RDD 中第一个元素 |
| take(n) | 以数组的形式返回 RDD 前 n 个元素 |
| takeSample(withReplacement,num, [seed]) | 返回一个数组，该数组由从数据集中随机采样的num个元素组成，可以选择是否用随机数替换不足的部分，seed用于指定随机数生成器种子 |
| saveAsTextFile(path) | 将 RDD 存储到文本文件中，并一次调用每个元素的toString方法将之转换成字符串保存成一行 |
| saveAsSequenceFile(path)  | 针对 key/value 类型的 RDD,保存成 SequenceFile 格式文件|
| countByKey() | 针对 key/value 类型的 RDD,统计每个 key出现的次数，并以 hashmap 形式返回 |
| foreach(func) |  将 RDD 中的元素一次交给 func 处理 |
| aggregate | 先对分区进行操作，再总体操作  |
|aggregateByKey||
| lookup(key: K) | 针对 key/value 类型的 RDD, 指定key值，返回RDD中该K对应的所有V值。 |
| foreachPartition |  类似于 foreach，但独立地在 RDD 的每一个分片上运行，其中可嵌入foreach算子 |


### RDD常用的算子操作演示
为了方便前期的测试和学习，可以使用spark-shell进行演示

```shell
spark-shell --master local[2]
```


#### map

```scala
val rdd1 = sc.parallelize(List(5, 6, 4, 7, 3, 8, 2, 9, 1, 10))

//把rdd1中每一个元素乘以10
rdd1.map(_*10).collect
```

#### filter

```scala
val rdd1 = sc.parallelize(List(5, 6, 4, 7, 3, 8, 2, 9, 1, 10))

//把rdd1中大于5的元素进行过滤
rdd1.filter(x => x >5).collect
```

#### flatMap

```scala
val rdd1 = sc.parallelize(Array("a b c", "d e f", "h i j"))
//获取rdd1中元素的每一个字母
rdd1.flatMap(_.split(" ")).collect
```

#### intersection、union

```scala
val rdd1 = sc.parallelize(List(5, 6, 4, 3))
val rdd2 = sc.parallelize(List(1, 2, 3, 4))
//求交集
rdd1.intersection(rdd2).collect

//求并集
rdd1.union(rdd2).collect
```

#### distinct

```scala
val rdd1 = sc.parallelize(List(1,1,2,3,3,4,5,6,7))
//去重
rdd1.distinct
```

#### join、groupByKey

```scala
val rdd1 = sc.parallelize(List(("tom", 1), ("jerry", 3), ("kitty", 2)))
val rdd2 = sc.parallelize(List(("jerry", 2), ("tom", 1), ("shuke", 2)))
//求join
val rdd3 = rdd1.join(rdd2)
rdd3.collect
//求并集
val rdd4 = rdd1 union rdd2
rdd4.groupByKey.collect
```

#### cogroup

```scala
val rdd1 = sc.parallelize(List(("tom", 1), ("tom", 2), ("jerry", 3), ("kitty", 2)))
val rdd2 = sc.parallelize(List(("jerry", 2), ("tom", 1), ("jim", 2)))
//分组
val rdd3 = rdd1.cogroup(rdd2)
rdd3.collect
```

#### reduce

```scala
val rdd1 = sc.parallelize(List(1, 2, 3, 4, 5))

//reduce聚合
val rdd2 = rdd1.reduce(_ + _)
rdd2.collect

val rdd3 = sc.parallelize(List("1","2","3","4","5"))
rdd3.reduce(_+_)

这里可能会出现多个不同的结果，由于元素在不同的分区中，每一个分区都是一个独立的task线程去运行。这些task运行有先后关系
```

#### reduceByKey、sortByKey

```scala
val rdd1 = sc.parallelize(List(("tom", 1), ("jerry", 3), ("kitty", 2),  ("shuke", 1)))
val rdd2 = sc.parallelize(List(("jerry", 2), ("tom", 3), ("shuke", 2), ("kitty", 5)))
val rdd3 = rdd1.union(rdd2)

//按key进行聚合
val rdd4 = rdd3.reduceByKey(_ + _)
rdd4.collect

//按value的降序排序
val rdd5 = rdd4.map(t => (t._2, t._1)).sortByKey(false).map(t => (t._2, t._1))
rdd5.collect
```



#### repartition、coalesce

```scala
val rdd1 = sc.parallelize(1 to 10,3)
//打印rdd1的分区数
rdd1.partitions.size

//利用repartition改变rdd1分区数
//减少分区
rdd1.repartition(2).partitions.size

//增加分区
rdd1.repartition(4).partitions.size

//利用coalesce改变rdd1分区数
//减少分区
rdd1.coalesce(2).partitions.size


//repartition:  重新分区， 有shuffle
//coalesce:     合并分区 / 减少分区 	默认不shuffle   
//默认 coalesce 不能扩大分区数量。除非添加true的参数，或者使用repartition。

//适用场景：
//1、如果要shuffle，都用 repartition
//2、不需要shuffle，仅仅是做分区的合并，coalesce
//3、repartition常用于扩大分区。

```

#### map、mapPartitions、mapPartitionsWithIndex

```scala
val rdd1=sc.parallelize(1 to 10,5)
rdd1.map(x => x*10)).collect
rdd1.mapPartitions(iter => iter.map(x=>x*10)).collect

//index表示分区号  可以获取得到每一个元素属于哪一个分区
rdd1.mapPartitionsWithIndex((index,iter)=>iter.map(x=>(index,x)))

map：用于遍历RDD,将函数f应用于每一个元素，返回新的RDD(transformation算子)。
mapPartitions:用于遍历操作RDD中的每一个分区，返回生成一个新的RDD（transformation算子）。

总结：
如果在映射的过程中需要频繁创建额外的对象，使用mapPartitions要比map高效
比如，将RDD中的所有数据通过JDBC连接写入数据库，如果使用map函数，可能要为每一个元素都创建一个connection，这样开销很大，如果使用mapPartitions，那么只需要针对每一个分区建立一个connection。
```

#### foreach、foreachPartition

```scala
val rdd1 = sc.parallelize(List(5, 6, 4, 7, 3, 8, 2, 9, 1, 10))

//foreach实现对rdd1里的每一个元素乘10然后打印输出
rdd1.foreach(x=>println(x * 10))

//foreachPartition实现对rdd1里的每一个元素乘10然后打印输出
rdd1.foreachPartition(iter => iter.foreach(x=>println(x * 10)))

foreach:用于遍历RDD,将函数f应用于每一个元素，无返回值(action算子)。
foreachPartition: 用于遍历操作RDD中的每一个分区。无返回值(action算子)。


总结：
一般使用mapPartitions或者foreachPartition算子比map和foreach更加高效，推荐使用。
```
