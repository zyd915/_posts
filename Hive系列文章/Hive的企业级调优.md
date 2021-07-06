---
title: Hive的企业级调优
permalink: hive-optimization
date: 2020-06-10 13:48:50
updated: 2020-06-10 13:48:52
tags: 
    - hive
categories: [大数据,hive]
toc: true
excerpt: 整理汇总hive使用过程中企业级调优。
---

## hive的企业级调优
### Fetch抓取
Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。

`select * from score;`，在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。

在hive-default.xml.template文件中 hive.fetch.task.conversion默认是more，老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。

把 hive.fetch.task.conversion设置成**none**，然后执行查询语句，都会执行mapreduce程序

```sql
set hive.fetch.task.conversion=none;
select * from score;
select s_id from score;
select s_id from score limit 3;
```

把hive.fetch.task.conversion设置成more，然后执行查询语句，如下查询方式都不会执行mapreduce程序。

```sql
set hive.fetch.task.conversion=more;
select * from score;
select s_id from score;
select s_id from score limit 3;
```

### 本地模式
在Hive客户端测试时，默认情况下是启用hadoop的job模式,把任务提交到集群中运行，这样会导致计算非常缓慢，Hive可以通过本地模式在单台机器上处理任务。对于小数据集，执行时间可以明显被缩短。

```sql
--未开启开启本地模式，并执行查询语句
set hive.exec.mode.local.auto;
select * from score cluster by s_id;

--开启本地模式，并执行查询语句；默认false，开启本地mr
set hive.exec.mode.local.auto=true;

--设置local mr的最大输入数据量，当输入数据量小于这个值时采用local  mr的方式，
--默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;

--设置local mr的最大输入文件个数，当输入文件个数小于这个值时采用local mr的方式，
--默认为4
set hive.exec.mode.local.auto.input.files.max=5;

--执行查询的sql语句
select * from score cluster by s_id;

 --关闭本地运行模式
set hive.exec.mode.local.auto=false;
```

### 表的优化

#### 小表、大表 join
将key相对分散，并且数据量小的表放在join的左边，这样可以有效减少内存溢出错误发生的几率；再进一步，可以使用map join让小的维度表（1000条以下的记录条数）先进内存，在map端完成reduce；多个表关联时，最好分拆成小段，避免大sql（无法控制中间Job）。

> 实际测试发现：新版的hive已经对小表 join 大表和大表 join 小表进行了优化。小表放在左边和右边已经没有明显区别。

#### 大表 join 大表

#### 空 key 过滤
有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。

此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。
```sql
## 测试环境准备
use myhive;
create table ori(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

create table nullidtable(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

create table jointable(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

load data local inpath '/opt/module/hivedatas/hive_big_table/*' into table ori; 
load data local inpath '/opt/module/hivedatas/hive_have_null_id/*' into table nullidtable;
```

过滤空key与不过滤空key的结果比较
```sql
不过滤：
INSERT OVERWRITE TABLE jointable
SELECT a.* FROM nullidtable a JOIN ori b ON a.id = b.id;
结果：
No rows affected (152.135 seconds)

过滤：
INSERT OVERWRITE TABLE jointable
SELECT a.* FROM (SELECT * FROM nullidtable WHERE id IS NOT NULL ) a JOIN ori b ON a.id = b.id;
结果：
No rows affected (141.585 seconds)
```

#### 空 key 转换
有时虽然某个 key 为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在 join 的结果中，此时我们可以表 a 中 key 为空l使得数据随机均匀地分不到不同的 reducer 上。

不随机分布：

```sql
-- 默认值256000000，即256m
set hive.exec.reducers.bytes.per.reducer=32123456;
set mapreduce.job.reduces=7;

INSERT OVERWRITE TABLE jointable
SELECT a.*
FROM nullidtable a
LEFT JOIN ori b ON CASE WHEN a.id IS NULL THEN 'hive' ELSE a.id END = b.id;
No rows affected (41.668 seconds)  
```

这样的后果就是所有为null值的id全部都变成了相同的字符串，及其容易造成数据的倾斜（所有的key相同，相同key的数据会到同一个reduce当中去。

为了解决这种情况，我们可以通过hive的rand函数，随记的给每一个为空的id赋上一个随机值，这样就不会造成数据倾斜。

