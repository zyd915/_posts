---
title: 精心整理的Hive数据导入导出的几种方式
permalink: hive-knowledge-export-import
date: 2021-08-04 18:47:13
updated: 2021-08-04 18:47:43
tags:
- 大数据
- hive
categories: [大数据,hive]
toc: true
thumbnail: https://static.studytime.xin//studytime/image/articles/kJDdDU.jpg
keywords: [Hive导入导出, Hive导入方式, Hive怎么导出数据, Hive导出数据, Hive怎么导出数据]
excerpt: 作为数据仓库的Hive，存储着海量用户数据,在平常的Hive使用过程中，难免对遇到将外部数据导入到Hive或将Hive中的数据导出的情况。
---

作为数据仓库的Hive，存储着海量用户数据,在平常的Hive使用过程中，难免对遇到将外部数据导入到Hive或将Hive中的数据导出的情况。

## Hive数据导入方式（Hive怎么导入数据）
Hive数据导入方式主要有直接向表中插入数据、通过load加载数据、通过查询加载数据、查询语句中创建表并加载数据等四种。

### 直接向表中插入数据

#### 语法格式
```
INSERT INTO TABLE tablename [PARTITION (partcol1[=val1], partcol2[=val2] ...)] VALUES values_row [, values_row ...]
```

#### 使用场景
强烈不推荐使用，插入速度极慢，一般用于插入少量测试数据。

#### 使用案例
```sql
-- 创建学生信息分桶表
CREATE TABLE students (
    name VARCHAR(64), 
    age INT, 
    gpa DECIMAL(3, 2)
) CLUSTERED BY (age) INTO 2 BUCKETS STORED AS ORC;
 
-- 向学生信息表中插入学生信息
INSERT INTO TABLE students
  VALUES ('fred flintstone', 35, 1.28), ('barney rubble', 32, 2.32);
 
 
-- 创建浏览信息表，按照stat_date分区，用户userid分桶
CREATE TABLE pageviews (
    userid VARCHAR(64), 
    link STRING, 
    came_from STRING
) PARTITIONED BY (stat_date STRING) CLUSTERED BY (userid) INTO 256 BUCKETS STORED AS ORC;
 
-- 向浏览信息表表中插入分区为2021-08-04的数据
INSERT INTO TABLE pageviews PARTITION (stat_date = '2021-08-04')
  VALUES ('jsmith', 'mail.com', 'sports.com'), ('jdoe', 'mail.com', null);

-- 直接向浏览信息表表中插入分区为2021-08-04的数据
INSERT INTO TABLE pageviews
  VALUES ('tjohnson', 'sports.com', 'finance.com', '2021-08-04'), ('tlee', 'finance.com', null, '2021-08-04');
```

### 通过load加载数据（必须掌握）
Hive 在将数据加载到表中时不做任何转换，加载操作目在Hive3.0 前是纯复制/移动操作，将数据文件移动到Hive对应表的位置，Hive 3.0 之后则是内部将加载重写为 INSERT AS SELECT。

#### 语法格式
```sql
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [INPUTFORMAT 'inputformat' SERDE 'serde'] (3.0 or later)
```

#### 语法参数说明
filepath 文件路径，可以是：

- 相对路径，例如 project/data1
- 绝对路径，例如 /user/hive/project/data1
- 加载方式及路径的完整UR，,例如 hdfs://namenode:9000/user/hive/project/data1，file:///user/hive/project/data1

加载到的目标可以是表或表的分区；如果使用 OVERWRITE 关键字，则目标表（或分区）的内容将被删除并替换为filepath引用的文件；否则filepath引用的文件将被添加到表中。
Hive 3.0 后支持一些特殊的加载操作，inputformat 可以是任何 Hive 输入格式，例如文本、ORC 等，serde可以是关联的 Hive SERDE，无论inputformat和SERDE区分大小写。

