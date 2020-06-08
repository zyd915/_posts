---
title: hive 学习之路（五)、Hive的分区表与分桶表
date: 2020-06-08 13:48:50
updated: 2020-06-08 13:48:52
tags: 
    - hive
categories: hive
toc: true
thumbnail: https://static.studytime.xin/article/CROh0hfuG3I.jpg
excerpt: Hive将表划分为分区(partition)表和分桶(bucket)表。分区可以让数据的部分查询变得更快，也就是说，在加载数据的时候可以指定加载某一部分数据，并不是全量的数据。分桶表通常是在原始数据中加入一些额外的结构，这些结构可以用于高效的查询，例如，基于ID的分桶可以使得用户的查询非常的块。
---


Hive将表划分为分区(partition)表和分桶(bucket)表。
分区可以让数据的部分查询变得更快，也就是说，在加载数据的时候可以指定加载某一部分数据，并不是全量的数据。
分桶表通常是在原始数据中加入一些额外的结构，这些结构可以用于高效的查询，例如，基于ID的分桶可以使得用户的查询非常的块。

# Hive分区表

## 什么是分区表？
Hive分区是指按照数据表的某一个字段或多个字段进行统一归类，并存储在在hdfs上的不同文件夹中。当查询过程中指定了分区条件时，只将该分区对应的目录作为Input，从而减少MapReduce的输入数据，提高查询效率。

### 分区表物理存储结构
分区表表在hdfs上作为一个文件夹存在。

```
0: jdbc:hive2://node03:10000> dfs -ls /user/hive/warehouse/myhive1.db/score;
+----------------------------------------------------+--+
|                     DFS Output                     |
+----------------------------------------------------+--+
| Found 4 items                                      |
| drwxr-xr-x   - hadoop supergroup          0 2020-06-07 15:57 /user/hive/warehouse/myhive1.db/score/month=201803 |
| drwxr-xr-x   - hadoop supergroup          0 2020-06-07 15:57 /user/hive/warehouse/myhive1.db/score/month=201804 |
| drwxr-xr-x   - hadoop supergroup          0 2020-06-07 15:57 /user/hive/warehouse/myhive1.db/score/month=201805 |
| drwxr-xr-x   - hadoop supergroup          0 2020-06-07 15:53 /user/hive/warehouse/myhive1.db/score/month=201806 |
```

### 分区表使用场景
实际工作中分区表常常被运用于按照某一维度进行统计分析的场景下，数据被按照某一个日期、年月日等等，将一个大的文件切分成一个个小文件，分而治之，这样处理起来性能就好多了。

### 分区表语法

#### 创建分区表语法

```
hive (myhive)> create table score(s_id string, c_id string, s_score int) partitioned by (month string) row format delimited fields terminated by '\t';
```

#### 创建一个表带多个分区

```
hive (myhive)> create table score2 (s_id string,c_id string, s_score int) partitioned by (year string, month string, day string) row format delimited fields terminated by '\t';
```

#### 加载数据到分区表当中去

```
hive (myhive)>load data local inpath '/kkb/install/hivedatas/score.csv' into table score partition (month='201806');
```

#### 加载数据到多分区表当中去

```
hive (myhive)> load data local inpath '/kkb/install/hivedatas/score.csv' into table score2 partition(year='2018', month='06', day='01');
```

#### 查看分区

```
show  partitions  score;

+---------------+--+
|   partition   |
+---------------+--+
| month=201803  |
| month=201804  |
| month=201805  |
| month=201806  |
+---------------+--+
```

#### 添加一个或多个分区

```
alter table score add partition(month='201805');

alter table score add partition(month='201804') partition(month = '201803');
```

特殊说明：添加分区之后就可以在hdfs文件系统当中看到表下面多了一个文件夹

#### 删除分区
```
alter table score drop partition(month = '201806');
```
特殊说明：同内部表和外部表一致，如果该分区表为外部表，则分区对应的HDFS目录数据不会被删除。

### 分区表练习一

#### 需求描述：

- 现在有一个文件score.csv文件，里面有三个字段，分别是s_id string, c_id string, s_score int
- 字段都是使用 \t进行分割
- 存放在集群的这个目录下/scoredatas/day=20180607，这个文件每天都会生成，存放到对应的日期文件夹下面去
- 文件别人也需要公用，不能移动
- 请创建hive对应的表，并将数据加载到表中，进行数据统计分析，且删除表之后，数据不能删除

#### 需求实现:

- 本地上传数据到hdfs
```
cd /opt/module/hive-1.1.0-cdh5.14.2/data/test
hdfs dfs -mkdir -p /scoredatas/day=20180607
hdfs dfs -put score.csv /scoredatas/day=20180607/
```

- 创建外部分区表，并指定文件数据存放目录

```
create external table score4(s_id string, c_id string, s_score int) partitioned by (day string) row format delimited fields terminated by '\t' location '/scoredatas';
```

- 进行表的修复，说白了就是建立我们表与我们数据文件之间的一个关系映射(),修复成功之后即可看到数据已经全部加载到表当中去了
`msck repair table score4;`

### 静态与动态分区的实现
上文讲到的数据导入场景即为静态分区的使用场景。

### 动态分区
指按照需求实现把数据自动导入到表的不同分区中，不需要手动指定。

#### 创建表

```
--创建普通表
create table t_order(
    order_number string,
    order_price  double, 
    order_time   string
)row format delimited fields terminated by '\t';

--创建目标分区表
create table order_dynamic_partition(
    order_number string,
    order_price  double    
)partitioned BY(order_time string)
row format delimited fields terminated by '\t';
```

