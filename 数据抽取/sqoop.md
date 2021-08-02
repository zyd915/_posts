---
title: sqoop数据抽取同步工具
permalink: sqoop-detail
date: 2020-11-15 23:48:01
updated: 2020-11-15 23:48:01
tags:
    - 大数据
    - sqoop
categories: [大数据,sqoop]
toc: true
excerpt: Sqoop是apache旗下的一款 ”Hadoop和关系数据库之间传输数据”的工具。 
---

## 概述
Sqoop是apache旗下的一款 ”Hadoop和关系数据库之间传输数据”的工具。
导入数据：将MySQL，Oracle导入数据到Hadoop的HDFS、HIVE、HBASE等数据存储系统
导出数据：从Hadoop的文件系统中导出数据到关系数据库

![](https://static.studytime.xin/article/20201117132229.png)

## Sqoop的工作机制
- 将导入和导出的命令翻译成mapreduce程序实现
- 在翻译出的mapreduce中主要是对inputformat和outputformat进行定制

## Sqoop1与Sqoop2架构对比
- sqoop在发展中的过程中演进出来了两种不同的架构.[架构演变史](https://blogs.apache.org/sqoop/entry/apache_sqoop_highlights_of_sqoop#comment-1561314193000)

### Sqoop1架构

![](https://static.studytime.xin/article/20201117132311.jpg)

> 版本号为1.4.x为sqoop1 
在架构上：sqoop1使用sqoop客户端直接提交的方式
访问方式：CLI控制台方式进行访问
安全性：命令或脚本中指定用户数据库名及密码

### Sqoop2架构

![](https://static.studytime.xin/article/20201117132359.jpg)

> 版本号为1.99x为sqoop2 
在架构上：sqoop2引入了sqoop server，对connector实现了集中的管理
访问方式：REST API、 JAVA API、 WEB UI以及CLI控制台方式进行访问

### sqoop1与sqoop2比较
![](https://static.studytime.xin/article/20201117132429.png)

## Sqoop安装部署
Sqoop安装很简单，解压好进行简单的修改就可以使用
### 第一步：下载安装包

http://archive.cloudera.com/cdh5/cdh/5/sqoop-1.4.6-cdh5.14.2.tar.gz

#### 第二步：上传并解压
```powershell
cd /opt/software/
tar -zxf sqoop-1.4.6-cdh5.14.2.tar.gz -C /opt/module/
```
### 第三步：修改配置文件
```shell
cd /opt/module/sqoop-1.4.6-cdh5.14.2/conf
mv sqoop-env-template.sh sqoop-env.sh
vim sqoop-env.sh
```

 ```shell
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.6.0-cdh5.14.2

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.6.0-cdh5.14.2

#set the path to where bin/hbase is available
export HBASE_HOME=/opt/module/hbase-1.2.0-cdh5.14.2

#Set the path to where bin/hive is available
export HIVE_HOME=/opt/module//hive-1.1.0-cdh5.14.2
```

### 第四步：添加两个必要的jar包
sqoop需要两个额外依赖的jar包，将课件资料当中两个jar包添加到sqoop的lib目录下
![](https://static.studytime.xin/article/20201117132540.png)

### 第五步：配置sqoop的环境变量

```shell
sudo vim /etc/profile

export SQOOP_HOME=/opt/module/sqoop-1.4.6-cdh5.14.2
export PATH=:$SQOOP_HOME/bin:$PATH

source /etc/profile
```


## Sqoop的数据导入

### 列举出所有的数据库

#### 命令行查看帮助
```shell
bin/sqoop list-databases --help
```

#### 列出node01主机所有的数据库
```shell
bin/sqoop list-databases --connect jdbc:mysql://node01:3306/ --username root --password '!Qaz123456'
```

#### 查看某一个数据库下面的所有数据表
```
bin/sqoop list-tables --connect jdbc:mysql://node01:3306/hive --username root --password '!Qaz123456'
```

###  准备表数据
在mysql中有一个库userdb中三个表：emp, emp_add和emp_conn

#### 表emp

| **id** | **name** | **deg**      | **salary** | **dept** |
| ------ | -------- | ------------ | ---------- | -------- |
| 1201   | gopal    | manager      | 50,000     | TP       |
| 1202   | manisha  | Proof reader | 50,000     | TP       |
| 1203   | khalil   | php dev      | 30,000     | AC       |
| 1204   | prasanth | php dev      | 30,000     | AC       |
| 1205   | kranthi  | admin        | 20,000     | TP       |

#### 表emp_add

| **id** | **hno** | **street** | **city** |
| ------ | ------- | ---------- | -------- |
| 1201   | 288A    | vgiri      | jublee   |
| 1202   | 108I    | aoc        | sec-bad  |
| 1203   | 144Z    | pgutta     | hyd      |
| 1204   | 78B     | old city   | sec-bad  |
| 1205   | 720X    | hitec      | sec-bad  |

#### 表emp_conn

| **id** | **phno** | **email**       |
| ------ | -------- | --------------- |
| 1201   | 2356742  | gopal@tp.com    |
| 1202   | 1661663  | manisha@tp.com  |
| 1203   | 8887776  | khalil@ac.com   |
| 1204   | 9988774  | prasanth@ac.com |
| 1205   | 1231231  | kranthi@tp.com  |

- 建表语句如下：

```
CREATE DATABASE /*!32312 IF NOT EXISTS*/`userdb` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `userdb`;

DROP TABLE IF EXISTS `emp`;

CREATE TABLE `emp` (
  `id` INT(11) DEFAULT NULL,
  `name` VARCHAR(100) DEFAULT NULL,
  `deg` VARCHAR(100) DEFAULT NULL,
  `salary` INT(11) DEFAULT NULL,
  `dept` VARCHAR(10) DEFAULT NULL,
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `is_delete` BIGINT(20) DEFAULT '1'
) ENGINE=INNODB DEFAULT CHARSET=latin1;

INSERT  INTO `emp`(`id`,`name`,`deg`,`salary`,`dept`) VALUES (1201,'gopal','manager',50000,'TP'),(1202,'manisha','Proof reader',50000,'TP'),(1203,'khalil','php dev',30000,'AC'),(1204,'prasanth','php dev',30000,'AC'),(1205,'kranthi','admin',20000,'TP');

DROP TABLE IF EXISTS `emp_add`;

CREATE TABLE `emp_add` (
  `id` INT(11) DEFAULT NULL,
  `hno` VARCHAR(100) DEFAULT NULL,
  `street` VARCHAR(100) DEFAULT NULL,
  `city` VARCHAR(100) DEFAULT NULL,
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `is_delete` BIGINT(20) DEFAULT '1'
) ENGINE=INNODB DEFAULT CHARSET=latin1;

INSERT  INTO `emp_add`(`id`,`hno`,`street`,`city`) VALUES (1201,'288A','vgiri','jublee'),(1202,'108I','aoc','sec-bad'),(1203,'144Z','pgutta','hyd'),(1204,'78B','old city','sec-bad'),(1205,'720X','hitec','sec-bad');

DROP TABLE IF EXISTS `emp_conn`;
CREATE TABLE `emp_conn` (
  `id` INT(100) DEFAULT NULL,
  `phno` VARCHAR(100) DEFAULT NULL,
  `email` VARCHAR(100) DEFAULT NULL,
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `is_delete` BIGINT(20) DEFAULT '1'
) ENGINE=INNODB DEFAULT CHARSET=latin1;

INSERT  INTO `emp_conn`(`id`,`phno`,`email`) VALUES (1201,'2356742','gopal@tp.com'),(1202,'1661663','manisha@tp.com'),(1203,'8887776','khalil@ac.com'),(1204,'9988774','prasanth@ac.com'),(1205,'1231231','kranthi@tp.com');
```

### 导入数据库表数据到HDFS，使用sqoop命令导入、导出数据前，要先启动hadoop集群
下面的命令用于从MySQL数据库服务器中的emp表导入HDFS。
```shell
sudo -u hdfs bin/sqoop import --connect jdbc:mysql://node01:3306/userdb --password '!Qaz123456' --username root --table emp --m 1
```

![](https://static.studytime.xin/article/20201117133423.png)

```shell
# 为了验证在HDFS导入的数据，请使用以下命令查看导入的数据

hdfs dfs -ls /user/root/emp
```

### 导入到HDFS指定目录
在导入表数据到HDFS使用Sqoop导入工具，我们可以指定目标目录。
使用参数 `--target-dir`来指定导出目的地，使用参数`--delete-target-dir`来判断导出目录是否存在，如果存在就删掉
```shell
bin/sqoop import --connect jdbc:mysql://node01:3306/userdb --username root --password '!Qaz123456' --delete-target-dir --table emp   --target-dir /sqoop/emp --m 1
```

```shell
# 查看导出的数据，默认使用逗号隔开

[root@node01 ~]# hdfs dfs -cat /sqoop/emp/part-m-00000
1201,gopal,manager,50000,TP,2020-11-17 13:29:18.0,2020-11-17 13:29:18.0,1
1202,manisha,Proof reader,50000,TP,2020-11-17 13:29:18.0,2020-11-17 13:29:18.0,1
1203,khalil,php dev,30000,AC,2020-11-17 13:29:18.0,2020-11-17 13:29:18.0,1
1204,prasanth,php dev,30000,AC,2020-11-17 13:29:18.0,2020-11-17 13:29:18.0,1
1205,kranthi,admin,20000,TP,2020-11-17 13:29:18.0,2020-11-17 13:29:18.0,1
```

### 导入到hdfs指定目录并指定字段之间的分隔符
```shell
bin/sqoop import --connect jdbc:mysql://node01:3306/userdb --username root --password 123456 --delete-target-dir --table emp --target-dir /sqoop/emp2 --m 1 --fields-terminated-by '\t'
```

```
[root@node01 ~]# hdfs dfs -cat /sqoop/emp2/part-m-00000

1201	gopal	manager	50000	TP	2020-11-17 13:29:18.0	2020-11-17 13:29:18.0	1
1202	manisha	Proof reader	50000	TP	2020-11-17 13:29:18.0	2020-11-17 13:29:18.0	1
1203	khalil	php dev	30000	AC	2020-11-17 13:29:18.0	2020-11-17 13:29:18.0	1
1204	prasanth	php dev	30000	AC	2020-11-17 13:29:18.0	2020-11-17 13:29:18.0	1
1205	kranthi	admin	20000	TP	2020-11-17 13:29:18.0	2020-11-17 13:29:18.0	1
```

### 导入关系表到HIVE

#### 第一步：拷贝jar包
将我们mysql表当中的数据直接导入到hive表中的话，我们需要将hive的一个叫做hive-exec-1.1.0-cdh5.14.0.jar的jar包拷贝到sqoop的lib目录下
```shell
cp /opt/module/hive-1.1.0-cdh5.14.0/lib/hive-exec-1.1.0-cdh5.14.2.jar /kkb/install/sqoop-1.4.6-cdh5.14.2/lib/
```

#### 第二步：准备hive数据库与表
```shell
hive (default)> create database sqooptohive;
hive (default)> use sqooptohive;
hive (sqooptohive)> create external table emp_hive(id int,name string,deg string,salary int ,dept string) row format delimited fields terminated by '\001';
```

#### 第三步：开始导入
```shell
bin/sqoop import --connect jdbc:mysql://node01:3306/userdb --username root --password '!Qaz123456' --table emp --fields-terminated-by '\001' --hive-import --hive-table sqooptohive.emp_hive --hive-overwrite --delete-target-dir --m 1
```

#### 第四步：hive表数据查看
```
select * from emp_hive;
```

![](https://static.studytime.xin/article/20201117134851.png)

### 导入关系表到hive并自动创建hive表
我们也可以通过命令来将我们的mysql的表直接导入到hive表当中去
```shell
bin/sqoop import --connect jdbc:mysql://node03:3306/userdb --username root --password 123456 --table emp_conn --hive-import -m 1 --hive-database sqooptohive;
```

通过这个命令，我们可以直接将我们mysql表当中的数据以及表结构一起倒入到hive当中去

### 将mysql表数据导入到hbase当中去,先要开启hbase集群

#### 第一步：修改sqoop配置文件
sqoop导入导出HBase的数据，需要修改sqoop的配置文件sqoop-env.sh

```
cd /opt/module/sqoop-1.4.6-cdh5.14.2/conf
vim sqoop-env.sh

#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.6.0-cdh5.14.2

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.6.0-cdh5.14.2

#set the path to where bin/hbase is available
export HBASE_HOME=/opt/module/hbase-1.2.0-cdh5.14.2

#Set the path to where bin/hive is available
export HIVE_HOME=/opt/module/hive-1.1.0-cdh5.14.2
```

 
#### 第二步：在mysql当中创建数据库以及数据库表并插入数据
```
CREATE DATABASE IF NOT EXISTS library;
USE library;
CREATE TABLE book(
id INT(4) PRIMARY KEY NOT NULL AUTO_INCREMENT, 
NAME VARCHAR(255) NOT NULL, 
price VARCHAR(255) NOT NULL);

INSERT INTO book(NAME, price) VALUES('Lie Sporting', '30'); 
INSERT INTO book (NAME, price) VALUES('Pride & Prejudice', '70'); 
INSERT INTO book (NAME, price) VALUES('Fall of Giants', '50'); 
```



#### 第三步：将mysql表当中的数据导入HBase表当中去
```
bin/sqoop import \
--connect jdbc:mysql://node03:3306/library \
--username root \
--password 123456 \
--table book \
--columns "id,name,price" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_book" \
--num-mappers 1 \
--split-by id
```

#### 第四步：HBase当中查看表数据
```
hbase(main):057:0> scan 'hbase_book'
ROW           COLUMN+CELL                            
 1            column=info:name, timestamp=1550634017823, value=Lie Sporting   
 1            column=info:price, timestamp=1550634017823, value=30       
 2            column=info:name, timestamp=1550634017823, value=Pride & Prejudice 
 2            column=info:price, timestamp=1550634017823, value=70       
 3            column=info:name, timestamp=1550634017823, value=Fall of Giants  
 3            column=info:price, timestamp=1550634017823, value=50
```

### 导入表数据子集
我们可以导入表的使用Sqoop导入工具，"where"子句的一个子集。它执行在各自的数据库服务器相应的SQL查询，并将结果存储在HDFS的目标目录。
```
--where <condition>  
```

- 按照条件进行查找，通过--where参数来查找表emp_add当中city字段的值为sec-bad的所有数据导入到hdfs上面去

```
bin/sqoop import \
--connect jdbc:mysql://node01:3306/userdb \
--username root --password '!Qaz123456' --table emp_add \
--target-dir /sqoop/emp_add -m 1 --delete-target-dir \
--where "city = 'sec-bad'"
```

#### sql语句查找导入hdfs

- 我们还可以通过 –query参数来指定我们的sql语句，通过sql语句来过滤我们的数据进行导入

```
bin/sqoop import \
--connect jdbc:mysql://node01:3306/userdb --username root --password '!Qaz123456' \
--delete-target-dir -m 1 \
--query 'select phno from emp_conn where 1=1 and  $CONDITIONS' \
--target-dir /sqoop/emp_conn
```

- 查看hdfs数据内容
```
hdfs dfs -text /sqoop/emp_conn/part*
```

> 注意：	
使用sql语句来进行查找是不能加参数--table 
并且必须要添加where条件，
并且where条件后面必须带一个$CONDITIONS 这个字符串，
并且这个sql语句必须用单引号，不能用双引号
	

#### 增量导入
在实际工作当中，数据的导入，很多时候都是只需要导入增量数据即可，并不需要将表中的数据全部导入到hive或者hdfs当中去，肯定会出现重复的数据的状况，所以我们一般都是选用一些字段进行增量的导入，为了支持增量的导入，sqoop也给我们考虑到了这种情况并且支持增量的导入数据.

- 增量导入是仅导入新添加的表中的行的技术。
- 它需要添加‘incremental’, ‘check-column’, 和 ‘last-value’选项来执行增量导入。

- 下面的语法用于Sqoop导入命令增量选项。
```
Incremental import arguments:
   --check-column <column>        Source column to check for incremental
                                  change
   --incremental <import-type>    Define an incremental import of type
                                  'append' or 'lastmodified'
   --last-value <value>           Last imported value in the incremental
                                  check columns
```

##### 第一种增量导入使用上面的选项来实现

- 导入emp表当中id大于1202的所有数据

- 注意：增量导入的时候，一定不能加参数--delete-target-dir否则会报错

```
bin/sqoop import \
--connect jdbc:mysql://node01:3306/userdb \
--username root \
--password '!Qaz123456' \
--table emp \
--incremental append \
--check-column id \
--last-value 1202 \
-m 1 \
--target-dir /sqoop/increment
```

- 查看数据内容
```
hdfs dfs -text /sqoop/increment/part*
```

##### 第二种增量导入通过--where条件来实现
- 或者我们使用--where来进行控制数据的选取会更加精准

```
bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password '!Qaz123456' \
--table emp \
--incremental append \
--where "create_time > '2018-06-17 00:00:00' and is_delete='1' and create_time < '2018-06-17 23:59:59'" \
--target-dir /sqoop/incement2 \
--check-column id \
--m 1
```

- 作业：增量导入hive表中该如何实现？？？

- 面试题：如何解决减量数据？？？

 

### Sqoop的数据导出

#### 将数据从HDFS把文件导出到RDBMS数据库
导出前，目标表必须存在于目标数据库中, 默认操作是从将文件中的数据使用INSERT语句插入到表中, 更新模式下，是生成UPDATE语句更新表数据, 数据是在HDFS当中的如下目录/sqoop/emp，数据内容如下

```
1201,gopal,manager,50000,TP,2018-06-17 18:54:32.0,2018-06-17 18:54:32.0,1
1202,manisha,Proof reader,50000,TP,2018-06-15 18:54:32.0,2018-06-17 20:26:08.0,1
1203,khalil,php dev,30000,AC,2018-06-17 18:54:32.0,2018-06-17 18:54:32.0,1
1204,prasanth,php dev,30000,AC,2018-06-17 18:54:32.0,2018-06-17 21:05:52.0,0
1205,kranthi,admin,20000,TP,2018-06-17 18:54:32.0,2018-06-17 18:54:32.0,1
```

##### 第一步：创建mysql表

```
CREATE TABLE `emp_out` (
 `id` INT(11) DEFAULT NULL,
 `name` VARCHAR(100) DEFAULT NULL,
 `deg` VARCHAR(100) DEFAULT NULL,
 `salary` INT(11) DEFAULT NULL,
 `dept` VARCHAR(10) DEFAULT NULL,
 `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
 `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
 `is_delete` BIGINT(20) DEFAULT '1'
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```
##### 第二步：执行导出命令

通过kkb来实现数据的导出，将hdfs的数据导出到mysql当中去

```
bin/sqoop export \
--connect jdbc:mysql://node01:3306/userdb \
--username root --password '!Qaz123456' \
--table emp_out \
--export-dir /sqoop/emp \
--input-fields-terminated-by ","
```

##### 第三步：验证mysql表数据
```mysql
mysql> select * from emp_out;
+------+----------+--------------+--------+------+---------------------+---------------------+-----------+
| id   | name     | deg          | salary | dept | create_time         | update_time         | is_delete |
+------+----------+--------------+--------+------+---------------------+---------------------+-----------+
| 1203 | khalil   | php dev      |  30000 | AC   | 2020-11-17 13:29:18 | 2020-11-17 13:29:18 |         1 |
| 1204 | prasanth | php dev      |  30000 | AC   | 2020-11-17 13:29:18 | 2020-11-17 13:29:18 |         1 |
| 1205 | kranthi  | admin        |  20000 | TP   | 2020-11-17 13:29:18 | 2020-11-17 13:29:18 |         1 |
| 1201 | gopal    | manager      |  50000 | TP   | 2020-11-17 13:29:18 | 2020-11-17 13:29:18 |         1 |
| 1202 | manisha  | Proof reader |  50000 | TP   | 2020-11-17 13:29:18 | 2020-11-17 13:29:18 |         1 |
+------+----------+--------------+--------+------+---------------------+---------------------+-----------+
5 rows in set (0.00 sec)
```

#### 将数据从Hbase导出到mysql
将hbase_book这张表当中的数据导出到mysql当中来
注意：sqoop不支持我们直接将HBase当中的数据导出，所以我们可以通过以下的转换进行导出
Hbase→hive外部表→hive内部表→通过sqoop→mysql

##### 第一步：创建hive外部表
- 进入hive客户端，创建hive外部表，映射hbase当中的hbase_book表
```
CREATE EXTERNAL TABLE course.hbase2mysql (id int,name string,price int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
"hbase.columns.mapping" =
":key,info:name, info:price"
)
TBLPROPERTIES( "hbase.table.name" = "hbase_book",
"hbase.mapred.output.outputtable" = "hbase2mysql");
```

##### 第二步：创建hive内部表并将外部表数据插入到内部表当中来

- 进入hive客户端，执行以下命令，创建hive内部表，并将外部表的数据插入到hive的内部表当中来

```
CREATE TABLE course.hbase2mysqlin(id int,name string,price int);
```

##### 第三步：外部表数据插入内部表

- 进入hive客户端执行以下命令，将hive外部表数据插入到hive内部表当中来

```
insert overwrite table course.hbase2mysqlin select * from course.hbase2mysql;
```



##### 第四步：清空mysql表数据

- 进入mysql客户端，执行以下命令，将mysql表数据清空

```
TRUNCATE TABLE book;
```

##### 第五步：执行sqoop导出hive内部表数据到

```
sqoop export -connect jdbc:mysql://node03:3306/library -username root -password 123456 -table book -export-dir /user/hive/warehouse/course.db/hbase2mysqlin --input-fields-terminated-by '\001' --input-null-string '\\N' --input-null-non-string '\\N';
```



### Sqoop常用命令及参数

#### 常用命令列举

- 这里给大家列出来了一部分Sqoop操作时的常用参数，以供参考，需要深入学习的可以参看对应类的源代码。

| **序号** | **命令**          | **类**              | **说明**                                                     |
| -------- | ----------------- | ------------------- | ------------------------------------------------------------ |
| 1        | import            | ImportTool          | 将数据导入到集群                                             |
| 2        | export            | ExportTool          | 将集群数据导出                                               |
| 3        | codegen           | CodeGenTool         | 获取数据库中某张表数据生成Java并打包Jar                      |
| 4        | create-hive-table | CreateHiveTableTool | 创建Hive表                                                   |
| 5        | eval              | EvalSqlTool         | 查看SQL执行结果                                              |
| 6        | import-all-tables | ImportAllTablesTool | 导入某个数据库下所有表到HDFS中                               |
| 7        | job               | JobTool             | 用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务。 |
| 8        | list-databases    | ListDatabasesTool   | 列出所有数据库名                                             |
| 9        | list-tables       | ListTablesTool      | 列出某个数据库下所有表                                       |
| 10       | merge             | MergeTool           | 将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中     |
| 11       | metastore         | MetastoreTool       | 记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为：~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改。 |
| 12       | help              | HelpTool            | 打印sqoop帮助信息                                            |
| 13       | version           | VersionTool         | 打印sqoop版本信息                                            |

#### 命令&参数详解

- 刚才列举了一些Sqoop的常用命令，对于不同的命令，有不同的参数，让我们来一一列举说明。

  首先来我们来介绍一下公用的参数，所谓公用参数，就是大多数命令都支持的参数。

##### 1、公用参数：数据库连接

| **序号** | **参数**             | **说明**               |
| -------- | -------------------- | ---------------------- |
| 1        | --connect            | 连接关系型数据库的URL  |
| 2        | --connection-manager | 指定要使用的连接管理类 |
| 3        | --driver             | JDBC的driver class     |
| 4        | --help               | 打印帮助信息           |
| 5        | --password           | 连接数据库的密码       |
| 6        | --username           | 连接数据库的用户名     |
| 7        | --verbose            | 在控制台打印出详细信息 |

##### 2、公用参数：import

| **序号** | **参数**                         | **说明**                                                     |
| -------- | -------------------------------- | ------------------------------------------------------------ |
| 1        | --enclosed-by  <char>            | 给字段值前后加上指定的字符                                   |
| 2        | --escaped-by  <char>             | 对字段中的双引号加转义符                                     |
| 3        | --fields-terminated-by  <char>   | 设定每个字段是以什么符号作为结束，默认为逗号                 |
| 4        | --lines-terminated-by  <char>    | 设定每行记录之间的分隔符，默认是\n                           |
| 5        | --mysql-delimiters               | Mysql默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹。 |
| 6        | --optionally-enclosed-by  <char> | 给带有双引号或单引号的字段值前后加上指定字符。               |

##### 3、公用参数：export

| **序号** | **参数**                               | **说明**                                   |
| -------- | -------------------------------------- | ------------------------------------------ |
| 1        | --input-enclosed-by  <char>            | 对字段值前后加上指定字符                   |
| 2        | --input-escaped-by  <char>             | 对含有转移符的字段做转义处理               |
| 3        | --input-fields-terminated-by  <char>   | 字段之间的分隔符                           |
| 4        | --input-lines-terminated-by  <char>    | 行之间的分隔符                             |
| 5        | --input-optionally-enclosed-by  <char> | 给带有双引号或单引号的字段前后加上指定字符 |

##### 4、公用参数：hive

| **序号** | **参数**                         | **说明**                                                  |
| -------- | -------------------------------- | --------------------------------------------------------- |
| 1        | --hive-delims-replacement  <arg> | 用自定义的字符串替换掉数据中的\r\n和\013 \010等字符       |
| 2        | --hive-drop-import-delims        | 在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符    |
| 3        | --map-column-hive  <map>         | 生成hive表时，可以更改生成字段的数据类型                  |
| 4        | --hive-partition-key             | 创建分区，后面直接跟分区名，分区字段的默认类型为string    |
| 5        | --hive-partition-value  <v>      | 导入数据时，指定某个分区的值                              |
| 6        | --hive-home  <dir>               | hive的安装目录，可以通过该参数覆盖之前默认配置的目录      |
| 7        | --hive-import                    | 将数据从关系数据库中导入到hive表中                        |
| 8        | --hive-overwrite                 | 覆盖掉在hive表中已经存在的数据                            |
| 9        | --create-hive-table              | 默认是false，即，如果目标表已经存在了，那么创建任务失败。 |
| 10       | --hive-table                     | 后面接要创建的hive表,默认使用MySQL的表名                  |
| 11       | --table                          | 指定关系数据库的表名                                      |

 

公用参数介绍完之后，我们来按照命令介绍命令对应的特有参数。

##### 5、命令&参数：import

- 将关系型数据库中的数据导入到HDFS（包括Hive，HBase）中，如果导入的是Hive，那么当Hive中没有对应表时，则自动创建。

**1)** **命令：**

如：导入数据到hive中

```
$ bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp \
--hive-import
```

如：增量导入数据到hive中，mode=append

append导入：  

```
$ bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp \
--num-mappers 1 \
--fields-terminated-by "\t" \
--target-dir /user/hive/warehouse/emp \
--check-column id \
--incremental append \
--last-value 3
```

易错提醒：append不能与--hive-等参数同时使用（Append mode for hive imports is not yet supported. Please remove the parameter --append-mode）

如：增量导入数据到hdfs中，mode=lastmodified

先在mysql中建表并插入几条数据：  

```
mysql> create table company.staff_timestamp(id int(4), name varchar(255), sex varchar(255), last_modified timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);
mysql> insert into company.staff_timestamp (id, name, sex) values(1, 'AAA', 'female');
mysql> insert into company.staff_timestamp (id, name, sex) values(2, 'BBB', 'female');
```

先导入一部分数据：  

```
$ bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_conn \
--delete-target-dir \
--m 1
```

再增量导入一部分数据：  

```
mysql> insert into company.staff_timestamp (id, name, sex) values(3, 'CCC', 'female');
$ bin/sqoop import \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_conn \
--check-column last_modified \
--incremental lastmodified \
--last-value "2018-0-28 22:20:38" \
-m 1 \
--append
```

易错提醒：使用lastmodified方式导入数据要指定增量数据是要--append（追加）还是要--merge-key（合并）

易错提醒：--incremental lastmodified模式下，last-value指定的值是会包含于增量导入的数据中。

**2)** **参数：**

| **序号** | **参数**                         | **说明**                                                     |
| -------- | -------------------------------- | ------------------------------------------------------------ |
| 1        | --append                         | 将数据追加到HDFS中已经存在的DataSet中，如果使用该参数，sqoop会把数据先导入到临时文件目录，再合并。 |
| 2        | --as-avrodatafile                | 将数据导入到一个Avro数据文件中                               |
| 3        | --as-sequencefile                | 将数据导入到一个sequence文件中                               |
| 4        | --as-textfile                    | 将数据导入到一个普通文本文件中                               |
| 5        | --boundary-query  <statement>    | 边界查询，导入的数据为该参数的值（一条sql语句）所执行的结果区间内的数据。 |
| 6        | --columns <col1, col2,  col3>    | 指定要导入的字段                                             |
| 7        | --direct                         | 直接导入模式，使用的是关系数据库自带的导入导出工具，以便加快导入导出过程。 |
| 8        | --direct-split-size              | 在使用上面direct直接导入的基础上，对导入的流按字节分块，即达到该阈值就产生一个新的文件 |
| 9        | --inline-lob-limit               | 设定大对象数据类型的最大值                                   |
| 10       | --m或–num-mappers                | 启动N个map来并行导入数据，默认4个。                          |
| 11       | --query或--e <statement>         | 将查询结果的数据导入，使用时必须伴随参--target-dir，--hive-table，如果查询中有where条件，则条件后必须加上$CONDITIONS关键字 |
| 12       | --split-by <column-name>         | 按照某一列来切分表的工作单元，不能与--autoreset-to-one-mapper连用（请参考官方文档） |
| 13       | --table <table-name>             | 关系数据库的表名                                             |
| 14       | --target-dir <dir>               | 指定HDFS路径                                                 |
| 15       | --warehouse-dir <dir>            | 与14参数不能同时使用，导入数据到HDFS时指定的目录             |
| 16       | --where                          | 从关系数据库导入数据时的查询条件                             |
| 17       | --z或--compress                  | 允许压缩                                                     |
| 18       | --compression-codec              | 指定hadoop压缩编码类，默认为gzip(Use Hadoop codec  default gzip) |
| 19       | --null-string  <null-string>     | string类型的列如果null，替换为指定字符串                     |
| 20       | --null-non-string  <null-string> | 非string类型的列如果null，替换为指定字符串                   |
| 21       | --check-column <col>             | 作为增量导入判断的列名                                       |
| 22       | --incremental <mode>             | mode：append或lastmodified                                   |
| 23       | --last-value <value>             | 指定某一个值，用于标记增量导入的位置                         |

##### 6、命令&参数：export

从HDFS（包括Hive和HBase）中奖数据导出到关系型数据库中。

**1)** **命令：**

**如：**

```
$ bin/sqoop export \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_add \
--export-dir /user/company \
--input-fields-terminated-by "\t" \
--num-mappers 1
```

 

**2)** **参数：**

| **序号** | **参数**                               | **说明**                                                     |
| -------- | -------------------------------------- | ------------------------------------------------------------ |
| 1        | --direct                               | 利用数据库自带的导入导出工具，以便于提高效率                 |
| 2        | --export-dir  <dir>                    | 存放数据的HDFS的源目录                                       |
| 3        | -m或--num-mappers  <n>                 | 启动N个map来并行导入数据，默认4个                            |
| 4        | --table  <table-name>                  | 指定导出到哪个RDBMS中的表                                    |
| 5        | --update-key  <col-name>               | 对某一列的字段进行更新操作                                   |
| 6        | --update-mode  <mode>                  | updateonly  allowinsert(默认)                                |
| 7        | --input-null-string  <null-string>     | 请参考import该类似参数说明                                   |
| 8        | --input-null-non-string  <null-string> | 请参考import该类似参数说明                                   |
| 9        | --staging-table  <staging-table-name>  | 创建一张临时表，用于存放所有事务的结果，然后将所有事务结果一次性导入到目标表中，防止错误。 |
| 10       | --clear-staging-table                  | 如果第9个参数非空，则可以在导出操作执行前，清空临时事务结果表 |

 

##### 7、命令&参数：codegen

将关系型数据库中的表映射为一个Java类，在该类中有各列对应的各个字段。

如：

```
$ bin/sqoop codegen \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_add \
--bindir /home/admin/Desktop/staff \
--class-name Staff \
--fields-terminated-by "\t"
```

 

| **序号** | **参数**                            | **说明**                                                     |
| -------- | ----------------------------------- | ------------------------------------------------------------ |
| 1        | --bindir  <dir>                     | 指定生成的Java文件、编译成的class文件及将生成文件打包为jar的文件输出路径 |
| 2        | --class-name  <name>                | 设定生成的Java文件指定的名称                                 |
| 3        | --outdir  <dir>                     | 生成Java文件存放的路径                                       |
| 4        | --package-name  <name>              | 包名，如com.z，就会生成com和z两级目录                        |
| 5        | --input-null-non-string  <null-str> | 在生成的Java文件中，可以将null字符串或者不存在的字符串设置为想要设定的值（例如空字符串） |
| 6        | --input-null-string  <null-str>     | 将null字符串替换成想要替换的值（一般与5同时使用）            |
| 7        | --map-column-java <arg>             | 数据库字段在生成的Java文件中会映射成各种属性，且默认的数据类型与数据库类型保持对应关系。该参数可以改变默认类型，例如：--map-column-java  id=long, name=String |
| 8        | --null-non-string  <null-str>       | 在生成Java文件时，可以将不存在或者null的字符串设置为其他值   |
| 9        | --null-string <null-str>            | 在生成Java文件时，将null字符串设置为其他值（一般与8同时使用） |
| 10       | --table <table-name>                | 对应关系数据库中的表名，生成的Java文件中的各个属性与该表的各个字段一一对应 |

##### 8、命令&参数：create-hive-table

生成与关系数据库表结构对应的hive表结构。

**命令：**

```
$ bin/sqoop create-hive-table \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_add \
--hive-table emp_add
```

**参数：**

| **序号** | **参数**            | **说明**                                              |
| -------- | ------------------- | ----------------------------------------------------- |
| 1        | --hive-home  <dir>  | Hive的安装目录，可以通过该参数覆盖掉默认的Hive目录    |
| 2        | --hive-overwrite    | 覆盖掉在Hive表中已经存在的数据                        |
| 3        | --create-hive-table | 默认是false，如果目标表已经存在了，那么创建任务会失败 |
| 4        | --hive-table        | 后面接要创建的hive表                                  |
| 5        | --table             | 指定关系数据库的表名                                  |

##### 9、命令&参数：eval

可以快速的使用SQL语句对关系型数据库进行操作，经常用于在import数据之前，了解一下SQL语句是否正确，数据是否正常，并可以将结果显示在控制台。

**命令：**

```
$ bin/sqoop eval \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--query "SELECT * FROM emp"
```

**参数：**

| **序号** | **参数**     | **说明**          |
| -------- | ------------ | ----------------- |
| 1        | --query或--e | 后跟查询的SQL语句 |

 

##### 10、命令&参数：import-all-tables

可以将RDBMS中的所有表导入到HDFS中，每一个表都对应一个HDFS目录

**命令：**

```
$ bin/sqoop import-all-tables \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--warehouse-dir /all_tables
```

 

**参数：**

| **序号** | **参数**                 | **说明**                               |
| -------- | ------------------------ | -------------------------------------- |
| 1        | --as-avrodatafile        | 这些参数的含义均和import对应的含义一致 |
| 2        | --as-sequencefile        |                                        |
| 3        | --as-textfile            |                                        |
| 4        | --direct                 |                                        |
| 5        | --direct-split-size  <n> |                                        |
| 6        | --inline-lob-limit  <n>  |                                        |
| 7        | --m或—num-mappers  <n>   |                                        |
| 8        | --warehouse-dir  <dir>   |                                        |
| 9        | -z或--compress           |                                        |
| 10       | --compression-codec      |                                        |

 

##### 11、命令&参数：job

用来生成一个sqoop任务，生成后不会立即执行，需要手动执行。

**命令：**

```
$ bin/sqoop job \
 --create myjob -- import-all-tables \
 --connect jdbc:mysql://node03:3306/userdb \
 --username root \
 --password 123456
$ bin/sqoop job \
--list
$ bin/sqoop job \
--exec myjob
```

易错提醒：注意import-all-tables和它左边的--之间有一个空格

易错提醒：如果需要连接metastore，则--meta-connect jdbc:hsqldb:hsql://node03:16000/sqoop

参数：

| **序号** | **参数**                   | **说明**                 |
| -------- | -------------------------- | ------------------------ |
| 1        | --create  <job-id>         | 创建job参数              |
| 2        | --delete  <job-id>         | 删除一个job              |
| 3        | --exec  <job-id>           | 执行一个job              |
| 4        | --help                     | 显示job帮助              |
| 5        | --list                     | 显示job列表              |
| 6        | --meta-connect  <jdbc-uri> | 用来连接metastore服务    |
| 7        | --show  <job-id>           | 显示一个job的信息        |
| 8        | --verbose                  | 打印命令运行时的详细信息 |

易错提醒：在执行一个job时，如果需要手动输入数据库密码，可以做如下优化

```
<property>
	<name>sqoop.metastore.client.record.password</name>
	<value>true</value>
	<description>If true, allow saved passwords in the metastore.</description>
</property>
```

 

##### 12、命令&参数：list-databases

**命令：**

如：

```
$ bin/sqoop list-databases \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456
```

 

**参数：**与公用参数一样

##### 13、命令&参数：list-tables

**命令：**

如：

```
$ bin/sqoop list-tables \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456
```

**参数：**与公用参数一样

##### 14、命令&参数：merge

将HDFS中不同目录下面的数据合并在一起并放入指定目录中

数据环境：

```
new_staff
1       AAA     male
2       BBB     male
3       CCC     male
4       DDD     male
old_staff
1       AAA     female
2       CCC     female
3       BBB     female
6       DDD     female
```

易错提醒：上边数据的列之间的分隔符应该为\t，行与行之间的分割符为\n，如果直接复制，请检查之。

**命令：**

如：

创建JavaBean：  

```
$ bin/sqoop codegen \
--connect jdbc:mysql://node03:3306/userdb \
--username root \
--password 123456 \
--table emp_conn \
--bindir /home/admin/Desktop/staff \
--class-name EmpConn \
--fields-terminated-by "\t"
```

开始合并：  

```
$ bin/sqoop merge \
--new-data /test/new/ \
--onto /test/old/ \
--target-dir /test/merged \
--jar-file /home/admin/Desktop/staff/EmpConn.jar \
--class-name Staff \
--merge-key id
```

结果：  

```
1	AAA	MALE
2	BBB	MALE
3	CCC	MALE
4	DDD	MALE
6	DDD	FEMALE
```

 

参数：

| **序号** | **参数**              | **说明**                                               |
| -------- | --------------------- | ------------------------------------------------------ |
| 1        | --new-data  <path>    | HDFS  待合并的数据目录，合并后在新的数据集中保留       |
| 2        | --onto  <path>        | HDFS合并后，重复的部分在新的数据集中被覆盖             |
| 3        | --merge-key  <col>    | 合并键，一般是主键ID                                   |
| 4        | --jar-file  <file>    | 合并时引入的jar包，该jar包是通过Codegen工具生成的jar包 |
| 5        | --class-name  <class> | 对应的表名或对象名，该class类是包含在jar包中的         |
| 6        | --target-dir  <path>  | 合并后的数据在HDFS里存放的目录                         |

 

##### 15、命令&参数：metastore

记录了Sqoop job的元数据信息，如果不启动该服务，那么默认job元数据的存储目录为~/.sqoop，可在sqoop-site.xml中修改。

**命令：**

如：启动sqoop的metastore服务

```
$ bin/sqoop metastore
```
**参数：**

| **序号** | **参数**   | **说明**      |
| -------- | ---------- | ------------- |
| 1        | --shutdown | 关闭metastore |
