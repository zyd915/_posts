---
title: CentOS 下 mysql 忘记 root 密码的处理方法
date: 2020-01-07 20:00:50
updated: 2020-01-07 21:23:50
permalink: centos-mysql-forgot-password
tags: 
    - mysql
categories: mysql
toc: true
excerpt: 本文整理记录 CentOS 下 mysql 忘记 root 密码的处理方法。
---

本文整理记录 CentOS 下 mysql 忘记 root 密码的处理方法。

### 修改 mysql 配置文件，在 `[mysqld]` 中添加 `skip-grant-tables`，跳过密码验证。

```
vim /etc/my.cnf

[mysqld]
skip-grant-tables
```

### 修改配置文件后，重启mysql
`service mysql restart`
 
### 重新登录，此时会跳过密码验证
```
mysql -uroot -p (直接点击回车，密码为空)
```

### 选择 mysql 数据库
```
use mysql;
```

### 修改root密码
```
update user set authentication_string=password('!Qaz123456') where user='root';
```
 
### 刷新权限
```
flush privileges;
exit;
```

### 删除 skip-grant-tables 配置文件
 
### 重启mysql
```
service mysql restart
```
