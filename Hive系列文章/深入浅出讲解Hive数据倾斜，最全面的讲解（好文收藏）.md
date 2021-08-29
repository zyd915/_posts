---
title: 深入浅出Hive数据倾斜，最全面的讲解（好文收藏）
permalink: hive-knowledge-data-skew
date: 2021-08-28 18:47:13
updated: 2021-08-28 18:47:23
tags:
    - 大数据
    - hive
categories: [大数据,hive]
toc: true
thumbnail: https://static.studytime.xin//studytime/image/articles/vK96xb.jpg
keywords: [Hive数据倾斜, 数据倾斜, 数据倾斜怎么解决, 什么是数据倾斜]
excerpt: 本文将为你深入浅出地讲解什么是Hive数据倾斜、数据倾斜产生的原因以及面对数据倾斜的解决方法，从而帮你快速完成工作！
---

## 背景
我们日常使用HIVE SQL的时候可能会遇到这样一个令人苦恼的场景：执行一个非常简单的SQL语句，任务的进度条长时间卡在99%，不确定还需多久才能结束，这种现象称之为数据倾斜。这一现象出现的原因在于数据研发工程师主要关注分析逻辑和数据结果的正确性，却很少关注SQL语句的执行过程与效率。
 
本文将为你深入浅出地讲解什么是Hive数据倾斜、数据倾斜产生的原因以及面对数据倾斜的解决方法，从而帮你快速完成工作！

## 什么是数据倾斜？
数据倾斜在 MapReduce 计算框架中经常发生。通俗理解，该现象指的是在整个计算过程中，大量相同的key被分配到了同一个任务上，造成“一个人累死、其他人闲死”的状况，这违背了分布式计算的初衷，使得整体的执行效率十分低下。

数据倾斜后直观表现是任务进度长时间维持在99%（或100%），查看任务监控页面，发现只有少量（1个或几个）reduce子任务未完成。因为其处理的数据量和其他reduce差异过大，单一reduce的记录数与平均记录数差异过大，最长时长远大于平均时长。

