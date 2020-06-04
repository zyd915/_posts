---
title: HDFS的shell常用命令操作
date: 2020-05-27 02:10:33
updated: 2020-05-27 02:10:36
tags: 
    - 大数据
    - hadoop
categories: hadoop
toc: true
excerpt: hdfs的shell命令操作,常用风格有两种分别为hadoop fs开头和hdfs dfs开头，两者都可以使用，效果相同，但建议使用hdfs dfs，因为hadoop fs为老版本用法，兼容保留。
---


HDFS的命令有两种风格：
- hadoop fs开头
- hdfs dfs开头
两种命令都可以使用，效果相同，建议使用hdfs dfs，因为hadoop fs为老版本用法，兼容保留。

### 启动集群
```
start-dfs.sh
start-yarn.sh
```

### 帮助命令
```
[hadoop@node01 ~]$ hdfs dfs -help ls
```

### 查看HDFS系统根目录
```
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 2 items
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
```

### 创建文件夹
```
[hadoop@node01 ~]$ hdfs dfs -mkdir /a
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 4 items
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /a
-rw-r--r--   3 hadoop    supergroup          0 2020-05-26 23:45 /edits.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
```

### 递归创建文件夹
```
[hadoop@node01 ~]$ hdfs dfs -mkdir -p  /aa/bb/cc
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 5 items
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /a
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /aa
-rw-r--r--   3 hadoop    supergroup          0 2020-05-26 23:45 /edits.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
```

### 查看hsdf系统根目录下的所有文件包括子文件夹里面的文件
```
[hadoop@node01 ~]$ hdfs dfs -ls -R /aa
drwxr-xr-x   - hadoop supergroup          0 2020-05-27 01:26 /aa/bb
drwxr-xr-x   - hadoop supergroup          0 2020-05-27 01:26 /aa/bb/cc
```

### 创建文件
```
[hadoop@node01 ~]$ hdfs dfs -touchz /edits.txt
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 3 items
-rw-r--r--   3 hadoop    supergroup          0 2020-05-26 23:45 /edits.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /use
```

### 从本地文件中读取文件追加到hdfs

```
vim words.txt

aaa
bbb
ccc
ddd
eee
fff
hhh

[hadoop@node01 ~]$ hdfs dfs -appendToFile world.txt /edits.txt
```

### 查看hdfs文件内容
```
[hadoop@node01 ~]$ hdfs dfs -cat /edits.txt
aaa
bbb
ccc
ddd
eee
fff
hhh
```

### 查看hdfs文件内容tail
```
[hadoop@node01 ~]$ hdfs dfs -tail /edits.txt
aaa
bbb
ccc
ddd
eee
fff
hhh
```

### 从本地文件上传文件到hdfs
```
[hadoop@node01 ~]$ cp world.txt world1.txt
[hadoop@node01 ~]$ hdfs dfs -put world1.txt /
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 6 items
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /a
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /aa
-rw-r--r--   3 hadoop    supergroup         28 2020-05-27 01:30 /edits.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
-rw-r--r--   3 hadoop    supergroup         28 2020-05-27 01:32 /world1.txt


# 类似命令
hdfs dfs -copyFromLocal world1.txt /
hdfs dfs -moveFromLocal world1.txt /
```

### 从hdfs下载文件到本地
```
hdfs dfs -get /world1.txt /
hdfs dfs -copyToLocal /world1.txt /
```

### hdfs 中删除文件,默认放入回收站中
```
[hadoop@node01 ~]$ hdfs dfs -rm /world1.txt
20/05/27 01:34:00 INFO fs.TrashPolicyDefault: Moved: 'hdfs://node01:9000/world1.txt' to trash at: hdfs://node01:9000/user/hadoop/.Trash/Current/world1.txt
```

### 递归删除文件
```
[hadoop@node01 ~]$ hdfs dfs -rm -r /aa/bb
20/05/27 01:39:12 INFO fs.TrashPolicyDefault: Moved: 'hdfs://node01:9000/aa/bb' to trash at: hdfs://node01:9000/user/hadoop/.Trash/Current/aa/bb
```

### hdfs中移动或重命名文件
```
[hadoop@node01 ~]$ hdfs dfs -mv /edits.txt /edits1.txt
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 5 items
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /a
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /aa
-rw-r--r--   3 hadoop    supergroup         28 2020-05-27 01:30 /edits1.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
```

