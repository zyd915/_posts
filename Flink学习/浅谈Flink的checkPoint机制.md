---
title: 浅谈Flink的checkPoint机制 
permalink: flink-checkpoint-and-savepoint
date: 2020-09-11 22:41:41
updated: 2020-09-11 22:41:41
tags:
    - 大数据
    - flink
categories: flink
toc: true
excerpt: 为了保证state的容错性，Flink需要对state进程checkPoint,Checkpoint是Flink实现容错机制最核心的功能。
---

## checkPoint基本概念
为了保证state的容错性，Flink需要对state进程checkPoint。
Checkpoint是Flink实现容错机制最核心的功能，它能够根据配置周期性地基于Stream中各个Operator/task的状态来生成快照，从而将这些状态数据定期持久化存储下来，当Flink程序一旦意外崩溃时，重新运行程序时可以有选择地从这些快照进行恢复，从而修正因为故障带来的程序数据异常。
### checkPoint步骤
- 暂停新数据的输入
- 等待流中on-the-fly的数据被处理干净，此时得到flink graph的一个snapshot
- 将所有Task中的State拷贝到State Backend中，如HDFS。此动作由各个Task Manager完成
- 各个Task Manager将Task State的位置上报给Job Manager，完成checkpoint
- 恢复数据的输入

### 如何设置配置checkPoint
默认checkpoint功能是disabled的，想要使用的时候需要先启用。
checkpoint开启之后，默认的checkPointMode是Exactly-once
checkpoint的checkPointMode有两种：
- Exactly-once: 数据处理且只被处理一次
- At-least-once：数据至少被处理一次
Exactly-once对于大多数应用来说是最合适的。At-least-once可能用在某些延迟超低的应用程序（始终延迟为几毫秒）

```scala
// 每隔1000 ms进行启动一个检查点【设置checkpoint的周期】
environment.enableCheckpointing(1000);

// 高级选项：
// 设置模式为exactly-once （这是默认值）
environment.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
// 确保检查点之间有至少500 ms的间隔【checkpoint最小间隔】
environment.getCheckpointConfig.setMinPauseBetweenCheckpoints(500);
// 检查点必须在一分钟内完成，或者被丢弃【checkpoint的超时时间】
environment.getCheckpointConfig.setCheckpointTimeout(60000);
// 同一时间只允许进行一个检查点
environment.getCheckpointConfig.setMaxConcurrentCheckpoints(1);
// 表示一旦Flink处理程序被cancel后，会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint【详细解释见备注】

/**
  * ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION:表示一旦Flink处理程序被cancel后，会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint
  * ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION: 表示一旦Flink处理程序被cancel后，会删除Checkpoint数据，只有job执行失败的时候才会保存checkpoint
  */
environment.getCheckpointConfig.enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```

### Flink重启策略概述
Flink支持不同的重启策略，以在故障发生时控制作业如何重启，集群在启动时会伴随一个默认的重启策略，在没有定义具体重启策略时会使用该默认策略。
### Flink 重启策略实现
如果在工作提交时指定了一个重启策略，该策略会覆盖集群的默认策略，默认的重启策略可以通过 Flink 的配置文件 flink-conf.yaml 指定。配置参数 restart-strategy 定义了哪个策略被使用。
#### 常用的重启策略
* （1）固定间隔 (Fixed delay)
* （2）失败率 (Failure rate)
* （3）无重启 (No restart)
如果没有启用 checkpointing，则使用无重启 (no restart) 策略。 
如果启用了 checkpointing，重启策略可以在`flink-conf.yaml`中配置，表示全局的配置。也可以在应用代码中动态指定，会覆盖全局配置
但没有配置重启策略，则使用固定间隔 (fixed-delay) 策略， 尝试重启次数默认值是：Integer.MAX_VALUE。

#### 固定间隔 (Fixed delay)
```
第一种：全局配置 flink-conf.yaml
restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s

第二种：应用代码设置,重启次数、重启时间间隔
environment.setRestartStrategy(RestartStrategies.fixedDelayRestart(5,10000))
```

#### 失败率 (Failure rate)
```
第一种：全局配置 flink-conf.yaml
//5分钟内若失败了3次则认为该job失败，重试间隔为10s
restart-strategy: failure-rate
restart-strategy.failure-rate.max-failures-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 10 s


第二种：应用代码设置
environment.setRestartStrategy(RestartStrategies.failureRateRestart(20, org.apache.flink.api.common.time.Time.seconds(100), org.apache.flink.api.common.time.Time.seconds(10)))
```

