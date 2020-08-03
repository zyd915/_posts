---
title: Spark RDD的依赖关系以及DAG划分stage
permalink: spark-knowledge-rdd-stage
date: 2020-08-02 14:50:20
updated: 2020-08-02 14:50:23
tags:
    - 大数据
    - spark
categories: spark
toc: true
excerpt: 由于 RDD 是粗粒度的操作数据集，每个 Transformation 操作都会生成一个新的 RDD，所以 RDD 之间就会形成类似流水线的前后依赖关系；RDD 和它依赖的父 RDD（s）的关系有两种不同的类型，即窄依赖（Narrow Dependency）和宽依赖（Wide Dependency）。
---
## RDD 的宽依赖和窄依赖

![宽依赖和窄依赖深度剖析图](https://static.studytime.xin/image/articles/spring-boot1900685-74407830ac62d852.png?x-oss-process=image/resize,w_1100)

由于 RDD 是粗粒度的操作数据集，每个 Transformation 操作都会生成一个新的 RDD，所以 RDD 之间就会形成类似流水线的前后依赖关系；RDD 和它依赖的父 RDD（s）的关系有两种不同的类型，即窄依赖（Narrow Dependency）和宽依赖（Wide Dependency）。

### 窄依赖
指的是子 RDD 只依赖于父 RDD 中一个固定数量的分区。

### 宽依赖
指的是子 RDD 的每一个分区都依赖于父 RDD 的所有分区。


由上图可知，join分为宽依赖和窄依赖，如果RDD有相同的partitioner，那么将不会引起shuffle，这种join是窄依赖，反之就是宽依赖

## DAG划分stage

![](https://static.studytime.xin/article/20200803003421.png)

### stage是什么

![](https://static.studytime.xin/image/articles/spring-boot1900685-e784179c0fd1f80c.png?x-oss-process=image/resize,w_1100)

在 Spark 中，Spark 会将每一个 Job 分为多个不同的 Stage, 而 Stage 之间的依赖关系则形成了有向无环图，Spark 会根据 RDD 之间的依赖关系将 DAG 图（有向无环图）划分为不同的阶段，对于窄依赖，由于 Partition 依赖关系的确定性，Partition 的转换处理就可以在同一个线程里完成，窄依赖就被 Spark 划分到同一个 stage 中，而对于宽依赖，只能等父RDD shuffle 处理完成后，下一个 stage 才能开始接下来的计算。

### stage类型
#### ShuffleMapStage
最后一个shuffle之前的所有变换的Stage叫ShuffleMapStage，对应的task是shuffleMapTask。
#### ResultStag
最后一个shuffle之后操作的Stage叫ResultStage，它是最后一个Stage。它对应的task是ResultTask.

### 为什么要划分stage
根据RDD之间依赖关系的不同将DAG划分成不同的Stage(调度阶段)
对于窄依赖，partition的转换处理在一个Stage中完成计算
对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，

由于划分完stage之后，在同一个stage中只有窄依赖，没有宽依赖，可以实现流水线计算，
stage中的每一个分区对应一个task，在同一个stage中就有很多可以并行运行的task。


###  如何划分stage
#### 划分stage的依据就是宽依赖
- 首先根据rdd的算子操作顺序生成DAG有向无环图，接下里从最后一个rdd往前推，创建一个新的stage，把该rdd加入到该stage中，它是最后一个stage。
- 在往前推的过程中运行遇到了窄依赖就把该rdd加入到本stage中，如果遇到了宽依赖，就从宽依赖切开，那么最后一个stage也就结束了。
- 重新创建一个新的stage，按照第二个步骤继续往前推，一直到最开始的rdd，整个划分stage也就结束了

![](https://static.studytime.xin/article/20200803003529.png)

### stage与stage之间的关系
划分完stage之后，每一个stage中有很多可以并行运行的task，后期把每一个stage中的task封装在一个taskSet集合中，最后把一个一个的taskSet集合提交到worker节点上的executor进程中运行。

rdd与rdd之间存在依赖关系，stage与stage之前也存在依赖关系，前面stage中的task先运行，运行完成了再运行后面stage中的task，也就是说后面stage中的task输入数据是前面stage中task的输出结果数据。

![](https://static.studytime.xin/article/20200803003558.png)
