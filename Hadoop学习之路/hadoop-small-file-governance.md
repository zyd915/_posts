---
title: HDFS 小文件治理方案
date: 2020-05-30 16:49:57
updated: 2020-05-30 16:53:57
tags: 
    - 大数据
    - hadoop
categories: [大数据,hadoop]
toc: true
excerpt: HDFS小文件是指文件远远小于HDFS配置的block文件大小的文件。在HDFS上中block的文件目录数、或增删改查操作等都是存储在内存中，以对象的方式存储，每个对象约占150byte。若大量的小文件存储占用一个block，则会占用大量NameNode内存。而集群存储文件的多少，由NameNode管理，

---

HDFS小文件是指文件远远小于HDFS配置的block文件大小的文件。在HDFS上中block的文件目录数、或增删改查操作等都是存储在内存中，以对象的方式存储，每个对象约占150byte。若大量的小文件存储占用一个block，则会占用大量NameNode内存。而集群存储文件的多少，由NameNode管理，

### 常见方案
- 小文件上传时合并上传
- Hadoop Archive方式
- Sequence file方式

### 小文件上传时合并上传
#### 将本地的小文件合并，上传到HDFS
本地存在多个小文件，需要上传到HDFS,HDFS的appendToFile命令可以实现多个本地文件合并上传HDFS。

```
cd /home/hadoop

[hadoop@node01 ~]$ cat world1.txt
aaa
bbb
ccc
ddd
eee
fff
hhh
[hadoop@node01 ~]$ cat world.txt
aaa
bbb
ccc
ddd
eee
fff
hhh

[hadoop@node01 ~]$ hdfs dfs -appendToFile world1.txt world.txt /tmp/world.txt

[hadoop@node01 ~]$ hdfs dfs -cat /tmp/world.txt
aaa
bbb
ccc
ddd
eee
fff
hhh
aaa
bbb
ccc
ddd
eee
fff
hhh
```

#### 下载HDFS的小文件到本地，合并成一个大文件
HDFS 存在多个小文件，下载合并到本地生成一个大文件。

```
[hadoop@node01 ~]$ hdfs dfs -mkdir /baihe
[hadoop@node01 ~]$ hdfs dfs -copyFromLocal world.txt /baihe
[hadoop@node01 ~]$ hdfs dfs -copyFromLocal world1.txt /baihe

[hadoop@node01 ~]$ hdfs dfs -getmerge /baihe/ local_largefile.txt

[hadoop@node01 ~]$ ll
总用量 12
-rw-r--r--. 1 hadoop hadoop 56 5月  30 14:45 local_largefile.txt
-rw-rw-r--. 1 hadoop hadoop 28 5月  27 01:32 world1.txt
-rw-rw-r--. 1 hadoop hadoop 28 5月  27 01:29 world.txt
[hadoop@node01 ~]$ tail -f -n 200 local_largefile.txt
aaa
bbb
ccc
ddd
eee
fff
hhh
aaa
bbb
ccc
ddd
eee
fff
hhh
```

#### 合并HDFS上的小文件
```
[hadoop@node01 ~]$ hdfs dfs -cat /baihe/* | hdfs dfs -appendToFile - /tmp/hdfs_largefile.txt
```

特殊说明：此类型处理方法，数据量非常大的情况下可能不太适合，最好使用MapReduce来合并。

### Hadoop Archive方式
Hadoop Archive或者HAR，是一个高效地将小文件放入HDFS块中的文件存档工具，它能够将多个小文件打包成一个HAR文件，这样在减少namenode内存使用的同时，仍然允许对文件进行透明的访问。


```
[hadoop@node01 ~]$ hadoop archive
archive -archiveName <NAME>.har -p <parent path> [-r <replication factor>]<src>* <dest>
```
- -archiveName <NAME>.har 指定归档后的文件名
- -p <parent path> 被归档文件所在的父目录
- <src>* 归档的目录结构
- <dest> 归档后存在归档文件的目录.
- 可以通过参数 `-D har.block.size` 指定HAR的大小 

#### 创建归档文件
```
hadoop fs -lsr /test/in
drwxr-xr-x   - root supergroup          0 2015-08-26 02:35 /test/in/har
drwxr-xr-x   - root supergroup          0 2015-08-22 12:02 /test/in/mapjoin
-rw-r--r--   1 root supergroup         39 2015-08-22 12:02 /test/in/mapjoin/address.txt
-rw-r--r--   1 root supergroup        129 2015-08-22 12:02 /test/in/mapjoin/company.txt
drwxr-xr-x   - root supergroup          0 2015-08-25 22:27 /test/in/small
-rw-r--r--   1 root supergroup          1 2015-08-25 22:17 /test/in/small/small.1
-rw-r--r--   1 root supergroup          1 2015-08-25 22:17 /test/in/small/small.2
-rw-r--r--   1 root supergroup          1 2015-08-25 22:17 /test/in/small/small.3
-rw-r--r--   1 root supergroup          3 2015-08-25 22:27 /test/in/small/small_data

hadoop archive -archiveName 0825.har -p /test/in/ small mapjoin /test/in/har
```

#### 查看归档文件内容
```
hdfs dfs -lsr /test/in/har/myhar.har
hdfs dfs -lsr har:///test/in/har/myhar.har
```

#### 解压归档文件
```
hdfs dfs -mkdir -p /test/in/har/myhar.har
hdfs dfs -cp har:///test/in/har/myhar.har
```

#### 删除har文件必须使用rmr命令,rm是不行的
```
hdfs dfs -rmr /test/in/har/myhar.har

```

特殊说明：
- 存档文件的源文件及目录都不会自动删除，需要手动删除
- 存档过程实际是一个MapReduce过程，所以需要hadoop的MapReduce支持，以及启动yarm集群
- 存档文件本身不支持压缩
- 存档文件一旦创建便不可修改，要想从中删除或增加文件，必须重新建立存档文件
- 创建存档文件会创建原始文件的副本，所以至少需要有与存档文件容量相同的磁盘空间
- 使用 HAR 作为MR的输入，MR可以访问其中所有的文件。但是由于InputFormat不会意识到这是个归档文件，也就不会有意识的将多个文件划分到单独的Input-Split中，所以依然是按照多个小文件来进行处理，效率依然不高

### Sequence file
sequence file 由一系列的二进制key/value 组成，如果为key 小文件名，value 为文件内容，则可以将大批小文件合并成一个大文件。

### CombineFileInputFormat
CombineFileInputFormat 是一种新的inputformat，用于将多个文件合并成一个单独的split，另外，它会考虑数据的存储位置。
