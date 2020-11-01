---
title: hive 学习之路（二）伪分布式安装
permalink: hive-knowledge-install
date: 2020-05-04 17:22:23
updated: 2020-05-04 17:25:43
tags:
    - 大数据
    - hive
categories: [大数据,hive]
toc: true
priority: 0
excerpt: Hive 是基于 Hadoop 的数据仓库解决方案，所以默认代表已经安装 Hadoop，本文只要整理了伪分布式场景下的安装和简单实用案例。
---

### Hive安装前置条件
- 按照 Hive 初始中所说，Hive 是基于 Hadoop 的数据仓库解决方案，所以默认代表已经安装 Hadoop。
- 常用解决方案为适应 MySQL 做为 Hive 的元数据库，所以先安装 MySQL。
- 启动 HDFS 和 YARN

### Hive的下载
[下载链接http://mirror.bit.edu.cn/apache/hive/](http://mirror.bit.edu.cn/apache/hive/)

### 解压文件到指定安装目录
```
tar -zxvf apache-hive-2.3.3-bin.tar.gz -C ~/App/
```

### 配置环境变量

打开全局环境变量`~/.bash_profile`，将 Hive 安装包 bin 目录导入全局变量中，重载全局变量文件 `source ~/.bash_profile`。

```
vim ~/.bash_profile

export HIVE_HOME=/Users/baihe/App/apache-hive-2.1.1-bin
export PATH=$HIVE_HOME/bin:$PATH

source ~/.bash_profile
```

### 修改配置文件

#### 进入 hive 配置文件目录
```
cd /Users/baihe/App/apache-hive-2.1.1-bin/conf
```

#### 新建hive-site.xml并添加以下内容，打开`vi hive-site.xml`文件

```
<configuration>
        <property>
                <name>hive.metastore.uris</name>
                <value>thrift://localhost:9083</value>
        </property>
        <property>
                <name>hive.server2.thrift.port</name>
                <value>10000</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>baihe2019</value>
        </property>
        <property>
                <name>hive.metastore.schema.verification</name>
                <value>false</value>
        </property>
        <property>
                <name>hive.metastore.warehouse.dir</name>
                <value>/warehouse</value>
        </property>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://bigdata:9000</value>
        </property>
        <property>
                <name>datanucleus.autoCreateSchema</name>
                <value>true</value>
        </property>
        <property>
                <name>datanucleus.autoStartMechanism</name>
                <value>SchemaTable</value>
        </property>
        <property>
                <name>datanucleus.schema.autoCreateTables</name>
                <value>true</value>
        </property>
        <property>
                <name>beeline.hs2.connection.user</name>
                <value>baihe</value>
        </property>
        <property>
                <name>beeline.hs2.connection.password</name>
                <value>baihe</value>
        </property>
</configuration>
```

配置说明：
- hive.metastore.uris: metastore server所在的机器，以及使用端口
- hive.server2.thrift.port： server2 使用端口
- javax.jdo.option.ConnectionURL: metastore 数据库配置 host信息
- javax.jdo.option.ConnectionDriverName： metastore 数据库配置 MySQL JDBC驱动类
- javax.jdo.option.ConnectionUserName：metastore 数据库配置 用户
- javax.jdo.option.ConnectionPassword： metastore 数据库配置 密码
- hive.metastore.warehouse.dir: 本地表默认位置
- fs.defaultFS：为 HDFS 的 namenode 启动的机器地址
- beeline.hs2.connection.user : beeline连接用户名 
- beeline.hs2.connection.password： beeline连接密码

### 启动hive

#### 1.若为第一次启动，需要执行初始化命令,以后则只需直接启动 metastore 和 hiveserver
```
schematool -dbType mysql -initSchema
```

#### 2.启动 metastore 
```
nohup hive --service metastore >> /Users/baihe/App/apache-hive-2.1.1-bin/metastore.log 2>&1 &
```

#### 3.启动 hive server
```
nohup hive --service hiveserver2 >> /Users/baihe/App/apache-hive-2.1.1-bin/hiveserver.log 2>&1 &
```

#### 4.查看 hive metastore 和 hiveserver2 是否启动成功
```
ps aux | grep hive
```

能输出两个进程，分别对应 metastore 和 hiveserver2。

```
➜  ~ ps aux | grep hive
baihe             1222 178.2  1.4  6312516 228304 s000  RN   11:43上午   0:04.60 /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/bin/java -Xmx256m -Djava.library.path=/Users/baihe/App/hadoop-2.7.3/lib -Djava.net.preferIPv4Stack=true -Dhadoop.log.dir=/Users/baihe/App/hadoop-2.7.3/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/Users/baihe/App/hadoop-2.7.3 -Dhadoop.id.str=baihe -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Djava.net.preferIPv4Stack=true -Xmx512m -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/Users/baihe/App/apache-hive-2.1.1-bin/conf/parquet-logging.properties -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /Users/baihe/App/apache-hive-2.1.1-bin/lib/hive-metastore-2.1.1.jar org.apache.hadoop.hive.metastore.HiveMetaStore
baihe             1410   0.0  0.0  4334948    648 s000  R+   11:43上午   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn hive
baihe             1363   0.0  0.0  4324956    604 s000  SN   11:43上午   0:00.00 bash /Users/baihe/App/apache-hive-2.1.1-bin/bin/hive --service hiveserver2
baihe             1328   0.0  0.0  4324956   1880 s000  SN   11:43上午   0:00.04 bash /Users/baihe/App/apache-hive-2.1.1-bin/bin/hive --service hiveserver2
```

### Hive 常见两种访问方式
- hive
- beeline

```
➜  bin beeline
Beeline version 1.2.1.spark2 by Apache Hive
beeline> !connect jdbc:hive2://localhost:10000/default baihe baihe
Connecting to jdbc:hive2://localhost:10000/default
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/baihe/App/spark-2.1.1-bin-hadoop2.7/jars/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/baihe/App/hadoop-2.7.3/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
19/09/17 11:45:25 INFO jdbc.Utils: Supplied authorities: localhost:10000
19/09/17 11:45:25 INFO jdbc.Utils: Resolved authority: localhost:10000
19/09/17 11:45:26 INFO jdbc.HiveConnection: Will try to open client transport with JDBC Uri: jdbc:hive2://localhost:10000/default
Connected to: Apache Hive (version 2.1.1)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://localhost:10000/default>
```
其中 bigdata 和 bigdata 分别是在 hive-site.xml 配置文件中由 beeline.hs2.connection.user 和 beeline.hs2.connection.password 设置的。 注：如果要使用 beeline 或 JDBC 连接 hive 时，遇到报错：“User: xxx is not allowed to impersonate yyy”，需在 hadoop 的配置文件 core-site.xml 中加入以下配置（其中 红色标志的“xxx”是你启动 hive server2 和 hive metastore 所采用的用户，用户名中 不要包含“.”，比如“cheng.dong”是不支持的），并重启 hiveserver2， hive metastore，HDFS 和 YARN：

```
<property>

<name>hadoop.proxyuser.xxx.groups</name>

<value>*</value>

</property>

<property>

<name>hadoop.proxyuser.xxx.hosts</name>

<value>*</value>

</property>
```

### 使用练习

需求说明：现有一个文件student.txt，将其存入hive中，student.txt数据格式如下：
```
95002,刘晨,女,19,IS
95017,王风娟,女,18,IS
95018,王一,女,19,IS
95013,冯伟,男,21,CS
95014,王小丽,女,19,CS
95019,邢小丽,女,19,IS
95020,赵钱,男,21,IS
95003,王敏,女,22,MA
95004,张立,男,19,IS
95012,孙花,女,20,CS
95010,孔小涛,男,19,CS
95005,刘刚,男,18,MA
95006,孙庆,男,23,CS
95007,易思玲,女,19,MA
95008,李娜,女,18,CS
95021,周二,男,17,MA
95022,郑明,男,20,MA
95001,李勇,男,20,CS
95011,包小柏,男,18,MA
95009,梦圆圆,女,18,MA
95015,王君,男,18,MA
```

#### 1.本地新建测试数据文件
```
vim /tmp/student.txt

写入上述数据即可
```
#### 2.创建一个数据库myhive
```
0: jdbc:hive2://localhost:10000/default> create database myhive;
No rows affected (1.01 seconds)
```
#### 3.使用新的数据库myhive
```
0: jdbc:hive2://localhost:10000/default> use myhive;
No rows affected (0.071 seconds)
```
#### 4.查看当前正在使用的数据库
```
0: jdbc:hive2://localhost:10000/default> select current_database();
+---------+--+
|   _c0   |
+---------+--+
| myhive  |
+---------+--+
1 row selected (1.667 seconds)
```
#### 5.在数据库myhive创建一张student表
```
0: jdbc:hive2://localhost:10000/default> create table student(id int, name string, sex string, age int, department string) row format delimited fields terminated by ",";
No rows affected (0.316 seconds)
```
#### 6.往表中加载数据
```
0: jdbc:hive2://localhost:10000/default> load data local inpath "/tmp/student.txt" into table student;
No rows affected (0.432 seconds)
```
#### 7.查询数据
```
0: jdbc:hive2://localhost:10000/default> select * from student;
+-------------+---------------+--------------+--------------+---------------------+--+
| student.id  | student.name  | student.sex  | student.age  | student.department  |
+-------------+---------------+--------------+--------------+---------------------+--+
| 95002       | 刘晨            | 女            | 19           | IS                  |
| 95017       | 王风娟           | 女            | 18           | IS                  |
| 95018       | 王一            | 女            | 19           | IS                  |
| 95013       | 冯伟            | 男            | 21           | CS                  |
| 95014       | 王小丽           | 女            | 19           | CS                  |
| 95019       | 邢小丽           | 女            | 19           | IS                  |
| 95020       | 赵钱            | 男            | 21           | IS                  |
| 95003       | 王敏            | 女            | 22           | MA                  |
| 95004       | 张立            | 男            | 19           | IS                  |
| 95012       | 孙花            | 女            | 20           | CS                  |
| 95010       | 孔小涛           | 男            | 19           | CS                  |
| 95005       | 刘刚            | 男            | 18           | MA                  |
| 95006       | 孙庆            | 男            | 23           | CS                  |
| 95007       | 易思玲           | 女            | 19           | MA                  |
| 95008       | 李娜            | 女            | 18           | CS                  |
| 95021       | 周二            | 男            | 17           | MA                  |
| 95022       | 郑明            | 男            | 20           | MA                  |
| 95001       | 李勇            | 男            | 20           | CS                  |
| 95011       | 包小柏           | 男            | 18           | MA                  |
| 95009       | 梦圆圆           | 女            | 18           | MA                  |
| 95015       | 王君            | 男            | 18           | MA                  |
+-------------+---------------+--------------+--------------+---------------------+--+
21 rows selected (0.538 seconds)
```
#### 8.查看表结构

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
5 rows selected (0.086 seconds)

 desc extended student;
 
 desc formatted student;
```