![数据倾斜表现形式](https://static.studytime.xin//studytime/image/articles/W0EgBY.jpg)

## 什么情况下会出现数据倾斜？
日常工作中数据倾斜主要发生在Reduce阶段，而很少发生在 Map阶段，其原因是Map端的数据倾斜一般是由于 HDFS 数据存储不均匀造成的（一般存储都是均匀分块存储，每个文件大小基本固定），而Reduce阶段的数据倾斜几乎都是因为数据研发工程师没有考虑到某种key值数据量偏多的情况而导致的。

Reduce阶段最容易出现数据倾斜的两个场景分别是Join和Count Distinct。有些博客和文档将Group By也纳入其中，但是我们认为如果不使用Distinct，仅仅对分组数据进行Sum和Count操作，并不会出现数据倾斜，作者的话这里也单独拉出来算作一种场景，毕竟也是会造成的。

| 关键词 | 情形 | 后果 |
| --- | --- | --- |
| Join |其中一个表较小，但是key集中  | 分发到某一个或几个reduce上的数据远高于平均值 |
| Join | 大表与大表，但是分桶的判断字段0值或空值过多 | 空值都由一个reduce处理，非常慢 |
| group by | group by 维度过小，某值的数量过多 | 处理某值的reduce耗时 |
| Count Distinct | 某特殊值过多 | 	处理此特殊值的reduce耗时 |


## 数据倾斜产生的原因？
- key分布不均匀
- 业务数据本身的特性
- 建表时考虑不周
- 某些SQL语句本身就有数据倾斜

## 数据倾斜的解决方案?

### 一、优先开启负载均衡
```sql
-- map端的Combiner,默认为ture
set hive.map.aggr=true；
-- 开启负载均衡
set hive.groupby.skewindata=true （默认为false）
```
如果发生数据倾斜，我们首先需要调整参数，进行负载均衡处理，这样 MapReduce 进程则会生成两个额外的 MR Job，这两个任务的主要操作如下：

第一步：MR Job 中Map 输出的结果集合首先会随机分配到 Reduce 中，然后每个 Reduce 做局部聚合操作并输出结果，这样处理的原因是相同的Group By Key有可能被分发到不同的 Reduce Job中，从而达到负载均衡的目的。
第二步：MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成聚合操作。

### 二、表join连接时引发的数据倾斜
两表进行join时，如果表连接的key存在倾斜，那么在 Shuffle 阶段必然会引起数据倾斜。

#### 2.1 小表join大表，某个key过大
通常做法是将倾斜的数据存到分布式缓存中，分发到各个Map任务所在节点。在Map阶段完成join操作，即MapJoin，这避免了 Shuffle，从而避免了数据倾斜。

MapJoin是Hive的一种优化操作，其适用于小表JOIN大表的场景，由于表的JOIN操作是在Map端且在内存进行的，所以其并不需要启动Reduce任务也就不需要经过shuffle阶段，从而能在一定程度上节省资源提高JOIN效率。

在Hive 0.11版本之前，如果想在Map阶段完成join操作，必须使用MAPJOIN来标记显示地启动该优化操作，由于其需要将小表加载进内存所以要注意小表的大小。

```sql
-- 常规join
SELECT
        pis.id_ id,
        service_name serviceName,
        service_code serviceCode,
        bd.code_text serviceType,
FROM
    prd_price_increment_service pis
    left join prd_price_increment_product pip on pis.id_ = pip.increment_service_id

-- Hive 0.11版本之前开启mapjoin
-- 将小表prd_price_increment_service，别名为pis的表放到map端的内存中
-- 如果想将多个表放到Map端内存中，只需在mapjoin()中写多个表名称即可，用逗号分隔，如将a表和c表放到Map端内存中，则 /* +mapjoin(a,c) */ 。
SELECT
　　　　  /*+ mapjoin(pis)*/
        pis.id_ id,
        service_name serviceName,
        service_code serviceCode,
        bd.code_text serviceType,
FROM
    prd_price_increment_service pis
    left join prd_price_increment_product pip on pis.id_ = pip.increment_service_id
```

在Hive 0.11版本及之后，Hive默认启动该优化，也就是不在需要显示的使用MAPJOIN标记，其会在必要的时候触发该优化操作将普通JOIN转换成MapJoin，可以通过以下两个属性来设置该优化的触发时机：

```sql
-- 自动开启MAPJOIN优化，默认值为true，
set hive.auto.convert.join=true;
-- 通过配置该属性来确定使用该优化的表的大小，如果表的大小小于此值就会被加载进内存中，默认值为2500000(25M)，
set hive.mapjoin.smalltable.filesize=2500000;
SELECT
        pis.id_ id,
        service_name serviceName,
        service_code serviceCode,
        bd.code_text serviceType,
FROM
    prd_price_increment_service pis
    left join prd_price_increment_product pip on pis.id_ = pip.increment_service_id

-- 特殊说明
使用默认启动该优化的方式如果出现莫名其妙的BUG(比如MAPJOIN并不起作用)，就将以下两个属性置为fase手动使用MAPJOIN标记来启动该优化:
-- 关闭自动MAPJOIN转换操作
set hive.auto.convert.join=false;
-- 不忽略MAPJOIN标记
set hive.ignore.mapjoin.hint=false;
SELECT
　　　　  /*+ mapjoin(pis)*/
        pis.id_ id,
        service_name serviceName,
        service_code serviceCode,
        bd.code_text serviceType,
FROM
    prd_price_increment_service pis
    left join prd_price_increment_product pip on pis.id_ = pip.increment_service_id
```

特说说明：将表放到Map端内存时，如果节点的内存很大，但还是出现内存溢出的情况，我们可以通过这个参数 mapreduce.map.memory.mb 调节Map端内存的大小。


#### 2.2 表中作为关联条件的字段值为0或空值的较多
解决方式：给空值添加随机key值，将其分发到不同的reduce中处理。由于null值关联不上，所以对结果无影响。
```sql
-- 方案一、给空值添加随机key值，将其分发到不同的reduce中处理。由于null值关联不上，所以对结果无影响。
SELECT * 
FROM log a left join users b 
on case when a.user_id is null then concat('hive',rand()) else a.user_id end = b.user_id;

-- 方案二：去重空值
SELECT a.*,b.name
FROM 
    (SELECT * FROM users WHERE LENGTH(user_id) > 1 OR user_id IS NOT NULL ) a
JOIN
    (SELECT * FROM log WHERE LENGTH(user_id) > 1 OR user_id IS NOT NULL) B
ON a.user_id; = b.user_id;
```

#### 2.3 表中作为关联条件的字段重复值过多
```sql
SELECT a.*,b.name
FROM 
    (SELECT * FROM users WHERE LENGTH(user_id) > 1 OR user_id IS NOT NULL ) a
JOIN
    (SELECT * FROM (SELECT *,row_number() over(partition by user_id order by create_time desc) rk FROM log WHERE LENGTH(user_id) > 1 OR user_id IS NOT NULL) temp where rk = 1) B
ON a.user_id; = b.user_id;
```

### 三、空值引发的数据倾斜
实际业务中有些大量的null值或者一些无意义的数据参与到计算作业中，表中有大量的null值，如果表之间进行join操作，就会有shuffle产生，这样所有的null值都会被分配到一个reduce中，必然产生数据倾斜。

之前有小伙伴问，如果A、B两表join操作，假如A表中需要join的字段为null，但是B表中需要join的字段不为null，这两个字段根本就join不上啊，为什么还会放到一个reduce中呢?

这里我们需要明确一个概念，数据放到同一个reduce中的原因不是因为字段能不能join上，而是因为shuffle阶段的hash操作，只要key的hash结果是一样的，它们就会被拉到同一个reduce中。

```sql
-- 解决方案
-- 场景：如日志中，常会有信息丢失的问题，比如日志中的 user_id，如果取其中的 user_id 和 用户表中的user_id 关联，会碰到数据倾斜的问题。
-- 方案一：可以直接不让null值参与join操作，即不让null值有shuffle阶段，所以user_id为空的不参与关联
SELECT * FROM log a JOIN users b  ON a.user_id IS NOT NULL   AND a.user_id = b.user_id UNION ALL SELECT * FROM log a WHERE a.user_id IS NULL; 

-- 方案二：因为null值参与shuffle时的hash结果是一样的，那么我们可以给null值随机赋值，这样它们的hash结果就不一样，就会进到不同的reduce中：
SELECT * FROM log a  LEFT JOIN users b ON CASE     WHEN a.user_id IS NULL THEN concat('hive_', rand())    ELSE a.user_id   END = b.user_id;
```

特殊说明：针对上述方案进行分析，方案二比方案一效率更好，不但io少了，而且作业数也少了。方案一中对log读取可两次，jobs是2。方案二job数是1。本优化适合对无效id (比如 -99, ’’, null等) 产生的倾斜问题。把空值的 key 变成一个字符串加上随机数，就能把倾斜的数据分到不同的reduce上,解决数据倾斜问题。

### 四、不同数据类型关联产生数据倾斜

对于两个表join，表a中需要join的字段key为int，表b中key字段既有string类型也有int类型。当按照key进行两个表的join操作时，默认的Hash操作会按int型的id来进行分配，这样所有的string类型都被分配成同一个id，结果就是所有的string类型的字段进入到一个reduce中，引发数据倾斜。

```sql
-- 如果key字段既有string类型也有int类型，默认的hash就都会按int类型来分配，那我们直接把int类型都转为string就好了，这样key字段都为string，hash时就按照string类型分配了：

方案一：把数字类型转换成字符串类型
SELECT * FROM users a  LEFT JOIN logs b ON a.usr_id = CAST(b.user_id AS string); 

方案二：建表时按照规范建设，统一词根，同一词根数据类型一致
```

### 五、count distinct 大量相同特殊值
由于SQL中的Distinct操作本身会有一个全局排序的过程，一般情况下，不建议采用Count Distinct方式进行去重计数，除非表的数量比较小。当SQL中不存在分组字段时，Count Distinct操作仅生成一个Reduce 任务，该任务会对全部数据进行去重统计；当SQL中存在分组字段时，可能某些 Reduce 任务需要去重统计的数量非常大。在这种情况下，我们可以通过以下方式替换。
```sql
-- 可能会造成数据倾斜的sql
select a,count(distinct b) from t group by a;
-- 先去重、然后分组统计
select a,sum(1) from (select a, b from t group by a,b) group by a;
-- 总结: 如果分组统计的数据存在多个distinct结果，可以先将值为空的数据占位处理，分sql统计数据，然后将两组结果union all进行汇总结算。
```

### 六、数据膨胀引发的数据倾斜

在多维聚合计算时，如果进行分组聚合的字段过多，且数据量很大，Map端的聚合不能很好地起到数据压缩的情况下，会导致Map端产出的数据急速膨胀，这种情况容易导致作业内存溢出的异常。如果log表含有数据倾斜key，会加剧Shuffle过程的数据倾斜。
```sql
-- 造成倾斜或内存溢出的情况
-- sql01
select a，b，c，count(1)from log group by a，b，c with rollup;
-- sql02
select a，b，c，count(1)from log grouping sets a，b，c;

-- 解决方案
-- 可以拆分上面的sql，将with rollup拆分成如下几个sql
select a,b,c,sum(1) from (
    SELECT a, b, c, COUNT(1) FROM log GROUP BY a, b, c
    union all
    SELECT a, b, NULL, COUNT(1) FROM log GROUP BY a, b 
    union all
    SELECT a, NULL, NULL, COUNT(1) FROM log GROUP BY a
    union all
    SELECT NULL, NULL, NULL, COUNT(1) FROM log
) temp;
-- 结论：但是上面这种方式不太好，因为现在是对3个字段进行分组聚合，那如果是5个或者10个字段呢，那么需要拆解的SQL语句会更多。

-- 在Hive中可以通过参数 hive.new.job.grouping.set.cardinality 配置的方式自动控制作业的拆解，该参数默认值是30。
-- 该参数主要针对grouping sets/rollups/cubes这类多维聚合的操作生效，如果最后拆解的键组合大于该值，会启用新的任务去处理大于该值之外的组合。如果在处理数据时，某个分组聚合的列有较大的倾斜，可以适当调小该值。
set hive.new.job.grouping.set.cardinality=10;
select a，b，c，count(1)from log group by a，b，c with rollup;
```

## 总结
上文为你深入浅出地讲解什么是Hive数据倾斜、数据倾斜产生的原因以及面对数据倾斜的解决方法。概括而言，让Map端的输出数据更均匀地分布到Reduce中，是我们的终极目标，也是解决Reduce端倾斜的必然途径。

在此过程中，掌握四点可以帮助我们更好地解决数据倾斜问题：
1. 如果任务长时间卡在99%则基本可以认为是发生了数据倾斜，建议开发者调整参数以实现负载均衡：set hive.groupby.skewindata=true
2. 小表关联大表操作，需要先看能否使用子查询，再看能否使用Mapjoin
3. Join操作注意关联字段不能出现大量的重复值或者空值
4. Count(distinct id ) 去重统计要慎用，尽量通过其他方式替换

最后，作者的想法还是遇到问题再解决问题，不要过度设计，正常情况下按照sql研发规范编写，遇到数据倾斜的场景一般不多，即使遇到数据倾斜问题、且数据量不大，如对结果产出延迟在可接受范围，就不必过度重视。

