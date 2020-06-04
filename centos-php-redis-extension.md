---
title: centos 下编译安装php7.3的redis扩展
date: 2020-01-07 00:23:52
updated: 2020-01-07 00:23:53
tags: 
    - php
    - centos7
    - redis
categories: php
toc: true
excerpt: 本文是基于PHP7.3编译安装版本，进行的redis的编译扩展安装方法整理。
---

本文是基于 [centos 下编译安装PHP7.3搭建PHP运行环境](https://www.studytime.xin/article/centos-php-install.html) 编译安装版本，进行的redis的编译扩展安装方法整理。

### 下载redis扩展包以及解压
```
wget http://pecl.php.net/get/redis-4.2.0.tgz
tar -xzvf redis-4.2.0.tgz -C /opt/module
```

### 进入解压后目录，编译安装
```
cd /opt/module/redis-4.2.0
/usr/local/php/bin/phpize
```

### 常见错误
```
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.

# 安装autoconf
yum install autoconf
```

### 查找php-config
```
find / -name php-config
```

### 编译安装
```
./configure --with-php-config=/usr/local/php/bin/php-config
make & make install
```

### 配置php.ini，添加redis.so扩展
```
vim /usr/local/php/etc/php.ini

extension=redis.so
```
### 重启php-fpm
```
killall php-fpm
/usr/local/sbin/php-fpm
```

### 查看php扩展
```
php -m
```
