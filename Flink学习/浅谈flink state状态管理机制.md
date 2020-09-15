---
title:  浅谈flink state状态管理机制
permalink: flink-state
date: 2020-09-09 22:41:41
updated: 2020-09-09 22:41:42
tags:
    - 大数据
    - flink
categories: flink
toc: true
excerpt: Apache Flink®—Stateful Computations over Data Streams,flink是一个默认就有状态的分分析引擎，针对流失计算引擎中的数据往往是转瞬即逝，但在flink真实业务场景确不能这样，什么都不能留下，肯定是需要有数据留下的，针对这些数据留下来存储下来，在flink中叫做state，中文可以翻译成状态。
---

## state 简述
**Apache Flink® — Stateful Computations over Data Streams**,flink是一个默认就有状态的分分析引擎，针对流失计算引擎中的数据往往是转瞬即逝，但在flink真实业务场景确不能这样，什么都不能留下，肯定是需要有数据留下的，针对这些数据留下来存储下来，在flink中叫做state，中文可以翻译成状态。

## state 类型
Flink中有两种基本类型的State, 分别为 Keyed State(键控状态) 和 Operator State(算子状态)。

## keyed State(键控状态)
Keyed State：顾名思义就是基于 KeyedStream 上的状态，这个状态是跟特定的Key绑定的。KeyedStrean 流上的每一个Key，都对应一个 State。Flink针对Keyed State提供了以下六种类可以保存 State 的数据结构类型。

### keyed state托管状态有六种类型
#### ValueState<T>
保存一个可以更新和检索的值（如上所述，每个值都对应到当前的输入数据的 Key，因此算子接收到的每个Key都可能对应一个值）。这个值可以通过 update(T)进行更新，通过 T value() 进行检索。

#### ListState<T>
保存一个元素的列表。可以往这个列表中追加数据，并在当前的列表上进行检索。可以通过 add(T) 或者 addAll(List<T>) 进行添加元素，通过 Iterable<T> get() 获取整个列表。还可以通过 update(List<T>) 覆盖当前的列表。

#### MapState<UK,UV>
维护了一个添加映射列表。你可以添加键值对到状态中，也可以获得反映当前所有映射的迭代器。使用 put(UK,UV) 或者 putAll(Map<UK,UV>) 分别检索映射、键和值的可迭代视图。

#### ReducingState<T>
保存一个单值，表示添加到状态的所有聚合。接口与 ListState 类似，使用 add(T)增加元素，会使用提供的 ReduceFunction 进行聚合

#### AggregatingState<IN.OUT>
保留一个单值，表示添加到状态的所以值的聚合。和ReducingState 相反的额是聚合类型可能与添加到状态的元素的类型不同。接口与 ListState 类似，但使用 add(IN) 添加的元素会用指定的 AggregateFunction 进行聚合。

#### FoldingState<T,ACC>
保留一个单值，表示添加到状态的所有值的聚合。与 ReducingState 相反，聚合类型可能与添加到状态的元素类型不同。接口与ListState类型，但使用 add(T) 添加的元素会用指定的FoldFunction 折叠成聚合值。