### 移动或重命名文件
```
[hadoop@node01 ~]$ hdfs dfs -mv /edits.txt /edits1.txt
[hadoop@node01 ~]$ hdfs dfs -ls /
Found 5 items
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /a
drwxr-xr-x   - hadoop    supergroup          0 2020-05-27 01:26 /aa
-rw-r--r--   3 hadoop    supergroup         28 2020-05-27 01:30 /edits1.txt
drwx-wx-wx   - hadoop    supergroup          0 2020-05-23 23:55 /tmp
drwxr-xr-x   - anonymous supergroup          0 2020-05-24 19:23 /user
```


### 拷贝文件
```
[hadoop@node01 ~]$ hdfs dfs -cp /edits1.txt /a
[hadoop@node01 ~]$ hdfs dfs -ls /a
Found 1 items
-rw-r--r--   3 hadoop supergroup         28 2020-05-27 01:38 /a/edits1.txt
```

### 列出本地文件的内容,默认hdfs内容
```
[hadoop@node01 ~]$ hdfs dfs -ls file:///home/hadoop/
Found 8 items
-rw-------   1 hadoop hadoop       4670 2020-05-24 12:01 file:///home/hadoop/.bash_history
-rw-r--r--   1 hadoop hadoop         18 2017-08-03 05:11 file:///home/hadoop/.bash_logout
-rw-r--r--   1 hadoop hadoop        406 2020-05-12 00:06 file:///home/hadoop/.bash_profile
-rw-r--r--   1 hadoop hadoop        231 2017-08-03 05:11 file:///home/hadoop/.bashrc
drwx------   - hadoop hadoop         80 2020-05-14 12:33 file:///home/hadoop/.ssh
-rw-------   1 hadoop hadoop       8068 2020-05-27 01:29 file:///home/hadoop/.viminfo
-rw-rw-r--   1 hadoop hadoop         28 2020-05-27 01:29 file:///home/hadoop/world.txt
-rw-rw-r--   1 hadoop hadoop         28 2020-05-27 01:32 file:///home/hadoop/world1.txt
```

### 查找文件
```
[hadoop@node01 ~]$ hadoop fs -find / -name edits1.txt
/a/edits1.txt
/edits1.txt
```

### 显示文件大小
```
[hadoop@node01 ~]$ hdfs dfs -du -h /
28       84     /a
0        0      /aa
28       84     /edits1.txt
```

### 改变文件所属组
```
hdfs dfs -chgrp [-R] GROUP
```


### 改变文件的权限
```
hdfs dfs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI …]
```

### 改变文件的拥有者
```
hdfs dfs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

[HDFS权限管理用户指南](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_permissions_guide.html)


### 清空回收站
```
hdfs dfs -expunge
```

### 查看集群运行状态
```
[hadoop@node01 ~]$ hdfs dfsadmin -report
Configured Capacity: 151298519040 (140.91 GB)
Present Capacity: 142971727989 (133.15 GB)
DFS Remaining: 141700521984 (131.97 GB)
DFS Used: 1271206005 (1.18 GB)
DFS Used%: 0.89%
Under replicated blocks: 45
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (3):

Name: 172.16.87.131:50010 (node01)
Hostname: node01
Decommission Status : Normal
Configured Capacity: 50432839680 (46.97 GB)
DFS Used: 423735335 (404.11 MB)
Non DFS Used: 2115256281 (1.97 GB)
DFS Remaining: 47893848064 (44.60 GB)
DFS Used%: 0.84%
DFS Remaining%: 94.97%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Wed May 27 01:42:18 CST 2020


Name: 172.16.87.132:50010 (node02)
Hostname: node02
Decommission Status : Normal
Configured Capacity: 50432839680 (46.97 GB)
DFS Used: 423735335 (404.11 MB)
Non DFS Used: 3714293721 (3.46 GB)
DFS Remaining: 46294810624 (43.12 GB)
DFS Used%: 0.84%
DFS Remaining%: 91.79%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Wed May 27 01:42:19 CST 2020


Name: 172.16.87.133:50010 (node03)
Hostname: node03
Decommission Status : Normal
Configured Capacity: 50432839680 (46.97 GB)
DFS Used: 423735335 (404.11 MB)
Non DFS Used: 2497241049 (2.33 GB)
DFS Remaining: 47511863296 (44.25 GB)
DFS Used%: 0.84%
DFS Remaining%: 94.21%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Wed May 27 01:42:18 CST 2020
```
