---
title: hive 常见报错以及解决方案
permalink: hive-run-questions
date: 2020-05-23 23:48:01
updated: 2020-05-23 23:48:02
tags:
    - 大数据
    - hive
categories: [大数据,hive]
toc: true
excerpt: 整理汇总hive使用过程中遇到的问题以及解决办法。
---

整理汇总hive使用过程中遇到的问题以及解决办法。

### 问题一： Hive 创建表时报错
`Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:An exception was thrown while adding/validating class(es) : Column length too big for column 'PARAM_VALUE' (max = 21845); use BLOB or TEXT instead
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Column length too big for column 'PARAM_VALUE' (max = 21845);
`

![](https://static.studytime.xin/article/20200523235044.png)


解决办法：
```
mysql> show variables like "char%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)

mysql>
mysql> use hive;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
mysql> alter database hive character set latin1;
Query OK, 1 row affected (0.00 sec)

mysql>
mysql> show variables like "char%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

hive 创建表成功：
```
0: jdbc:hive2://node03:10000> create table stu(id int,name string);
No rows affected (0.996 seconds)
```


### 问题二： `找不到或无法加载主类 org.apache.hadoop.mapreduce.v2.app.MRAppMaster`

```
[hadoop@node01 ~]$ hadoop classpath
/opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common/lib/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/hdfs:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/hdfs/lib/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/hdfs/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/yarn/lib/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/yarn/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/mapreduce/lib/*:/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/mapreduce/*:/opt/module/hadoop-2.6.0-cdh5.14.2/contrib/capacity-scheduler/*.jar
```

修改 yarn-site.xml

```
vim /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop/yarn-site.xml

<property>
        <name>yarn.application.classpath</name>
        <value>hadoop classpath返回信息</value>
</property>
```

在所有的Master和Slave节点进行如上设置，设置完毕后重启Hadoop集群，重新运行刚才的MapReduce程序，成功运行。