#### 使用场景
日志数据、用户行为数据等日志文本类数据，可以上传到hdfs上，然后通过load命名加载到数仓ods层表中；一些填报类的数据或者非业务类数据，以csv形式导入数仓中。


#### 使用案例

```sql

-- 创建订单load表
CREATE TABLE order_info_load (
  order_id STRING COMMENT '订单id',
  order_status STRING COMMENT '订单状态',
  create_time STRING COMMENT '创建时间',
  update_time STRING COMMENT '最后更新时间'
) PARTITIONED BY (stat_date string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';


-- 本地创建order-info-0322.txt文本文件
1,待支付,2021-03-22 12:20:11,2021-03-22 12:20:11
2,待支付,2021-03-22 14:20:11,2021-03-22 14:20:11
3,待支付,2021-03-22 16:20:11,2021-03-22 16:20:11

-- 本地创建order-info-0323.txt文本文件
1,待支付,2021-03-22 12:20:11,2021-03-22 12:20:11
2,已支付,2021-03-23 14:20:11,2021-03-23 14:21:11
3,已支付,2021-03-23 16:20:11,2021-03-23 16:21:11
3,待支付,2021-03-23 12:20:11,2021-03-23 12:20:11
3,待支付,2021-03-23 14:20:11,2021-03-23 14:20:11

-- 将本地创建的 order-info-0323.txt 文本文件上传到hdfs上
sudo -u hive hdfs dfs -put /data/order-info-0323.txt /tmp/


-- 加载服务器本地order-info-0322.txt文本数据到order_info_load订单表的20210322分区中
load data local inpath '/data/order-info-0322.txt' overwrite into table order_info_load partition(stat_date='20210322');

-- 验证load加载本地数据结果集
select * from order_info_load where stat_date = '20210322';

+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
| order_info_load.order_id  | order_info_load.order_status  | order_info_load.create_time  | order_info_load.update_time  | order_info_load.stat_date  |
+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
| 1                         | 待支付                           | 2021-03-22 12:20:11          | 2021-03-22 12:20:11          | 20210322                   |
| 2                         | 待支付                           | 2021-03-22 14:20:11          | 2021-03-22 14:20:11          | 20210322                   |
| 3                         | 待支付                           | 2021-03-22 16:20:11          | 2021-03-22 16:20:11          | 20210322                   |
+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
3 rows selected (0.332 seconds)

-- 加载hdfs上的order-info-0323.txt文本数据到order_info_load订单表的20210323分区中
load data inpath '/tmp/order-info-0323.txt' overwrite into table order_info_load partition(stat_date='20210323');

-- 验证load加载hdfs数据结果集
select * from order_info_load where stat_date = '20210323';
+----------------+----------------+----------------+----------------+----------------+
| order_id  |order_status  | create_time  | update_time  | stat_date  |
+----------------+----------------+----------------+----------------+----------------+
| 1         | 待支付       | 2021-03-22 12:20:11          | 2021-03-22 12:20:11          | 20210323                   |
| 2         | 已支付       | 2021-03-23 14:20:11          | 2021-03-23 14:21:11          | 20210323                   |
| 3         | 已支付       | 2021-03-23 16:20:11          | 2021-03-23 16:21:11          | 20210323                   |
| 3         | 待支付       | 2021-03-23 12:20:11          | 2021-03-23 12:20:11          | 20210323                   |
| 3         | 待支付       | 2021-03-23 14:20:11          | 2021-03-23 14:20:11          | 20210323                   |
+----------------+----------------+----------------+----------------+----------------+
5 rows selected (0.737 seconds)
 
-- 重新加载服务器本地order-info-0322.txt文本数据到order_info_load订单表的20210322分区中，为什么重新上传呢？因为上次本文件已经被加载到hive中，对应的数据文件已经被移动到了hive表中的hdfs文件目录
sudo -u hive hdfs dfs -put /data/order-info-0323.txt /tmp/


-- 加载hdfs上的order-info-0323.txt文本数据到order_info_load订单表的20210324分区中
load data inpath 'hdfs://node01:8020/tmp/order-info-0323.txt' overwrite into table order_info_load partition(stat_date='20210324');

-- 验证load加载hdfs数据结果集
select * from order_info_load where stat_date = '20210324';
+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
| order_info_load.order_id  | order_info_load.order_status  | order_info_load.create_time  | order_info_load.update_time  | order_info_load.stat_date|
+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
| 1                         | 待支付                           | 2021-03-22 12:20:11          | 2021-03-22 12:20:11          | 20210324       |
| 2                         | 已支付                           | 2021-03-23 14:20:11          | 2021-03-23 14:21:11          | 20210324       |
| 3                         | 已支付                           | 2021-03-23 16:20:11          | 2021-03-23 16:21:11          | 20210324       |
| 3                         | 待支付                           | 2021-03-23 12:20:11          | 2021-03-23 12:20:11          | 20210324       |
| 3                         | 待支付                           | 2021-03-23 14:20:11          | 2021-03-23 14:20:11          | 20210324       |
+---------------------------+-------------------------------+------------------------------+------------------------------+----------------------------+
5 rows selected (0.169 seconds)
```

