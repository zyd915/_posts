---
title: Hbase学习之路（五）HBase数据存储机制
permalink: hbase-data-store-operate
date: 2020-06-13 19:33:22
updated: 2020-06-13 19:33:23
tags: 
    - bigdata
    - hbase
categories: [大数据,hbase]
toc: true
excerpt: HBase 采用了经典的 master/slave 架构，与 Hdfs 不同的是，他的 master 与 slave 不直接互联，而是引入 zookeeper 让两类服务解耦，这样使得 master 变得完全无状态，而避免了 master 宕机导致的整个集群不可用。
---

## HBase的数据存储原理
![](https://static.studytime.xin/article/2020/09/16009436955258.jpg)
![](https://static.studytime.xin/article/2020/09/16009437221730.jpg)
HBase 采用了经典的 master/slave 架构，与 Hdfs 不同的是，他的 master 与 slave 不直接互联，而是引入 zookeeper 让两类服务解耦，这样使得 master 变得完全无状态，而避免了 master 宕机导致的整个集群不可用。
同时从 HBase 的架构图上可以看出，HBase 中的组件包括 Client、Zookeeper、HMaster、HRegionServer、HRegion、Store、MemStore、StoreFile、HFile、HLog等。下面将会简单介绍他们的功能。

### HBase 组件
###  HMaster 介绍
HMaster 在整个集群中可以存在多个，但只可以有一个主 HMaster。主 HMaster 由 ZooKeeper 选举产生，当主 HMaster 故障后，系统可由 ZooKeeper 动态选举出新的主 HMaster 接管服务。HMaster 是无状态的（所有状态信息均保存在 ZooKeeper ）中，主 HMaster 挂掉后，因为 client 的读写服务是与 ZooKeeper 通信来完成的，所以不受影响。

#### HMaster 的主要职责
- 协调 RegionServer：包括为 RegionServer 分配 Region，均衡各 RegionServer 的负载，发现失效的 RegionServer 并重新分配其上的 Region。
- 元信息管理：管理⽤户对 Table 的增、删、改、查操作，（处理 Schema 更新请求，表的创建，删除，修改，列簇的增加等等）。
- HDFS 上的垃圾文件（HBase）回收

### ZooKeeper
ZooKeeper 内部存储着有关 HBase 的重要元信息和状态信息，担任着 Master 与RegionServer 之间的服务协调角色。

#### ZooKeeper的主要职责
- 保证任何时候，集群中只有一个HMaster，同时出现故障时，选举新的HMaster，解决单点故障问题。
- 存储所有 Region 的寻址入口，-ROOT-表在哪台服务器上。-ROOT-这张表的位置信息
- 实时监控 RegionServer 的状态，将 RegionServer 的上线和下线信息，实时通知master.
- 存储 HBase 的 schema 和 table 元数据。而HMaster则是管理着这些原信息的操作。

### Client
Client 提供 HBase 访问接口，与 RegionServer 交互读写数据，并维护 cache 加快对 HBase 的访问速度。

HBase 有两张特殊表：
.META.：记录了用户所有表拆分出来的的 Region 映射信息，.META.可以有多个 Regoin
-ROOT-：记录了.META.表的 Region 信息，-ROOT-只有一个 Region，无论如何不会分裂

Client 访问用户数据前需要首先访问 ZooKeeper，找到-ROOT-表的 Region 所在的位置，然后访问-ROOT-表，接着访问.META.表，最后才能找到用户数据的位置去访问，中间需要多次网络操作。同时Client端会做cache缓存。

### RegionServer
HRegionServer 内部管理了⼀系列 HRegion 对象，也可以说一个HRegionServer会负责管理很多个region。
RegionServer 负责 Master 分配给他的 Regin 的存储和管理（比如Region切分），并与Client交互，处理读写请求。

### HRegion
每个 HRegion 对应了 Table 中的⼀个 Region，HRegion 中由多个 HStore 组成。
Region 是 HBase 中分布式存储和负载均衡的最小单元，即不同的 Region 可以分别在不同的 Region Server 上，但同一个 Region 是不会拆分到多个 server 上。
Region 按⼤⼩分割的，每个表开始只有⼀个 Region，随着数据增多， Region 不断增⼤，当增⼤到⼀个阀值的时候，Region 就会等分会两个新的 Region，之后会有越来越多的 Region。每⼀个 Region 都包含所有的列簇。

#### Store
一个列族就划分成一个store。如果一个表中只有1个列族，那么每一个region中只有一个store，如果一个表中有N个列族，那么每一个region中有N个store。每个 Store 将数据分为两部分：存储在内存中的数据( MemStore )和 存储在⽂件系统的数据( StoreFile )。HBase 以 store 的大小来判断是否需要切分 region。

#### MemStore
MemStore 是一块内存区域，只存储新增的和修改过的数据，写入的数据会先写入memstore进行缓冲，并在 MemStore 的大小达到一个阀值（默认128MB）时，将数据 flush 到 StoreFile 中。

#### StoreFile
一个store里面有很多个StoreFile, 最后数据是以很多个HFile这种数据结构的文件保存在HDFS上StoreFile是HFile的抽象对象，也可以说StoreFile就等于HFile，每次memstore刷写数据到磁盘，就生成对应的一个新的HFile文件出来。

#### HFile
HFile 是 Hadoop 的二进制格式文件，实际上 StoreFile 就是对 Hfile 做了轻量级包装，即 StoreFile 底层就是 HFile。

#### HLog
HLog(WAL Log)：WAL意为 Write Ahead Log，保存在 HDFS上的日志文件，用来做灾难恢复使用，HLog 记录数据的所有变更，一旦 RegionServer 宕机，就可以从 Log 中进行恢复。
每个 Region Server 维护一个 Hlog,而不是每个 Region 一个。这样不同 region(来自不同 table) 的日志会混在一起，这样做的目的是不断追加单个文件相对于同时写多个文件而言，可以减 少磁盘寻址次数，因此可以提高对 table 的写性能。带来的麻烦是，如果一台 region server 下线，为了恢复其上的 region，需要将 region server 上的 log 进行拆分，然后分发到其它 region server 上进行恢复。

#### BlockCache
BlockCache：读缓存，负责缓存频繁读取的数据，采用了LRU置换策略

![](https://static.studytime.xin/article/2020/09/16009445547083.jpg)

## HBase读数据流程
![](https://static.studytime.xin/article/2020/09/16009446339085.jpg)

> 特殊说明：HBase集群，只有一张meta表，此表只有一个region，该region数据保存在一个HRegionServer上

### HBase读数据流程详解
> 1、客户端首先与zk进行连接；从zk找到meta表的region位置，即meta表的数据存储在某一HRegionServer上；客户端与此HRegionServer建立连接，然后读取meta表中的数据；meta表中存储了所有用户表的region信息，我们可以通过`scan  'hbase:meta'`来查看meta表信息
> 2、根据要查询的namespace、表名和rowkey信息。找到写入数据对应的region信息
> 3、找到这个region对应的regionServer，然后发送请求
> 4、查找并定位到对应的region
> 5、先从memstore查找数据，如果没有，再从BlockCache上读取,HBase上Regionserver的内存分为两个部分:一部分作为Memstore，主要用来写；另外一部分作为BlockCache，主要用于读数据；
> 6、如果BlockCache中也没有找到，再到StoreFile上进行读取,从storeFile中读取到数据之后，不是直接把结果数据返回给客户端，而是把数据先写入到BlockCache中，目的是为了加快后续的查询；然后在返回结果给客户端。



## HBase写数据流程
![](https://static.studytime.xin/article/2020/09/16009450014124.jpg)

> 1、客户端首先从zk找到meta表的region位置，然后读取meta表中的数据，meta表中存储了用户表的region信息
> 2、根据namespace、表名和rowkey信息。找到写入数据对应的region信息
> 3、找到这个region对应的regionServer，然后发送请求
> 4、把数据分别写到HLog（write ahead log）和memstore各一份
> 5、memstore达到阈值后把数据刷到磁盘，生成storeFile文件
> 6、删除HLog中的历史数据

## HBase的flush、compact机制
![](https://static.studytime.xin/article/2020/09/16009450692401.jpg)

### Flush触发条件

#### memstore级别限制
当Region中任意一个MemStore的大小达到了上限（hbase.hregion.memstore.flush.size，默认128MB），会触发Memstore刷新。

```xml
<property>
	<name>hbase.hregion.memstore.flush.size</name>
	<value>134217728</value>
</property>
```

#### region级别限制
当Region中所有Memstore的大小总和达到了上限（hbase.hregion.memstore.block.multiplier * hbase.hregion.memstore.flush.size，默认 2* 128M = 256M），会触发memstore刷新。

```xml
<property>
	<name>hbase.hregion.memstore.flush.size</name>
	<value>134217728</value>
</property>
<property>
	<name>hbase.hregion.memstore.block.multiplier</name>
	<value>2</value>
</property>   
```

#### Region Server级别限制
当一个Region Server中所有Memstore的大小总和超过低水位阈值hbase.regionserver.global.memstore.size.lower.limit*hbase.regionserver.global.memstore.size（前者默认值0.95），RegionServer开始强制flush；
- 先Flush Memstore最大的Region，再执行次大的，依次执行；
- 如写入速度大于flush写出的速度，导致总MemStore大小超过高水位阈值hbase.regionserver.global.memstore.size（默认为JVM内存的40%），此时RegionServer会阻塞更新并强制执行flush，直到总MemStore大小低于低水位阈值

```xml
<property>
	<name>hbase.regionserver.global.memstore.size.lower.limit</name>
	<value>0.95</value>
</property>
<property>
	<name>hbase.regionserver.global.memstore.size</name>
	<value>0.4</value>
</property>
```

#### HLog数量上限
当一个Region Server中HLog数量达到上限（可通过参数hbase.regionserver.maxlogs配置）时，系统会选取最早的一个 HLog对应的一个或多个Region进行flush

#### 定期刷新Memstore
默认周期为1小时，确保Memstore不会长时间没有持久化。为避免所有的MemStore在同一时间都进行flush导致的问题，定期的flush操作有20000左右的随机延时。

#### 4.1.6 手动flush
用户可以通过shell命令`flush ‘tablename’`或者`flush ‘region name’`分别对一个表或者一个Region进行flush。

### flush的流程
为了减少flush过程对读写的影响，将整个flush过程分为三个阶段：
  - prepare阶段：遍历当前Region中所有的Memstore，将Memstore中当前数据集CellSkipListSet做一个**快照snapshot**；然后再新建一个CellSkipListSet。后期写入的数据都会写入新的CellSkipListSet中。prepare阶段需要加一把updateLock对**写请求阻塞**，结束之后会释放该锁。因为此阶段没有任何费时操作，因此持锁时间很短。
  - flush阶段：遍历所有Memstore，将prepare阶段生成的snapshot持久化为**临时文件**，临时文件会统一放到目录.tmp下。这个过程因为涉及到磁盘IO操作，因此相对比较耗时。
  - commit阶段：遍历所有Memstore，将flush阶段生成的临时文件移到指定的ColumnFamily目录下，针对HFile生成对应的storefile和Reader，把storefile添加到HStore的storefiles列表中，最后再**清空**prepare阶段生成的snapshot。

### Compact合并机制
hbase为了防止小文件过多，以保证查询效率，hbase需要在必要的时候将这些小的store file合并成相对较大的store file，这个过程就称之为compaction。
在hbase中主要存在两种类型的compaction合并
  - inor compaction 小合并
  - major compaction 大合并

#### minor compaction 小合并
在将Store中多个HFile合并为一个HFile，在这个过程中会选取一些小的、相邻的StoreFile将他们合并成一个更大的StoreFile，对于超过了TTL的数据、更新的数据、删除的数据仅仅只是做了标记。并没有进行物理删除，一次Minor Compaction的结果是更少并且更大的StoreFile。这种合并的触发频率很高。

#### minor compaction触发条件由以下几个参数共同决定：
```xml
<!--表示至少需要三个满足条件的store file时，minor compaction才会启动-->
<property>
	<name>hbase.hstore.compactionThreshold</name>
	<value>3</value>
</property>

<!--表示一次minor compaction中最多选取10个store file-->
<property>
	<name>hbase.hstore.compaction.max</name>
	<value>10</value>
</property>

<!--默认值为128m,
表示文件大小小于该值的store file 一定会加入到minor compaction的store file中
-->
<property>
	<name>hbase.hstore.compaction.min.size</name>
	<value>134217728</value>
</property>

<!--默认值为LONG.MAX_VALUE，
表示文件大小大于该值的store file 一定会被minor compaction排除-->
<property>
	<name>hbase.hstore.compaction.max.size</name>
	<value>9223372036854775807</value>
</property>
```

#### major compaction 大合并
合并Store中所有的HFile为一个HFile, 将所有的StoreFile合并成一个StoreFile，这个过程还会清理三类无意义数据：被删除的数据、TTL过期数据、版本号超过设定版本号的数据。合并频率比较低，默认7天执行一次，并且性能消耗非常大，建议生产关闭(设置为0)，在应用空闲时间手动触发。一般可以是手动控制进行合并，防止出现在业务高峰期。
  
##### major compaction触发时间条件
 ```xml
  <!--默认值为7天进行一次大合并，-->
  <property>
  	<name>hbase.hregion.majorcompaction</name>
  	<value>604800000</value>
  </property>
```

##### 手动触发
 ```ruby
  ##使用major_compact命令
  major_compact tableName
 ```



## 5. region 拆分机制
region中存储的是大量的rowkey数据 ,当region中的数据条数过多的时候,直接影响查询效率.当region过大的时候.hbase会拆分region , 这也是Hbase的一个优点 .

### HBase的region split策略一共有以下几种：

#### 1、ConstantSizeRegionSplitPolicy, 0.94版本前默认切分策略
当region大小大于某个阈值(hbase.hregion.max.filesize=10G)之后就会触发切分，一个region等分为2个region。
但是在生产线上这种切分策略却有相当大的弊端：切分策略对于大表和小表没有明显的区分。阈值(hbase.hregion.max.filesize)设置较大对大表比较友好，但是小表就有可能不会触发分裂，极端情况下可能就1个，这对业务来说并不是什么好事。如果设置较小则对小表友好，但一个大表就会在整个集群产生大量的region，这对于集群的管理、资源使用、failover来说都不是一件好事。
  
  
  
#### IncreasingToUpperBoundRegionSplitPolicy,0.94版本~2.0版本默认切分策略
切分策略稍微有点复杂，总体看和ConstantSizeRegionSplitPolicy思路相同，一个region大小大于设置阈值就会触发切分。但是这个阈值并不像ConstantSizeRegionSplitPolicy是一个固定的值，而是会在一定条件下不断调整，调整规则和region所属表在当前regionserver上的region个数有关系.
##### region split的计算公式是：
regioncount^3 * 128M * 2，当region达到该size的时候进行split
例如：
第一次split：1^3 * 256 = 256MB 
第二次split：2^3 * 256 = 2048MB 
第三次split：3^3 * 256 = 6912MB 
第四次split：4^3 * 256 = 16384MB > 10GB，因此取较小的值10GB 
后面每次split的size都是10GB了
  
#### 3、SteppingSplitPolicy,2.0版本默认切分策略
这种切分策略的切分阈值又发生了变化，相IncreasingToUpperBoundRegionSplitPolicy 简单了一些，依然和待分裂region所属表在当前regionserver上的region个数有关系，如果region个数等于1，
切分阈值为flush size * 2，否则为MaxRegionFileSize。这种切分策略对于大集群中的大表、小表会比 IncreasingToUpperBoundRegionSplitPolicy 更加友好，小表不会再产生大量的小region，而是适可而止。
  
#### 4、KeyPrefixRegionSplitPolicy
根据rowKey的前缀对数据进行分组，这里是指定rowKey的前多少位作为前缀，比如rowKey都是16位的，指定前5位是前缀，那么前5位相同的rowKey在进行region split的时候会分到相同的region中。
  
#### 5、DelimitedKeyPrefixRegionSplitPolicy
 保证相同前缀的数据在同一个region中，例如rowKey的格式为：userid_eventtype_eventid，指定的delimiter为 _ ，则split的的时候会确保userid相同的数据在同一个region中。


#### 6、DisabledRegionSplitPolicy,不启用自动拆分, 需要指定手动拆分



## HBase表的预分区
当一个table刚被创建的时候，Hbase默认的分配一个region给table。也就是说这个时候，所有的读写请求都会访问到同一个regionServer的同一个region中，这个时候就达不到负载均衡的效果了，集群中的其他regionServer就可能会处于比较空闲的状态。解决这个问题可以用pre-splitting,在创建table的时候就配置好，生成多个region。

### 为何要预分区？

* 增加数据读写效率
* 负载均衡，防止数据倾斜
* 方便集群容灾调度region
* 优化Map数量

### 预分区原理
每一个region维护着startRow与endRowKey，如果加入的数据符合某个region维护的rowKey范围，则该数据交给这个region维护。

### 手动指定预分区
#### 方式一
```ruby
create 'person','info1','info2',SPLITS => ['1000','2000','3000','4000']
```
![](https://static.studytime.xin/article/2020/09/16009637833340.jpg)


#### 方式二：也可以把分区规则创建于文件中
```shell
  cd /kkb/install
  
  vim split.txt
  
  # 文件内容
  ~~~
  aaa
  bbb
  ccc
  ddd
  
  # hbase shell中，执行命令
create 'student','info',SPLITS_FILE => '/kkb/install/split.txt'

# 成功后查看web界面
 ```
![](https://static.studytime.xin/article/2020/09/16009638635036.jpg)


### HexStringSplit 算法
HexStringSplit会将数据从“00000000”到“FFFFFFFF”之间的数据长度按照**n等分**之后算出每一段的其实rowkey和结束rowkey，以此作为拆分点。
例如：
```ruby
  create 'mytable', 'base_info',' extra_info', {NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
  ```
![](https://static.studytime.xin/article/2020/09/16009638912172.jpg)

## region 合并

### region合并说明
Region的合并不是为了性能,  而是出于维护的目的.比如删除了大量的数据 ,这个时候每个Region都变得很小,存储多个Region就浪费了 ,这个时候可以把Region合并起来，进而可以减少一些Region服务器节点 

### 如何进行region合并

#### 通过Merge类冷合并Region,执行合并前，需要先关闭hbase集群
- 创建一张hbase表：
```ruby
create 'test','info1',SPLITS => ['1000','2000','3000']
```
- 查看表region
![](https://static.studytime.xin/article/2020/09/16009639627285.jpg)
- 需求：
  需要把test表中的2个region数据进行合并：
  test,,1565940912661.62d28d7d20f18debd2e7dac093bc09d8.
  test,1000,1565940912661.5b6f9e8dad3880bcc825826d12e81436.
- 这里通过org.apache.hadoop.hbase.util.Merge类来实现，不需要进入hbase shell，直接执行（需要先关闭hbase集群）：
  hbase org.apache.hadoop.hbase.util.Merge test test,,1565940912661.62d28d7d20f18debd2e7dac093bc09d8. test,1000,1565940912661.5b6f9e8dad3880bcc825826d12e81436.

- 成功后界面观察
![](https://static.studytime.xin/article/2020/09/16009639821724.jpg)

#### 通过online_merge热合并Region,不需要关闭hbase集群，在线进行合并
与冷合并不同的是，online_merge的传参是Region的hash值，而Region的hash值就是Region名称的最后那段在两个.之间的字符串部分。
- 需求：需要把test表中的2个region数据进行合并：
  test,2000,1565940912661.c2212a3956b814a6f0d57a90983a8515.
  test,3000,1565940912661.553dd4db667814cf2f050561167ca030.
- 需要进入hbase shell：
  ```ruby
  merge_region 'c2212a3956b814a6f0d57a90983a8515','553dd4db667814cf2f050561167ca030'
  ```
- 成功后观察界面
![](https://static.studytime.xin/article/2020/09/16009639981316.jpg)







