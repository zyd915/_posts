---
title: centos 下编译安装php7.3的mongodb扩展
date: 2020-01-08 20:38:15
updated: 2020-01-09 00:23:53
tags: 
    - php
    - centos7
    - mongodb
categories: php
toc: true
excerpt: 本文是基于PHP7.3编译安装版本，进行的 mongodb 的编译扩展安装方法整理。
---

本文是基于PHP7.3编译安装版本，进行的 mongodb 的编译扩展安装方法整理。

### 下载 mongodb 扩展包以及解压
```
wget  https://pecl.php.net/get/mongodb-1.5.5.tgz
tar -xzvf mongodb-1.5.5.tgz -C /opt/module
```

### 进入解压后目录，编译安装
```
cd /opt/module/mongodb-1.5.5

sudo find / -name phpize
/usr/local/php/bin/phpize
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
php -i | grep php.ini
vim /usr/local/php/etc/php.ini

extension=mongodb.so
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