### 通过查询加载数据（必须掌握）

Hive也可以使用 insert 子句将查询结果插入到表中。

#### 语法格式
```
-- 标准格式：
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement;
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement;

-- 多个插入格式
FROM from_statement
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1
[INSERT OVERWRITE TABLE tablename2 [PARTITION ... [IF NOT EXISTS]] select_statement2]
[INSERT INTO TABLE tablename2 [PARTITION ...] select_statement2] ...;
```

#### 使用场景
使用Hive清洗数据时，将ods数据合并清洗到dwd层，或者增量入仓方式的增量数据合并场景等。


#### 使用案例

```sql
-- 标准格式演示
create table order_info_select_load like order_info_load;
insert overwrite table order_info_select_load partition(stat_date = '20210323') select s_id,c_id,s_score from order_info_load where stat_date = '20210323';

-- 多个插入格式
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country)
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip, pvs.cnt
```

### 查询语句中创建表并加载数据（as select）
查询语句中创建表并加载数据（as select），是将查询的结果保存到一张表当中去，先创建表再导入数据。

```sql
create table order_info_as_select_load as select * from order_info_load;
```

### import 导入 hive表数据（内部表操作）

```sql
-- 创建表
create table order_info_import_load like order_info_load;

-- 重新加载服务器本地order-info-0322.txt文本数据到order_info_load订单表的20210322分区中，为什么重新上传呢？因为上次本文件已经被加载到hive中，对应的数据文件已经被移动到了hive表中的hdfs文件目录
sudo -u hive hdfs dfs -put /data/order-info-0323.txt /tmp/

-- 导入hdfs数据
import table order_info_import_load from '/data/order-info-0323.txt';
```


## Hive数据导出（Hive怎么导出数据）
Hive数据导出方式，和导入方式一样，针对不同场景也是有很多方式，主要方式有insert 导出、Hive Shell 命令导出、export命令导出等两种。

### Hive insert导出

```sql
-- 将查询的结果导出到本地
insert overwrite local directory '/data/order_info_load_out1' select * from order_info_load where stat_date = '20210324';

-- 将查询的结果按照指定格式化后导出到本地
insert overwrite local directory '/data/order_info_load_out2' row format delimited fields terminated by ',' select * from order_info_load where stat_date = '20210324';

-- 将查询的结果导出到HDFS上(没有local)
insert overwrite directory '/tmp/order_info_load_out3' row format delimited fields terminated by  ','  select * from order_info_load where stat_date = '20210324';
```

### Hive Shell命令导出

```
-- 语法格式
hive -e "sql语句" >   file
hive -f  sql文件   >    file

-- 使用案例
hive -e 'sselect * from order_info_load where stat_date = '20210324';' > /data/order_info_load_out.txt
```

### export导出到HDFS上
```sql
export table  order_info_load to '/tmp/order_info_load_out4';
```