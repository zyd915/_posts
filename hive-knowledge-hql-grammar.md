---
title: hive 学习之路（四）HQL的基本语法
date: 2020-05-04 17:36:45
updated: 2020-05-04 17:39:43
tags:
    - 大数据
    - hive
categories: hive
toc: true
priority: 0
excerpt: HQL 是建立在 Hive 上的类sql语言，虽然和mysql等sql语言有很多一致的地方，但还是有所不同的。本文整理汇总了 HQL的 相关 DDL 操作。
---

### 数据库相关的 DDL

1、创建库

```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
　　[COMMENT database_comment]　　　　　　 //关于数据块的描述
　　[LOCATION hdfs_path]　　　　　　　　　　//指定数据库在HDFS上的存储位置
　　[WITH DBPROPERTIES (property_name=property_value, ...)];　　　　//指定数据块属性
```

1.1、创建普通的数据库
```
create database db1;
```
1.2、创建库的时候检查存与否
```
create database IF NOT EXISTS db1;
```
1.3、创建库的时候带注释
```
create database IF NOT EXISTS db1 COMMENT '测试库一';
```
1.4、创建带属性的库
```
create database db1 with deproperties('creator'='hadoop','date'='2019-09-16')
```

2、查看库

2.1、查看所有数据库
```
show databases;
```
2.2、显示数据库的详细属性信息
```
desc database extended t3;
```
2.3、查看当前正在使用的数据库
```
select current_database();
```
2.4、查看创建库的详细语句
```
show create database db1;
```

3、删除库
语法：
```
drop database dbname;
drop database if exists dbname;
```
默认情况下，hive 不允许删除包含表的数据库，有两种解决办法：
- 手动删除库下所有表，然后删除库
- 使用 cascade 关键字

3.1、删除不包含表的库
```
drop database db1;
```

3.2、删除含有表的数据库

```
drop database if exists db1 cascade;
```

4、切换库
```
 use db1;
```

### 数据表相关的 DDL

1、 创建表

语法：
```
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];

```

下面按照数据表语法定义，依次解析各个关键的的含义：

数据类型：Hive 提供了丰富的数据类型，它不仅提供了类似关系型数据库的基本数据类型，也提供了对高级数据类型的支持。

分区表与分桶表：为了加速数据处理，数据表可以进一步划分为更小的存储单位，即分区或分桶。

分区表：数据表可以按照一个或多个字段进一步划分为一个或多个数据分区。`PARTITIONED BY (String city,String sex)`。不同分区的数据将存放在不同的目录内，当一个查询语句只需要用到里面若干个分区，其他分区则可以直接跳过扫描，大大减少了磁盘扫描。

分桶表：数据表或数据分区可进一步按照某个字段分层若干个桶。`clustered by (userid) into 21 buckets`。将数据表按照userid字段分成21个桶。

行格式（ROW FORMAT）：该配置用于指定每行的数据格式，仅对行式存储格式有意义。
创建数据表时，delimited 和 SerDe 两种配置最多设置一个，可以不设置，几个关键字含义如下：
-  FIELDS TERMINATED BY char：每行中的字段分隔符 char
-  COLLECTION ITEMS TERMINATED BY char：map、struct或者array 中的每个元素分隔符 char
-  MAP KEYS TERMINATED BY char：map 中的key、value 分隔符 char
-  LINES TERMINATED BY char：行分隔符 char

SerDe 关键字允许用户通过定制序列化和反序列化七规定数据存储格式。

Json格式例如：
```
row format serde 'org.apache.hive.hctalog.data,JsonSerDe' STORED  AS TEXTFILE;
```


STORED AS：指定数据存储格式。Hive支持多种数据存储格式：
- TEXTFILE：文本文件，这是默认文本文件格式。用户可通过 hive:default:fileformat 修改默认值，可选参数有 testfile、SEQUENCEFILE、RCFILE、ORC
- SEQUENCEFILE
- RCFILE
- ORC
- PARQUET
- AVRO
- 'org.apache.hadoop.hive.hbase.HBaseStorageHandler’ WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,a:b,a:c,d:e"); 行存储与列存储

数据表的类别：Hive 数据表可以分为三类，分别为  临时表、外部表和受管理表，其区别如下：
临时表：仅对当前 session可见，一旦session退出 ，则该数据表会自动退出，使用TEMPORARY关键字
外部表：外部表的数据存储路径是用户定义的非Hive默认存放位置，外部表被删除后，其对应的数据不会被清除，使用EXTERNAL关键字，与LOCATION配合使用
收管理表：默认数据表的类型，受Hive管理，与元数据声生命周期是一致的

同时外部表被删除后，只是删除metastore数据，原始数据仍然不变。


### 常用案例

一、创建默认的内部表

```
create table studyent(id int,name string,sex string,age int,department string) row format delemited fields terminated by ",";
```

```
0: jdbc:hive2://localhost:10000/default> desc student;
+-------------+------------+----------+--+
|  col_name   | data_type  | comment  |
+-------------+------------+----------+--+
| id          | int        |          |
| name        | string     |          |
| sex         | string     |          |
| age         | int        |          |
| department  | string     |          |
+-------------+------------+----------+--+
5 rows selected (0.098 seconds)
```

二、创建外部表
```
create external table student_ext
(id int, name string, sex string, age int,department string) row format delimited fields terminated by "," location "/tmp/student";
```

三、创建分区表
```
create external table student_ptn(id int, name string, sex string, age int,department string) partitioned by (city string)  row format delimited fields terminated by "," location "/hive/student_ptn";

创建分区
alter table student_ptn add partition(city="beijing") location '/user/hive/warehouse/c02_clickstat_fatdt1/part20131202';

删除分区
ALTER TABLE student_ptn DROP PARTITION (city='beijing');
```

四、使用CTAS创建表

```
 create table student_ctas as select * from student where id < 95012;
```

五、复制表结构
```
create table student_copy like student;
```




