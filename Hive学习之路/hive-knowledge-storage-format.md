---
title: hive 学习之路（三）hive存储格式
permalink: hive-knowledge-storage-format
date: 2020-05-04 17:34:21
updated: 2020-05-04 17:35:43
tags:
    - 大数据
    - hive
categories: hive
toc: true
excerpt: Hive会为每个创建的数据库在HDFS上创建一个目录，该数据库的表会以子目录形式存储，表中的数据会以表目录下的文件形式存储。对于default数据库，默认的缺省数据库没有自己的目录，default数据库下的表默认存放在/user/hive/warehouse目录下。
---

Hive会为每个创建的数据库在HDFS上创建一个目录，该数据库的表会以子目录形式存储，表中的数据会以表目录下的文件形式存储。对于default数据库，默认的缺省数据库没有自己的目录，default数据库下的表默认存放在/user/hive/warehouse目录下。

### 常见数据格式
- textfile 
- SequenceFile 
- RCFile 
- ORCFile 
- Parquet 

### 数据格式
当数据存储在文本文件中，必须按照一定格式区别行和列，并且在Hive中指明这些区分符。Hive默认使用了几个平时很少出现的字符，这些字符一般不会作为内容出现在记录中。

Hive默认的行和列分隔符如下表所示：
| 分隔符 |描述  |
| --- | --- |
| \n | 对于文本文件来说，每行是一条记录，所以\n 来分割记录 |
| ^A | 分割字段，也可以用\001 来表示 |
| ^B | ARRAY或STRUCT中元素分隔符，或MAP中 key与value分隔符，通常写成“\002” |
| ^C | MAP中key/value对间的分隔符，通常写成 “\003” |


常用实例：
```
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\001' COLLECTION ITEMS TERMINATED BY '\002' MAP KEYS TERMINATED BY '\003' LINES TERMINATED BY '\n'
```

```
John Doe^A100000.0^AMary Smith^BTodd Jones^AFederal Taxes^C.2^BState Taxes^C.05^BInsurance^C.1^A1 Michigan Ave.^BChicago^BIL^B60600

Mary Smith^A80000.0^ABill King^AFederal Taxes^C.2^BState Taxes^C.05^BInsurance^C.1^A100 Ontario St.^BChicago^BIL^B60601

Todd Jones^A70000.0^A^AFederal Taxes^C.15^BState Taxes^C.03^BInsurance^C.1^A200 Chicago Ave.^BOak Park^BIL^B60700

Bill King^A60000.0^A^AFederal Taxes^C.15^BState Taxes^C.03^BInsurance^C.1^A300 Obscure Dr.^BObscuria^BIL^B60100

```