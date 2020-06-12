---
title: Django 项目如何通过加载不同env文件来区分不同环境
permalink: python-django-env
date: 2020-05-05 00:04:00
updated: 2020-05-05 00:05:05
tags: 
    - python
    - env
categories: python
toc: true
excerpt: 本文主要是整理如何在 django  项目中解决不同环境下加载不同 .env  环境配置文件的方案。主要内容包含 django-environ  的安装使用，以及 django-crontab 脚本环境下的配置使用。
---


本文主要是整理如何在 django  项目中解决不同环境下加载不同 .env  环境配置文件的方案。主要内容包含 django-environ  的安装使用，以及 django-crontab 脚本环境下的配置使用。

### django-environ 的使用

#### 安装
[官方 django-environ  github链接地址](https://django-environ.readthedocs.io/en/latest/)

`pip install django-environ`

#### 使用

- 创建 .env文件

```
# 项目目录中
touch .env

# 增加环境变量
DEBUG=True
SECRET_KEY=your-secret-key
DATABASE_URL=psql://urser:un-githubbedpassword@127.0.0.1:8458/database
SQLITE_URL=sqlite:///my-local-sqlite.db
CACHE_URL=memcache://127.0.0.1:11211,127.0.0.1:11212,127.0.0.1:11213
REDIS_URL=rediscache://127.0.0.1:6379/1?client_class=django_redis.client.DefaultClient&password=ungithubbed-secret
```

- 与`settings.py`一起使用。为了方便使用目前使用gitlab为内网环境，故而`.env`不需要加入`.gitignore`，实现文件忽略上传。

```
import environ

# initialize env
env = environ.Env(
    # set casting, default value
    DEBUG=(bool, False)
)
# reading .env file
environ.Env.read_env()

# False if not in os.environ
DEBUG = env('DEBUG')

# Raises django's ImproperlyConfigured exception if SECRET_KEY not in os.environ
SECRET_KEY = env('SECRET_KEY')

# Parse database connection url strings like psql://user:pass@127.0.0.1:8458/db
DATABASES = {
    # read os.environ['DATABASE_URL'] and raises ImproperlyConfigured exception if not found
    'default': env.db(),
    # read os.environ['SQLITE_URL']
    'extra': env.db('SQLITE_URL', default='sqlite:////tmp/my-tmp-sqlite.db')
}

CACHES = {
    # read os.environ['CACHE_URL'] and raises ImproperlyConfigured exception if not found
    'default': env.cache(),
    # read os.environ['REDIS_URL']
    'redis': env.cache('REDIS_URL')
}
```

### 如何实现不同环境下加载不同环境变量

在只使用一个 settings.py 文件情况下，可以只使用一个环境变量，通过此环境变量来读取不同环境的 .env 文件以区分不同环境。
这个环境变量就没法写在 .env 文件了，必须手动指定。假设我们这个环境变量叫PROJECT_ENV。那么我们在命令行执行任何命令的时候，在前面加上PROJECT_ENV=xxx来指定环境变量。
当然，也可以export来指定这个环境变量，也可以修改shell配置文件来指定这个环境变量，甚至supervisord和uwsgi的配置文件也可以指定环境变量。

#### 执行方法
```
PROJECT_ENV=local python manage.py runserver
```

#### 多环境.env文件设置
假设目前存在 local 以及 product 两种环境，则在 envs 下新建 .env.local 以及 .env.product 文件。

#### settings.py 代码兼容

```
import os
import environ
import logging.config

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

env = environ.Env(
    # set casting, default value
    DEBUG=(bool, False)
)

"""
通过指定环境遍历加载不同env文件
PROJECT_ENV=production python manage.py crontab add
默认场景下可不指定，则加载local文件
"""
env_name = env.str('PROJECT_ENV', 'local')
# reading .env file
env.read_env('envs/.env.%s' % env_name)
```

### 如何在 django-crontab 脚本环境下加载不同配置

django-crontab 的使用可以参考 [django-crontab实现服务端的定时任务](https://www.studytime.xin/python/2020/02/11/python-django-crontab.html)

#### 代码调整
在 settings.py 文件中，django-crontab 配置任务 CRONJOBS 下增加
`CRONTAB_COMMAND_PREFIX = 'PROJECT_ENV=' + env_name` 参数指定

```
CRONJOBS = [
    ("*/5 * * * *", "analysis.cron.test_cron", ">>" + LOG_DIR + "log/fba_claim_shipment.log"),
]

CRONTAB_COMMAND_PREFIX = 'PROJECT_ENV=' + env_name
```

#### 执行方法
```
PROJECT_ENV=local python manage.py crontab add
```