#### 准备数据

```
cd /opt/module/hive-1.1.0-cdh5.14.2/data/test
vim order_partition.txt

10001	100	2019-03-02 
10002	200	2019-03-02
10003	300	2019-03-02
10004	400	2019-03-03
10005	500	2019-03-03
10006	600	2019-03-03
10007	700	2019-03-04
10008	800	2019-03-04
10009	900	2019-03-04
```

#### 向普通表t_order加载数据

```
load data local inpath '/opt/module/hive-1.1.0-cdh5.14.2/data/test/order_partition.txt' overwrite into table t_order;
```

#### 动态加载数据到分区表中
```
-- 要想进行动态分区，需要设置参数
-- 开启动态分区功能
hive> set hive.exec.dynamic.partition=true; 
-- 设置hive为非严格模式
hive> set hive.exec.dynamic.partition.mode=nonstrict; 
hive> insert into table order_dynamic_partition partition(order_time) select order_number, order_price, order_time from t_order;
```

#### 查看分区
```
show partitions order_dynamic_partition;

+------------------------+--+
|       partition        |
+------------------------+--+
| order_time=2019-03-02  |
| order_time=2019-03-03  |
| order_time=2019-03-04  |
+------------------------+--+
```

## Hive分桶表

### 什么是分桶表？
Hive分桶是相对分区进行更细粒度的划分。是将整个数据内容按照某列取hash值，对桶的个数取模的方式决定该条记录存放在哪个桶当中；具有相同hash值的数据进入到同一个文件中。

如要安装name属性分为3个桶，就是对name属性值的hash值对3取摸，按照取模结果对数据分桶。如取模结果为0的数据记录存放到一个文件，取模为1的数据存放到一个文件，取模为2的数据存放到一个文件。

### 创建分桶表

#### 在创建分桶表之前要执以下的命令，开启对分桶表的支持以及reduce个数
```
set hive.enforce.bucketing=true;

# 设置与桶相同的reduce个数（默认只有一个reduce）
set mapreduce.job.reduces=4;
```

#### 创建分桶表
```
create table myhive1.user_buckets_demo(id int, name string)
clustered by(id) 
into 4 buckets 
row format delimited fields terminated by '\t';
```

#### 如何向分桶表中导入数据
向分桶表中导入数据，不可以直接加载，需要先导入普通标，再导入分桶表中，这种和动态分区类似。

```
# 创建普通表
create table user_demo(id int, name string)
row format delimited fields terminated by '\t';

# 准备数据文件 buckets.txt
cd /opt/module/hive-1.1.0-cdh5.14.2/data/test
vim user_bucket.txt

1	anzhulababy1
2	anzhulababy2
3	anzhulababy3
4	anzhulababy4
5	anzhulababy5
6	anzhulababy6
7	anzhulababy7
8	anzhulababy8
9	anzhulababy9
10	anzhulababy10

# 向普通标中导入数据
load data local inpath '/opt/module/hive-1.1.0-cdh5.14.2/data/test/user_bucket.txt'  overwrite into table user_demo; 

# 查看数据
select * from user_demo;

+---------------+-----------------+--+
| user_demo.id  | user_demo.name  |
+---------------+-----------------+--+
| 1             | anzhulababy1    |
| 2             | anzhulababy2    |
| 3             | anzhulababy3    |
| 4             | anzhulababy4    |
| 5             | anzhulababy5    |
| 6             | anzhulababy6    |
| 7             | anzhulababy7    |
| 8             | anzhulababy8    |
| 9             | anzhulababy9    |
| 10            | anzhulababy10   |
+---------------+-----------------+--+

# 加载数据到桶表user_buckets_demo中
insert into table user_buckets_demo select * from user_demo;
```

### 分桶表物理存储结构
分桶表表在hdfs上作为一个文件存在。
```
0: jdbc:hive2://node03:10000> dfs -ls /user/hive/warehouse/myhive1.db/user_buckets_demo;
+----------------------------------------------------+--+
|                     DFS Output                     |
+----------------------------------------------------+--+
| Found 4 items                                      |
| -rwxr-xr-x   3 hadoop supergroup         30 2020-06-08 13:30 /user/hive/warehouse/myhive1.db/user_buckets_demo/000000_0 |
| -rwxr-xr-x   3 hadoop supergroup         45 2020-06-08 13:30 /user/hive/warehouse/myhive1.db/user_buckets_demo/000001_0 |
| -rwxr-xr-x   3 hadoop supergroup         47 2020-06-08 13:30 /user/hive/warehouse/myhive1.db/user_buckets_demo/000002_0 |
| -rwxr-xr-x   3 hadoop supergroup         30 2020-06-08 13:30 /user/hive/warehouse/myhive1.db/user_buckets_demo/000003_0 |
+----------------------------------------------------+--+
```

### 分通表使用场景
- 取样sampling更高效。没有分桶的话需要扫描整个数据集。
- 提升某些查询操作效率，例如map side join
 
### 如何抽样查询桶表的数据
tablesample抽样语句语法：tablesample(bucket  x  out  of  y)
- x表示从第几个桶开始取数据
- y与进行采样的桶数的个数、每个采样桶的采样比例有关

```
select * from user_buckets_demo tablesample(bucket 1 out of 2);
-- 需要采样的总桶数=4/2=2个
-- 先从第1个桶中取出数据
-- 1+2=3，再从第3个桶中取出数据
```

特殊说明：分区表可以与分桶表一起使用