### Keyed State案例演示
### ValueState 使用valueState实现平均值求取
```scala
package xin.studytime.scala

import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.util.Collector

object ValueStateOperate {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    env.fromCollection(List(
      (1L, 3d),
      (1L, 5d),
      (1L, 7d),
      (1L, 4d),
      (1L, 2d)
    ))
      .keyBy(_._1)
      .flatMap(new CountAverageWithValue())
      .print()
    env.execute()
  }
}

class CountAverageWithValue extends RichFlatMapFunction[(Long, Double), (Long, Double)] {
  //定义ValueState类型的变量

  private var sum: ValueState[(Long,Double)] = _
  override def open(parameters: Configuration): Unit = {
    //初始化获取历史状态的值
    val average = new ValueStateDescriptor[(Long, Double)]("average", classOf[(Long, Double)])
    sum = getRuntimeContext.getState(average)
  }

  override def flatMap(input: (Long, Double), out: Collector[(Long, Double)]): Unit = {
    // access the state value
    val tmpCurrentSum = sum.value
    // If it hasn't been used before, it will be null
    val currentSum = if (tmpCurrentSum != null) {
      tmpCurrentSum
    } else {
      (0L, 0d)
    }
    // update the count
    val newSum = (currentSum._1 + 1, currentSum._2 + input._2)

    // update the state
    sum.update(newSum)

    // if the count reaches 2, emit the average and clear the state
    if (newSum._1 >= 2) {
      out.collect((input._1, newSum._2 / newSum._1))
      //将状态清除
      //sum.clear()
    }
  }
}
```
### ListState 求取数据平均值
```scala
package xin.studytime.scala

import java.lang
import java.util.Collections

import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.util.Collector
import scala.collection.JavaConverters._

object ListStateOperate {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._
    env.fromCollection(List(
      (1L, 3d),
      (1L, 5d),
      (1L, 7d),
      (2L, 4d),
      (2L, 2d),
      (2L, 6d)
    )).keyBy(_._1)
      .flatMap(new CountAverageWithList)
      .print()
    env.execute()
  }
}

class CountAverageWithList extends RichFlatMapFunction[(Long, Double), (Long, Double)] {
  private var elementsByKey: ListState[(Long, Double)] = _

  override def open(parameters: Configuration): Unit = {
    val listState = new ListStateDescriptor[(Long, Double)]("listState", classOf[(Long, Double)])
    elementsByKey = getRuntimeContext.getListState(listState)
  }

  override def flatMap(in: (Long, Double), collector: Collector[(Long, Double)]): Unit = {
    val currentState: lang.Iterable[(Long, Double)] = elementsByKey.get()

    if (currentState == null) {
      elementsByKey.addAll(Collections.emptyList())
    }
    elementsByKey.add(in)
    val allElements: Iterator[(Long, Double)] = elementsByKey.get().iterator().asScala
    val allElementList: List[(Long, Double)] = allElements.toList
    if (allElementList.size > 3) {
      var count = 0L
      var sum = 0d
      for (eachElement <- allElementList) {
        count += 1
        sum += eachElement._2
      }
      collector.collect(in._1, sum / count)
    }
  }
}
```
### MapState 求取数据平均值
```scala
package xin.studytime.scala

import java.util.UUID

import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.util.Collector
import org.apache.flink.api.common.state.{MapState, MapStateDescriptor}

object MapStateOperate {
  def main(args: Array[String]): Unit = {
    val environment = StreamExecutionEnvironment.getExecutionEnvironment
    import org.apache.flink.api.scala._

    environment.fromCollection(List(
      (1L, 3d),
      (1L, 5d),
      (1L, 7d),
      (2L, 4d),
      (2L, 2d),
      (2L, 6d)
    )).keyBy(_._1).flatMap(new CountAverageMapState).print()

    environment.execute()
  }
}

class CountAverageMapState extends RichFlatMapFunction[(Long, Double), (Long, Double)] {

  private var mapState: MapState[String, Double] = _

  override def open(parameters: Configuration): Unit = {
    val mapStateOperate = new MapStateDescriptor[String, Double]("mapStateOperate", classOf[String], classOf[Double])
    mapState = getRuntimeContext.getMapState(mapStateOperate)
  }

  override def flatMap(in: (Long, Double), out: Collector[(Long, Double)]): Unit = {

    //将相同的key对应的数据放到一个map集合当中去，就是这种对应  key -> Map((key1, value1),(key2, value2))
    //每次都构建一个map集合
    mapState.put(UUID.randomUUID().toString, in._2)
    import scala.collection.JavaConverters._

    //获取map集合当中所有的value，我们每次将数据的value给放到map的value里面去
    val listState: List[Double] = mapState.values().iterator().asScala.toList
    if (listState.size >= 3) {
      var count = 0L
      var sum = 0d
      for (eachState <- listState) {
        count += 1
        sum += eachState
      }
      out.collect(in._1, sum / count)
    }
  }
}
```
### ReducingState、AggregatingState等考虑实际场景使用不多，就不在此列举。

