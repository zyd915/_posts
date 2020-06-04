---
title: MySQL性能优化之max_connections配置参数浅析
date: 2020-05-04 00:46:26
updated: 2020-05-04 20:01:49
tags: 
    - mysql
    - 数据库
categories: data
toc: true
excerpt: 本文主要整理了 MySQL 参数中 max_connections 配置参数的浅析，该参数在 MySQL 中是用来设置最大连接（用户）数。每个连接 MySQL 的用户均作为一个连接，max_connections 的默认值为100，本文将讲解此参数的详细作用与性能影响。
---

本文主要整理了 MySQL 参数中 max_connections 配置参数的浅析，该参数在 MySQL 中是用来设置最大连接（用户）数。每个连接 MySQL 的用户均作为一个连接，max_connections 的默认值为100，本文将讲解此参数的详细作用与性能影响。

### 与max_connections有关的特性
MySQL 无论如何都会保留一个用于管理员（SUPER）登陆的连接，用于管理员连接数据库进行维护操作，即使当前连接数已经达到了 max_connections，因此 MySQL 的实际最大可连接数为 max_connections+1。

这个参数实际起作用的最大值（实际最大可连接数）为 16384，即该参数最大值不能超过 16384，即使超过也以 16384 为准。增加 max_connections 参数的值，不会占用太多系统资源。系统资源（CPU、内存）的占用主要取决于查询的密度、效率等，该参数设置过小的最明显特征是出现 ”Too many connections” 错误。

### 如何查看当前 MySQL 的max_connections的值：
```
show variables like "max_connections";

显示的结果如下格式

+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 100   |
+-----------------+-------+
```

### 修改max_connections的值
```
set global max_connections = 1000;
```
此设置方法会立即生效，但是当mysql重启时这个设置会失效，更好的办法是修改 mysql 的ini配置文件 my.ini。


```
vim /etc/my.ini

# mysqld 下修改或添加此配置
[mysqld]
max_connections=1000

```
这样修改之后，即便重启mysql也会默认载入这个配置。

总体来说，该参数在服务器资源够用的情况下应该尽量设置大，以满足多个客户端同时连接的需求。否则将会出现类似”Too many connections”的错误。
一般情况下根据同时在线人数设置一个比较综合的数字，我们设置的是10000。
