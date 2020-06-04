---
title: 'django2.2 和 PyMySQL版本兼容问题'
date: 2020-05-05 00:04:00
updated: 2020-05-05 00:05:05
tags: 
    - python
    - PyMySQL
    - Django
categories: python
toc: true
excerpt: '错误信息为：`django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.13 or newer is required; you have 0.9.3.`解决方案'
---

错误信息为`django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.13 or newer is required; you have 0.9.3.`

### 错误原因：
因为Django连接MySQL时默认使用MySQLdb驱动，但MySQLdb不支持Python3，因此这里将MySQL驱动设置为pymysql。由此产生的版本兼容问题。

### pymysql安装方法：

```python
#安装pymysql
pip install pymysql

#__init__.py
import pymysql
pymysql.install_as_MySQLdb()
```

### 解决办法：

#### 1. django降到2.1.4版本

#### 2. 修复源码

2.1 找到Python环境下 django包，并进入到backends下的mysql文件夹

```
#  使用此命令可以看到对应的文件夹目录
➜  daliyfresh pip install pymysql
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/
Requirement already satisfied: pymysql in /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages (0.9.3)
```

2.2 找到base.py文件，注释掉 base.py 中如下部分（35/36行）
```python
if version < (1, 3, 3):
     raise ImproperlyConfigured("mysqlclient 1.3.3 or newer is required; you have %s" % Database.__version__)
```

2.3 此时仍然报错，找到operations.py文件，将decode改为encode
```
AttributeError: ‘str’ object has no attribute ‘decode’
```

解决办法：

```python
#linux vim 查找快捷键：？decode
if query is not None:
    query = query.decode(errors='replace')
return query
#改为
if query is not None:
    query = query.encode(errors='replace')
return query
```

### 测试，执行数据迁移
```python
python manage.py makemigrations
python manage.py migrate
```
![](https://static.studytime.xin/image/articles/20200130191134.png)
