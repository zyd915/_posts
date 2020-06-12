---
title: centos7 下python2与python3共存
permalink: python-centos7-install
date: 2020-05-04 20:03:35
updated: 2020-05-04 20:59:49
tags: 
    - python
categories: python
toc: true
excerpt: centos7 下python2与python3共存
---

### 查看python的版本号

```
[root@application-server baihe]# python -V
Python 2.7.5
[root@application-server baihe]# which python
/usr/bin/python
[root@application-server baihe]# ll  /usr/bin/python
lrwxrwxrwx. 1 root root 7 1月   7 21:06 /usr/bin/python -> python2
```

特殊说明：从以上操作可以看到，系统默认安装版本为2.7.5。此处不去动它。
目前还有很多系统和软件使用的还是python2。例如yum 安装和更新等。

### 安装python3的相关依赖库
```
yum -y install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel wget
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc
yum -y install libffi-devel 
```

### 下载python3、解压
```
wget https://www.python.org/ftp/python/3.7.6/Python-3.7.6.tgz
tar -zxvf Python-3.7.6.tgz
```

### 编译安装
```
cd Python-3.7.6

./configure --prefix=/usr/local/python3

make && make install
```

### 创建软连接
```
ln -s /usr/local/python3/bin/python3 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3
```

### 验证是否成功
```
[root@application-server Python-3.7.6]# python3 -V
Python 3.7.6

[root@application-server Python-3.7.6]# pip3 -V
pip 19.2.3 from /usr/local/python3/lib/python3.7/site-packages/pip (python 3.7)
```