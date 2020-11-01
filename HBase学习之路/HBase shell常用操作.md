---
title: Hbase学习之路（四）HBase常用shell操作
permalink: hbase-shell-operate
date: 2020-06-12 19:33:22
updated: 2020-06-12 19:33:23
tags: 
    - bigdata
    - hbase
categories: [大数据,hbase]
toc: true
excerpt: 本内容整理了部分常用命令、以及实验案例。
---

本内容整理了部分常用命令、以及实验案例。Hbase有大量命令，此处只列举出部分。更多使用案例和方法，`list`命令已经有很好的支持。


## 目录

### 进入Hbase shell

进入命令
```
➜  hbase-1.2.4 bin/hbase shell
```

help 帮助命令

```
hbase(main):001:0> help
HBase Shell, version 1.2.4, r67592f3d062743907f8c5ae00dbbe1ae4f69e5af, Tue Oct 25 18:10:20 CDT 2016
Type 'help "COMMAND"', (e.g. 'help "get"' -- the quotes are necessary) for help on a specific command.
Commands are grouped. Type 'help "COMMAND_GROUP"', (e.g. 'help "general"') for help on a command group.
```

get 查看单独的某个命令的帮助

```
hbase(main):001:0> help 'get'
Get row or cell contents; pass table name, row, and optionally
a dictionary of column(s), timestamp, timerange and versions. Examples:

  hbase> get 'ns1:t1', 'r1'
  hbase> get 't1', 'r1'
  hbase> get 't1', 'r1', {TIMERANGE => [ts1, ts2]}
  hbase> get 't1', 'r1', {COLUMN => 'c1'}
  hbase> get 't1', 'r1', {COLUMN => ['c1', 'c2', 'c3']}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
  hbase> get 't1', 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
  hbase> get 't1', 'r1', 'c1'
  hbase> get 't1', 'r1', 'c1', 'c2'
  hbase> get 't1', 'r1', ['c1', 'c2']
  hbase> get 't1', 'r1', {COLUMN => 'c1', ATTRIBUTES => {'mykey'=>'myvalue'}}
  hbase> get 't1', 'r1', {COLUMN => 'c1', AUTHORIZATIONS => ['PRIVATE','SECRET']}
  hbase> get 't1', 'r1', {CONSISTENCY => 'TIMELINE'}
  hbase> get 't1', 'r1', {CONSISTENCY => 'TIMELINE', REGION_REPLICA_ID => 1}

```

### 一般命令

status 查看状态

```
hbase(main):002:0> status
1 active master, 0 backup masters, 1 servers, 0 dead, 5.0000 average load
```

version 查看版本

```
hbase(main):003:0> version
1.2.4, r67592f3d062743907f8c5ae00dbbe1ae4f69e5af, Tue Oct 25 18:10:20 CDT 2016
```

whoami 显示当前的Hbase用户

```
hbase(main):004:0> whoami
```

### DDL（数据定义语言Data Definition Language）命令

作用在Hbase表（元信息）上的命令，主要包括下列命令：

create 创建表 `create 't1', 'f1', 'f2', 'f3'`

```
# 创建blog表，该表有两个列族：‘article’和‘author’
hbase(main):004:0> create 'blog','article','author'
0 row(s) in 1.3010 seconds

=> Hbase::Table - blog
```

list 列出所有表 

> list ‘abc.*’ #显示abc开头的表


```
hbase(main):005:0> list
```

desc 获得表的描述
```
hbase(main):009:0> describe 'employee'
```

alter 增加列簇、删除列簇

```
# 更改或添加t1表中的f1列簇,以保持最多5个单元格的VERSIONS
alter't1'，NAME =>'f1'，VERSIONS => 5

# 操作多个列簇
alter't1'，'f1'，{NAME =>'f2'，IN_MEMORY => true}，{NAME =>'f3'，VERSIONS => 5}

# 删除列簇
alter 't1',NAME=>'f1',METHOD=>'delete'

# 更改范围相关属性MAX_FILESIZE，READONLY，MEMSTORE_FLUSHSIZE，DEFERRED_LOG_FLUSH；可以考虑将改属性放置最后
```


删除表

```

hbase(main):002:0> disable 'blog'
0 row(s) in 2.4030 seconds

hbase(main):003:0> drop 'blog'
0 row(s) in 1.2920 seconds

hbase(main):004:0>
```

exists 查询表是否存在