## Operator State(算子状态)
Operator State 与 Key 无关，而是与 Operator 绑定，整个 Operator 只对应一个 State。比如：Flink 中的 Kafka Connector 就使用了 Operator State，它会在每个 Connector 实例中，保存该实例消费 Topic 的所有（partition，offset）映射，operator state 只有一种托管状态：ValueState

### Operator State案例演示

## Flink的状态管理之State Backend
默认情况下，state会保存在taskmanager的内存中，checkpoint会存储在JobManager的内存中。state 的存储和checkpoint的位置取决于State Backend的配置。Flink一共提供了3种StateBackend，MemoryStateBackend基于内存存储、FsStateBackend基于文件系统存储、RocksDBStateBackend基于数据库存储。可以通过  `StreamExecutionEnvironment.setStateBackend(...)`来设置state存储的位置。

### MemoryStateBackend
将数据持久化状态存储到内存当中，state数据保存在java堆内存中，执行checkpoint的时候，会把state的快照数据保存到jobmanager的内存中。基于内存的state backend在生产环境下不建议使用。

![](https://static.studytime.xin/article/2020/09/15997351713923.jpg)

#### 代码配置
`environment.setStateBackend(new MemoryStateBackend())` 设置MemoryStateBackend。

#### 使用场景
- 本地调试场景
- flink任务状态数据量较小的场景

### FsStateBackend
state数据保存在taskmanager的内存中，执行checkpoint的时候，会把state的快照数据保存到配置的文件系统中。可以使用hdfs等分布式文件系统.

![](https://static.studytime.xin/article/2020/09/15997354537411.jpg)

#### 代码配置
`environment.setStateBackend(new FsStateBackend("hdfs://node01:8020/flink/checkDir"))` 

#### 使用场景
- 状态数据特别的多，还有长时间的window算子等，它很安全，因为基于hdfs，所以数据有备份很安全
- 大状态、长窗口、大key/value状态的的任务
- 全高可用配置

### RocksDBStateBackend
RocksDB介绍：
	RocksDB使用一套日志结构的数据库引擎，它是Flink中内置的第三方状态管理器,为了更好的性能，这套引擎是用C++编写的。 Key和value是任意大小的字节流。RocksDB跟上面的都略有不同，它会在本地文件系统中维护状态，state会直接写入本地rocksdb中。同时它需要配置一个远端的filesystem uri（一般是HDFS），在做checkpoint的时候，会把本地的数据直接复制到fileSystem中。fail over的时候从fileSystem中恢复到本地RocksDB克服了state受内存限制的缺点，同时又能够持久化到远端文件系统中，比较适合在生产中使用.
	
	![](https://static.studytime.xin/article/2020/09/15997362546353.jpg)

#### 代码配置
```scala
# 导入jar包然后配置代码
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
    <version>1.9.2</version>
</dependency>

# 代码配置
environment.setStateBackend(new RocksDBStateBackend("hdfs://node01:8020/flink/checkDir",true))
```
### 使用场景
- 大状态、长窗口、大key/value状态的的任务
- 全高可用配置
    
### 如何选择以及使用state-backend
由于RocksDBStateBackend将工作状态存储在taskManger的本地文件系统，状态数量仅仅受限于本地磁盘容量限制，对比于FsStateBackend保存工作状态在内存中，RocksDBStateBackend能避免flink任务持续运行可能导致的状态数量暴增而内存不足的情况，因此适合在生产环境使用。

#### 修改state-backend的两种方式
##### 单任务调整
```
env.setStateBackend(
new FsStateBackend("hdfs://node01:8020/flink/checkDir"))
或者new MemoryStateBackend()
或者new RocksDBStateBackend(filebackend, true);【需要添加第三方依赖】
```
#### 全局调整
```
vim flink-conf.yaml

state.backend: filesystem
state.checkpoints.dir: hdfs://node01:8020/flink/checkDir
```
注意：state.backend的值可以是下面几种
- jobmanager    表示使用 MemoryStateBackend
- filesystem    表示使用 FsStateBackend
- rocksdb       表示使用 RocksDBStateBackend