```sql
set hive.exec.reducers.bytes.per.reducer=32123456;
set mapreduce.job.reduces=7;

INSERT OVERWRITE TABLE jointable
SELECT a.*
FROM nullidtable a
LEFT JOIN ori b ON CASE WHEN a.id IS NULL THEN concat('hive', rand()) ELSE a.id END = b.id;

No rows affected (42.594 seconds) 
```

#### map join 
大表join小表与小表join大表时，如果不指定MapJoin 或者不符合 MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用 MapJoin 把小表全部加载到内存，在map端进行join，避免reducer处理。
```sql
set hive.auto.convert.join = true;

# 大表小表的阈值设置（默认25M一下认为是小表）
set hive.mapjoin.smalltable.filesize=26214400;
```


#### group By
默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了,并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

```sql
--是否在Map端进行聚合，默认为True
set hive.map.aggr = true;
--在Map端进行聚合操作的条目数目；默认100000
set hive.groupby.mapaggr.checkinterval = 100000;
--有数据倾斜的时候进行负载均衡（默认是false）
set hive.groupby.skewindata = true;

当选项设定为 true，生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。
```

#### count(distinct) 
数据量小的时候无所谓，数据量大的情况下，由于count distinct 操作需要用一个reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般count distinct使用先group by 再count的方式替换

```sql
select  count(distinct ip )  from log_text;

-- 转换成, 虽然会多用一个Job来完成，但在数据量大的情况下，这个绝对是值得的。
set hive.exec.reducers.bytes.per.reducer=32123456;
select count(ip) from (select ip from log_text group by ip) t;
```

#### 笛卡尔积
尽量避免笛卡尔积，即避免join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。


### 使用分区剪裁、列剪裁
尽可能早地过滤掉尽可能多的数据量，避免大量数据流入外层SQL。

#### 列剪裁
只获取需要的列的数据，减少数据输入，少用select  *
#### 分区裁剪
分区在hive实质上是目录，分区裁剪可以方便直接地过滤掉大部分数据，尽量使用分区过滤

```sql
SELECT a.id
FROM bigtable a
LEFT JOIN ori b ON a.id = b.id
WHERE b.id <= 10;
```

正确的写法是写在ON后面：先过滤再关联

```sql
SELECT a.id
FROM ori a
LEFT JOIN bigtable b ON (a.id <= 10 AND a.id = b.id);
```

或者直接写成子查询：

```sql
SELECT a.id
FROM bigtable a
RIGHT JOIN (SELECT id
FROM ori
WHERE id <= 10
) b ON a.id = b.id;
```

### 并行执行
把一个sql语句中没有相互依赖的阶段并行去运行，提高集群资源利用率

```sql
--开启并行执行；默认false
set hive.exec.parallel=true;
--同一个sql允许最大并行度，默认为8。
set hive.exec.parallel.thread.number=16;
```



### 严格模式
Hive提供了一个严格模式，可以防止用户执行那些可能意想不到的不好的影响的查询。

```sql
--设置非严格模式（默认nonstrict）
set hive.mapred.mode=nonstrict;

--设置严格模式
set hive.mapred.mode=strict;
```

通过设置属性hive.mapred.mode值为默认是非严格模式nonstrict 。开启严格模式需要修改hive.mapred.mode值为stric，开启严格模式可以禁止3种类型的查询。

#### 对于分区表，除非where语句中含有分区字段过滤条件来限制范围，否则不允许执行

```sql
--设置严格模式下 执行sql语句报错； 非严格模式下是可以的
select * from score; -- score是个分区表

异常信息：Error: Error while compiling statement: FAILED: SemanticException [Error 10041]: No partition predicate found for Alias "order_partition" Table "order_partition" 
```

#### 对于使用了order by语句的查询，要求必须使用limit语句

```sql
--设置严格模式下 执行sql语句报错； 非严格模式下是可以的
select * from score where month='201806' order by s_score; 

异常信息：Error: Error while compiling statement: FAILED: SemanticException 1:61 In strict mode, if ORDER BY is specified, LIMIT must also be specified. Error encountered near token 'order_price'
```

#### 限制笛卡尔积的查询，严格模式下，避免出现笛卡尔积的查询



### JVM重用
JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是对于很难避免小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。

