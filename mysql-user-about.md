---
title: MySQL 数据库用户权限相关操作
date: 2020-05-04 19:33:25
updated: 2020-05-04 19:59:49
tags: 
    - mysql
    - 数据库
categories: data
toc: true
excerpt: 本文主要整理了用户创建、删除、以及用户权限赋予、权限删除等 mysql 用户权限相关命令，以备记录自己使用。
---

### 创建用户
创建 user1 用户，密码为 UserPass123% ，且使用localhost本地连接
```
mysql> CREATE USER 'user1'@'localhost' IDENTIFIED BY 'UserPass123%';
Query OK, 0 rows affected (0.01 sec)
```
如果密码过于简单则会提示 `ERROR 1819 (HY000): Your password does not satisfy the current policy requirements` ，使用大小写+数字+特字符即可。

### 删除用户
```
drop user 'user1'@'%';
flush privileges;
```

### 重命名用户
```
rename user 'user1'@'%' to 'user1'@'%';
```

### 创建数据库
```
mysql> CREATE DATABASE test1;
Query OK, 1 row affected (0.00 sec)
```

### 赋予用户权限命令
MySQL 赋予用户权限命令的简单格式可概括为：`grant 权限 on 数据库对象 to 用户@访问权限主机 identified by 密码`

### grant 普通数据用户，查询、插入、更新、删除 数据库中所有表数据的权利。 

```
# 赋予查询权限
grant select on test1.* to user1@'%' identified by 'UserPass123%';
# 赋予插入权限
grant insert on test1.* to user1@'%' identified by 'UserPass123%';
# 赋予更新权限
grant update on test1.* to user1@'%' identified by 'UserPass123%';
# 赋予删除权限
grant delete on test1.* to user1@'%' identified by 'UserPass123%';

# 赋予所有权限
grant select, insert, update, delete on test1.* to user1@'%' identified by 'UserPass123%';
```

特殊说明：最后一天权限百分号%代表远程访问权限。


### 如何指定访问数据库主机权限
```
# 赋予本机访问权限
grant select on test1.* to user1@'%' identified by 'UserPass123%';

# 赋予任意主机访问权限
grant select on test1.* to user1@'%' identified by 'UserPass123%';

# 赋予 ip为172.16.225.129 的主机访问权限
grant select on test1.* to user1@'172.16.225.129' identified by 'UserPass123%';

# 赋予 ip以172.16.225开头 的主机访问权限
grant select on test1.* to user1@'172.16.225.*' identified by 'UserPass123%';
```

### grant 普通 DBA 管理某个 MySQL 数据库的权限
```
grant all privileges on test1.* to user1@localhost identified by 'UserPass123%';
```

### grant 高级 DBA 管理 MySQL 中所有数据库的权限
```
grant all on *.* to dba@'localhost' 
```

### MySQL grant 权限，分别可以作用在多个层次上 

#### grant 作用在整个 MySQL 服务器上：

```
grant select on *.* to dba@localhost ; -- dba 可以查询 MySQL 中所有数据库中的表。 
grant all    on *.* to dba@localhost ; -- dba 可以管理 MySQL 中的所有数据库 
```

#### grant 作用在单个数据库上：
```
grant select on testdb.* to dba@localhost ; -- dba 可以查询 testdb 中的表。
```

#### grant 作用在单个数据表上：  
```
grant select, insert, update, delete on testdb.orders to dba@localhost ;  
```

#### grant 作用在表中的列上：  
```
grant select(id, se, rank) on testdb.apache_log to dba@localhost ; 
```

#### grant 作用在存储过程、函数上：  
```
grant execute on procedure testdb.pr_add to 'dba'@'localhost'  
grant execute on function  testdb.fn_add to 'dba'@'localhost' 
```

### 查看 MySQL 用户权限 查看用户权限： 
```
# 查看当前 MySQL 用户权限 
show grants; 

# 查看其他 MySQL 用户权限
show grants for dba@localhost; 
```

### 撤销已经赋予给 MySQL 用户权限的权限

revoke 跟 grant 的语法差不多，只需要把关键字 “to” 换成 “from” 即可
```
grant all on *.* to   dba@localhost; 
revoke all on *.* from dba@localhost; 
```

### 如何刷新权限
```
flush privileges;
```

### MySQL grant、revoke 用户权限注意事项 

1. grant, revoke 用户权限后，该用户只有重新连接 MySQL 数据库，权限才能生效。 
2. 如果想让授权的用户，也可以将这些权限 grant 给其他用户，需要选项 “grant option“ 
grant select on testdb.* to dba@localhost with grant option; 这个特性一般用不到。实际中，数据库权限最好由 DBA 来统一管理。


### 知识扩展

#### mysql 权限列表

| 权限 | 作用范围 | 作用 |
| --- | --- | --- |
all	| 服务器|所有权限|
select| 	表、列	|选择行|
insert|	表、列	|插入行|
update|	表、列	|更新行|
delete|	表	|删除行|
create|	数据库、表、索引|	创建|
drop|	数据库、表、视图	|删除|
reload|	服务器	|允许使用flush语句|
shutdown| 服务器 |	关闭服务|
process| 服务器 |	查看线程信息|
file|服务器|文件操作|
grant option|	数据库、表、存储过程|	授权|
references|	数据库、表|	外键约束的父表|
index|	表	|创建/删除索引|
alter|	表|	修改表结构|
show databases|	服务器	|查看数据库名称|
super|	服务器|	超级权限|
create temporary tables|	表	|创建临时表|
lock tables|	数据库|	锁表|
execute|	存储过程	|执行|
replication client|	服务器|	允许查看主/从/二进制日志状态|
replication slave	|服务器	|主从复制|
create view|	视图|	创建视图|
show view|	视图|	查看视图|
create routine	|存储过程|	创建存储过程|
alter routine	|存储过程	|修改/删除存储过程|
create user|	服务器|	创建用户|
event	|数据库	|创建/更改/删除/查看事件|
trigger	|表	|触发器|
create tablespace	|服务器	|创建/更改/删除表空间/日志文件|
proxy|	服务器	|代理成为其它用户|
usage|	服务器|	没有权限|