```
hbase(main):012:0> exists 'employee'
Table employee does exist
0 row(s) in 0.0220 seconds
```

is_enabled 验证表是否enabled

```
hbase(main):015:0> is_enabled 'employee'
true
0 row(s) in 0.0140 seconds
```

更改表名

```
需要开启快照功能，在hbase-site.xml文件中添加如下配置项：
<property>
<name>hbase.snapshot.enabled</name>
<value>true</value>
</property>

//命令
hbase shell> disable 'tableName'
hbase shell> snapshot 'tableName', 'tableSnapshot'
hbase shell> clone_snapshot 'tableSnapshot', 'newTableName'
hbase shell> delete_snapshot 'tableSnapshot'
hbase shell> drop 'tableName'
```

disable_all 下线匹配到的数据表

```
disable_all 't.*'
```

is_disabled 表是否可用

```
hbase(main):017:0> is_disabled  'employee'
```


### DML（data manipulation language）操作
作用在数据上的命令，主要包括：

put 插入 `put 't1', 'r1', 'c1', 'value'`

```
put 'blog','blog1','article:title','mapreduce'
put 'blog','blog2','article:title','hadoop'
```

get 获取一条数据 `t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}`

```
hbase(main):043:0>  get 'blog','blog2',{COLUMN=>['author:name','author:age']}
```

put 更新一条记录

```
 put 'blog','blog1','author:age','40'
```

delete 删除记录

```
delete 'blog','blog3','article:tag'
```

scan 全表扫描

1. 限制展示某些列

```
scan ‘t1’,{COLUMNS=>’cf’}
scan ‘t1’,{COLUMNS=>’cf:f1’}
scan ‘t1’,{COLUMNS=>[‘cf1’,’cf2’]}
```

2. 限制查找行数

```
scan ‘t1’,{COLUMNS=>[‘cf1’,’cf2’],LIMIT=>2}
```

3. 限制时间范围

```
scan ‘t1’,{TIMERANGE=>[1448045892646,1448045892647]}
```

4. PrefixFilter:rowKey前缀过滤

```
scan 'blog’,{FILTER=>PrefixFilter(‘blog’)}
```

5. QualifierFilter:列过滤器,QualifierFilter对列的名称进行过滤，而不是列的值。

```
scan ‘blog’,{FILTER=>”PrefixFilter(‘t’) AND QualifierFilter(>=,’binary:b’)”}
```

6. TimestampsFilter:时间戳过滤器

```
scan ‘qy’,{FILTER=>”TimestampsFilter(1448069941270,1548069941230)” }
```

7. count 返回Hbase表中总的记录数

```
INTERVAL: 每隔多少行显示一次count，默认是1000
CACHE:每次去取的缓存区大小，默认是10，调整该参数可提高查询速度
count 'blog',INTERVAL=>1,CACHE => 1
```

8. truncate 清空表

```
HBase是先将表disable，再drop the table，最后creating table。
truncate 'table_name'
```


### 练习


![](https://static.studytime.xin/image/articles/spring-boot20190821013459.png)

1. 建表：表名blog，有两个列族：‘article’和‘author’

```
create 'blog','article','author'
```
2. 插⼊数据到表blog中，数据如上图

```
put 'blog','blog1','article:title','mapreduce'
put 'blog','blog2','article:title','hadoop'
put 'blog','blog3','article:title','hdfs'
```
3. 读出rowkey为“blog2”的author的name和age

```
get 'blog','blog2',{COLUMN=>['author:name','author:age']}
```

4. 读出所有article的title

```
scan 'blog',{COLUMNS=>['article:title']}
```

5. 更新“blog1”作者的age为40

```
 put 'blog','blog1','author:age','40'
```

6. 读出rowkey为“blog1”中author的name和age

```
get 'blog','blog1',{COLUMN=>['author:name','author:age']}
```

7. 删除rowkey为“blog3”中article的tag

```
delete 'blog','blog3','article:tag'
```

8. 读出所有article的title和tag，同时读出所有author的name

```
scan 'blog',{COLUMNS=>['article:title','article:tag','author:name']}
```

### 参考文档

文档一: [https://learnhbase.net/2013/03/02/hbase-shell-commands/](https://learnhbase.net/2013/03/02/hbase-shell-commands/)

文档二: [https://www.maiyewang.com/?p=6607](https://www.maiyewang.com/?p=6607)