#### 无重启 (No restart)
```
第一种：全局配置 flink-conf.yaml
restart-strategy: none

第二种：应用代码设置
environment.setRestartStrategy(RestartStrategies.noRestart())
```
### checkPoint保存多个历史版本，恢复多个版本
默认情况下，如果设置了Checkpoint选项，则Flink只保留最近成功生成的1个Checkpoint，而当Flink程序失败时，可以从最近的这个Checkpoint来进行恢复。
如果我们希望保留多个Checkpoint，并能够根据实际需要==选择其中一个进行恢复==，这样会更加灵活，比如，我们发现最近4个小时数据记录处理有问题，希望将整个状态还原到4小时之前。
Flink可以支持保留多个Checkpoint，需要在Flink的配置文件==conf/flink-conf.yaml==中，添加如下配置，指定最多需要保存Checkpoint的个数。
如果希望回退到某个Checkpoint点，只需要指定对应的某个Checkpoint路径即可实现
```
state.checkpoints.num-retained: 20

这样设置以后就查看对应的Checkpoint在HDFS上存储的文件目录
hdfs dfs -ls hdfs://node01:8020/flink/checkpoints
```
### checkPoiny 恢复历史某个版本
如果Flink程序异常失败，或者最近一段时间内数据处理错误，我们可以将程序从某一个Checkpoint点进行恢复。
flink run -m yarn-cluster -yn 2 -yjm 1024 -ytm 1024 -s hdfs://node01:8020/fsStateBackend/971ae7ac4d5f20e704747ea7c549b356/chk-50/_metadata -c com.kaikeba.checkpoint.TestCheckPoint original-flink_study-1.0-SNAPSHOT.jar

## savePoint 基本概念
savePoint是检查点一种特殊实现，底层其实也是使用Checkpoints的机制。
savePoint是用户以手工命令的方式触发checkpoint，并将结果持久化到指定的存储目录中。

### savePoint 使用场景
* 1、应用程序代码升级
  * 通过触发保存点并从该保存点处运行新版本，下游的应用程序并不会察觉到不同
* 2、Flink版本更新
  * Flink 自身的更新也变得简单，因为可以针对正在运行的任务触发保存点，并从保存点处用新版本的 Flink 重启任务。
* 3、维护和迁移
  * 使用保存点，可以轻松地“暂停和恢复”应用程序   
          
### savePoint的使用
#### 在flink-conf.yaml中配置Savepoint存储位置
不是必须设置，但是设置后，后面创建指定Job的Savepoint时，可以不用在手动执行命令时指定Savepoint的位置
```
state.savepoints.dir: hdfs://node01:8020/flink/savepoints
```
#### 触发一个savepoint
##### 手动触发savepoint
```
#【针对on standAlone模式】
bin/flink savepoint jobId [targetDirectory] 

#【针对on yarn模式需要指定-yid参数】
bin/flink savepoint jobId [targetDirectory] [-yid yarnAppId]

#jobId 				需要触发savepoint的jobId编号
#targetDirectory     指定savepoint存储数据目录
#-yid                指定yarnAppId 

##例如：
flink savepoint 8d1bb7f88a486815f9b9cf97c304885b  -yid application_1594807273214_0004
```
##### 取消任务并手动触发savepoint
```
##【针对on standAlone模式】
bin/flink cancel -s [targetDirectory] jobId 

##【针对on yarn模式需要指定-yid参数】
bin/flink cancel -s [targetDirectory] jobId [-yid yarnAppId]

##例如：
flink cancel 8d1bb7f88a486815f9b9cf97c304885b -yid application_1594807273214_0004
```
##### 从指定的savepoint启动job
```
bin/flink run -s savepointPath [runArgs]

##例如：
flink run -m yarn-cluster -yn 2 -yjm 1024 -ytm 1024 -s hdfs://node01:8020/flink/savepoints/savepoint-8d1bb7-c9187993ca94 -c com.kaikeba.checkpoint.TestCheckPoint original-flink_study-1.0-SNAPSHOT.jar
```
##### 清除savepoint数据
```
bin/flink savepoint -d savepointPath
```