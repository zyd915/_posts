---
title: Mac Os 下MySQL忘记密码后如何重置密码
date: 2020-05-04 00:46:26
updated: 2020-05-04 20:01:50
tags: 
    - mysql
    - 数据库
categories: data
toc: true
excerpt: Mac Os 下MySQL忘记密码后如何重置密码
---

Mac Os 下MySQL忘记密码后如何重置密码

### 停止 mysql server
通常是在 '系统偏好设置' > MySQL > 'Stop MySQL Server'

### 打开命令行输入：`sudo /usr/local/mysql/bin/mysqld_safe --skip-grant-tables`

### 打开另一个新终端：`sudo /usr/local/mysql/bin/mysql -u root`

### 重置密码
```
UPDATE mysql.user SET authentication_string=PASSWORD('新密码') WHERE User='root';
FLUSH PRIVILEGES;
```

### 重启MySQL即可

### 特殊说明
以上方法针对 mysql V5.7.9, 旧版的mysql请使用：
`UPDATE mysql.user SET Password=PASSWORD('新密码') WHERE User='root';`