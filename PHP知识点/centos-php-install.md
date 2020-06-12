---
title: centos 下编译安装PHP7.3搭建PHP运行环境
permalink: centos-php-install
date: 2020-01-07 00:23:49
updated: 2020-01-07 00:23:50
tags: 
    - php
    - centos7
categories: php
toc: true
excerpt: 研发过程中，发现原有安装的php7.2.3已经无法兼容支持laravel6之后的版本，就针对原有php的编译安装文档进行了升级支持。以便记录使用同事也分享出来，希望解决大家的一些问题。
---

研发过程中，发现原有安装的php7.2.3已经无法兼容支持laravel6之后的版本，就针对原有php的编译安装文档进行了升级支持。以便记录使用同事也分享出来，希望解决大家的一些问题。

### 新建存储目录

```
# 文件安装包
mkdir /opt/software

# 文件加压目录
mkdir /opt/module

# 新建日志存放目录
mkdir -p /data/log
```

### 新增php-fpm用户和用户组
```
useradd -r -s /sbin/nologin php-fpm
```


### 下载 php 软件安装包并解压
```
wget https://www.php.net/distributions/php-7.3.17.tar.gz
tar -zxvf php-7.3.17.tar.gz -C /opt/module/
cd /opt/module/
```

### 安装依赖扩展库
```
yum -y install libxml2 libxml2-devel openssl openssl-devel curl-devel libjpeg-devel libpng-devel freetype-devel libmcrypt-devel libxslt libicu-devel libxslt-devel bzip2-devel
```

### 开始编译


```
./configure --prefix=/usr/local/php7.3 --with-config-file-path=/usr/local/php7.3/etc --with-fpm-user=php-fpm --with-fpm-group=php-fpm --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-mysqli --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enable-zip --enable-fpm
```

### 编译时常见错误问题

#### 问题一：`configure: error: libxml2 not found. Please check your libxml2 installation.`

解决办法： 安装扩展`yum install -y libxml2-devel`即可解决。

#### 问题二：`checking for libzip... configure: error: system libzip must be upgraded to version >= 0.11` libzip版本过低。

解决办法：
```
# 删除旧版本
yum remove -y libzip

# 下载编译安装
wget https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install
```

#### 问题三：`configure: error: off_t undefined; check your library configuration` off_t undefined 报错

解决办法：
```
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64'>>/etc/ld.so.conf

# 更新配置
ldconfig -v
```

### 编译完成后进行安装
`make && make instal`

#### 常见报错问题：`usr/local/include/zip.h:59:21: fatal error: zipconf.h: No such file or directory`

解决办法： `cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h`

### 建立软链进行映射
```
ln -s /usr/local/php7.3/ /usr/local/php
ln -s /usr/local/php/bin/php /usr/local/bin
ln -s /usr/local/php/sbin/php-fpm /usr/local/sbin
```

### 对php-fpm运行用户进行设置
```
cd /opt/module/php-7.3.17/

#默认安装好之后，你会发现/usr/local/php/etc下面没有php.ini文件，这个去哪里要呢？在php7的源码安装包里
cp ./php.ini-development ./php.ini-production /usr/local/php/etc
cp /usr/local/php/etc/php.ini-development /usr/local/php/etc/php.ini

cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```

### 建立软链进行映射
```
ln -s /usr/local/php/etc/php.ini /usr/local/etc/
ln -s /usr/local/php/etc/php-fpm.conf /usr/local/etc/
ln -s /usr/local/php/etc/php-fpm.d/www.conf /usr/local/etc/
```

### 配置环境变量，加入全局命令
```
vim /etc/profile

export PATH=/usr/local/php/bin:$PATH

source /etc/profile
```

### 启动php-fpm 服务
```
/usr/local/sbin/php-fpm
```

### 查看是否启动
```
netstat -lnt | grep 9000
```

### 查看php配置文件所在目录
```
php -i|grep php.ini
```


### 杀死php-fpm
```
killall php-fpm
```

### 将php-fpm加入systemtl服务
```
cd /opt/module/php-7.3.17/sapi/fpm
cp php-fpm.service /usr/lib/systemd/system/
```

### 使用systemtl服务管理php-fpm 

```
# 启动
systemctl start php-fpm
# 状态
systemctl status php-fpm
# 重启
systemctl restart php-fpm
# 停止
systemctl stop php-fpm
```