Hadoop的默认配置通常是使用派生JVM来执行map和Reduce任务的。这时JVM的启动过程可能会造成相当大的开销，尤其是执行的job包含有成百上千task任务的情况。JVM重用可以使得JVM实例在同一个job中重新使用N次。N的值可以在Hadoop的mapred-site.xml文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出。

```xml
<property>
<name>mapreduce.job.jvm.numtasks</name>
<value>10</value>
<description>How many tasks to run per jvm. If set to -1, there is
no limit. 
</description>
</property>
```

```sql
set mapred.job.reuse.jvm.num.tasks=10;
```

这个功能的缺点是，开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡的”job中有某几个reduce task执行的时间要比其他Reduce task消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。

### 压缩
Hive表中间数据压缩

```shell
# 设置为true为激活中间数据压缩功能，默认是false，没有开启
set hive.exec.compress.intermediate=true;
#设置中间数据的压缩算法
set mapred.map.output.compression.codec= org.apache.hadoop.io.compress.SnappyCodec;
```


### 使用EXPLAIN（执行计划）
```sql
explain select * from score where month='201806';
```

### 排序选择
cluster by：对同一字段分桶并排序，不能和 sort by 连用
distribute by + sort by：分桶，保证同一字段值只存在一个结果文件当中，结合 sort by 保证 每个 reduceTask 结果有序
sort by：单机排序，单个 reduce 结果有序
order by：全局排序，缺陷是只能使用一个 reduce



### 数据倾斜

#### 合理设置Map数

1. 通常情况下，作业会通过input的目录产生一个或者多个map任务。

```
主要的决定因素有：input的文件总个数，input的文件大小，集群设置的文件块大小。

举例：
a)  假设input目录下有1个文件a，大小为780M，那么hadoop会将该文件a分隔成7个块（6个128m的块和1个12m的块），从而产生7个map数。
b) 假设input目录下有3个文件a，b，c大小分别为10m，20m，150m，那么hadoop会分隔成4个块（10m，20m，128m，22m），从而产生4个map数。即，如果文件大于块大小(128m)，那么会拆分，如果小于块大小，则把该文件当成一个块。
```

2. 是不是map数越多越好？

```
答案是否定的。如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。
```

3. 是不是保证每个map处理接近128m的文件块，就高枕无忧了？

```
答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时。

针对上面的问题2和3，我们需要采取两种方式来解决：即减少map数和增加map数；

```

#### 小文件合并
在map执行前合并小文件，减少map数，CombineHiveInputFormat 具有对小文件进行合并的功能（系统默认的格式）

```sql

##每个 Map 最大分割大小
set mapred.max.split.size=112345600;
set mapred.min.split.size.per.node=112345600;
set mapred.min.split.size.per.rack=112345600;
```

这个参数表示执行前进行小文件合并，前面三个参数确定合并文件块的大小，大于文件块大小128m的，按照128m来分隔，小于128m，大于100m的，按照100m来分隔，把那些小于100m的（包括小文件和分隔大文件剩下的），进行合并。

#### 复杂文件增加Map数
当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。

##### 增加map的方法为
根据 `computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))`公式
调整maxSize最大值,让maxSize最大值低于blocksize就可以增加map的个数。

```
mapreduce.input.fileinputformat.split.minsize=1 默认值为1

mapreduce.input.fileinputformat.split.maxsize=Long.MAXValue 默认值Long.MAXValue因此，默认情况下，切片大小=blocksize 

maxsize（切片最大值): 参数如果调到比blocksize小，则会让切片变小，而且就等于配置的这个参数的值。

minsize(切片最小值): 参数调的比blockSize大，则可以让切片变得比blocksize还大。
```

#### 合理设置Reduce数
##### 调整reduce个数方法一
```sql
每个Reduce处理的数据量默认是256MB
set hive.exec.reducers.bytes.per.reducer=256000000;

每个任务最大的reduce数，默认为1009
set hive.exec.reducers.max=1009;

计算reducer数的公式
N=min(参数2，总输入数据量/参数1)
```


##### 调整reduce个数方法二

```sql
--设置每一个job中reduce个数
set mapreduce.job.reduces=3;
```

##### reduce个数并不是越多越好
- 过多的启动和初始化reduce也会消耗时间和资源；
- 同时过多的reduce会生成很多个文件，也有可能出现小文件问题

