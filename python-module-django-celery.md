---
title: python 之 Django与Celery的安装使用
date: 2020-05-04 22:13:35
updated: 2020-05-04 22:13:40
tags: 
    - python
    - pycharm
    - celery
    - django
categories: python
toc: true
excerpt: Celery 是一个强大的 分布式任务队列 的 异步处理框架，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）。
---

Celery 是一个强大的 分布式任务队列 的 异步处理框架，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）。

### Celery的核心模块和架构
Celery的架构由三部分组成，消息中间件（message broker），任务执行单元（worker）和任务执行结果存储（task result store）组成。

![](https://static.studytime.xin/image/articles/20200329230238.png)

Celery 主要包含以下几个模块：
#### 任务模块 Task 
包含异步任务和定时任务。其中，异步任务通常在业务逻辑中被触发并发往任务队列，而定时任务由 Celery Beat 进程周期性地将任务发往任务队列。

#### 消息中间件 Broker 
Broker，即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列。Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。

#### 任务执行单元 Worker 
Worker 是执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。
#### Beat

定时任务调度器，根据配置定时将任务发送给Broler
#### 任务结果存储 Backend
Backend 用于存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 RabbitMQ, redis 和 MongoDB 等。

综上celery总结如下：celery 是一个处理大量消息的分布式系统，能异步任务、定时任务，使用场景一般用于耗时操作的多任务或者定时性的任务。

### celery 分布式脚本架构图

![](https://static.studytime.xin/image/articles/celery分布式脚本架构图.png)


### 常见使用场景

#### 使用Celery实现异步任务
1. 创建Celery实例
2. 启动Celery Worker，通过delay()或者apply_async()将任务发布到broker
3. 应用程序调用异步任务
4. 存储结果
Celery Beat: 任务调度器，Beat进程会读取配置文件的内容，周期性的将配置中到期需要执行的任务发送给任务队列

#### 使用Celery定时任务
1. 创建Celery实例
2. 配置文件中配置任务，发送任务celery -A xxx beat
3. 启动Celery Worker celery -A xxx worker -l info -P eventlet
4. 存储结果

### celery安装与使用

#### 版本选择

- celery 3.1.26.post2
- django-celery 3.3.0
- django-redis 4.10.0
- redis 2.10.6

特殊说明：redis版本过高会报错，具体错误信息为：
```
return iter(x.items())
AttributeError: 'str' object has no attribute 'items'
```

#### 安装命令
因为是集成在django项目中，所以需要往外安装django-celery，当然也可以不安装。

```
pip3 install celery==3.1.26.post2
pip3 install django-redis==4.11.0
pip3 install django-celery==3.3.1
pip3 install redis==2.10.6
```

#### django 进行注册
在django的settings文件中将djcelery进行注册：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    #注册
    'djcelery',  
]
```

#### 代码配置目录

```
任务所在目录
    ├── celery_tasss # celery包 如果celery_task只是建了普通文件夹__init__可以没有,如果是包一定要有
    │   ├── __init__.py # 包文件 看情况要不要存在
    │   ├── celery.py   # celery连接和配置相关文件，且名字必须交celery.py,其实也不是必须的不然你指令可能要修改
    │   └── tasks.py    # 所有任务函数


├── celery_tasks
│   ├── celeryconfig.py  # celery 实例
│   ├── celery.py       # celery 配置相关文件
│   ├── __init__.py     # 包文件，看情况要不要存在
│   └── tasks.py        # 所有任务函数
```

#### 配置 celeryconfig.py
```
from kombu import Queue, Exchange

BROKER_URL = 'redis://127.0.0.1:6379/7'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/8'

CELERY_IMPORTS = (
    'celery_tasks.tasks',
)

# 任务序列化和反序列化    pickle
CELERY_TASK_SERIALIZER = 'json'

# 结果序列化格式，默认为pickle
CELERY_RESULT_SERIALIZER = 'json'

# 指定接受的内容类型(默认为允许所有格式)   ['pickle', 'json', 'msgpack', 'yaml']
CELERY_ACCEPT_CONTENT = ['json']

# 启动时区设置
CELERY_ENABLE_UTC = True
# 设置时区
CELERY_TIMEZONE = 'Asia/Shanghai'

# 任务结果过期时间
CELERY_TASK_RESULT_EXPIRES = 60 * 60 * 24

# 任务过期时间，600s后结果结束
# CELERY_TASK_TIME_LIMIT= 10 * 60

# 并发的worker数量，也是命令行-c指定的数目，worker数量不是越多越好，保证任务不堆积，加上一些新增任务的预留就可以了
CELERYD_CONCURRENCY = 10

# celery worker 每次去BROKER中预取任务的数量
CELERYD_PREFETCH_MULTIPLIER = 4

# 每个worker最多执行1000个任务就会被销毁，可防止内存泄露
CELERYD_MAX_TASKS_PER_CHILD = 1000

# 任务过期时间，celery任务执行结果的超时时间
CELERY_TASK_RESULT_EXPIRES = 24 * 60 * 60

# Worker在任务执行完后才向Broker发送acks，告诉队列这个任务已经处理了，可靠性较强，但也可能出现重复执行
CELERY_ACKS_LATE = True

# 设置默认的队列名称，未指定的情况下都会放入默认队列中
CELERY_DEFAULT_QUEUE = "default"

# 再不关注结果的情况下，可以设置成True，忽略结果上传broker
# CELERY_IGNORE_RESULT = True


CELERY_QUEUES = (
    Queue('default', exchange=Exchange('default'), routing_key='default'),
    Queue('for_email', exchange=Exchange('for_email'), routing_key='for_email'),
   
)

CELERY_ROUTES = {
    'celery_tasks.tasks.send_mail_task': {'queue': 'for_email', 'routing_key': 'for_email'},
}

```


#### celery.py
```
from celery import Celery
from celery_tasks import celeryconfig
from vdw_mws import settings
import os

# 为celery设置环境变量
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "vdw_mws.settings")

## 创建celery app
app = Celery('celery_tasks')

# 从单独的配置模块中加载配置
app.config_from_object(celeryconfig)

# 设置app自动加载任务
app.autodiscover_tasks(['celery_tasks'])
```

#### task.py 邮件服务task
```
from celery_tasks.celery import app as celery_app # 导入创建好的celery应用
from django.core.mail import send_mail # 使用django内置函数发送邮件
from vdw_mws import settings  # 导入django的配置

@celery_app.task
def send_mail_task(title,email,msg):
# 使用django内置函数发送邮件
send_mail(title, '', settings.EMAIL_FROM,[email],html_message=msg)
```

#### 在django的settings文件中增加邮箱测试配置文件
```
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.163.com'
EMAIL_PORT = 465
EMAIL_USE_SSL = True  # SSL加密方式
# #发送邮件的邮箱
EMAIL_HOST_USER = 'huangrong08260@163.com'
# #在邮箱中设置的客户端授权密码
EMAIL_HOST_PASSWORD = 'token95'
# #收件人看到的发件人，必须和上面的邮箱一样，否则发不出去
EMAIL_FROM = '天天生鲜<huangrong08260@163.com>'
```

#### 启动服务
```
celery worker -A celery_tasks -l INFO -c 2 -Q for_email


[tasks]
  . celery_tasks.tasks.send_mail_task
 

[2020-03-29 23:10:16,922: INFO/MainProcess] Connected to redis://127.0.0.1:6379/8
[2020-03-29 23:10:16,933: INFO/MainProcess] mingle: searching for neighbors
[2020-03-29 23:10:17,939: INFO/MainProcess] mingle: all alone
[2020-03-29 23:10:17,958: WARNING/MainProcess] /Users/baihe/.virtualenvs/mws/lib/python3.6/site-packages/celery/fixups/django.py:265: UserWarning: Using settings.DEBUG leads to a memory leak, never use this setting in production environments!
  warnings.warn('Using settings.DEBUG leads to a memory leak, never '
[2020-03-29 23:10:17,959: WARNING/MainProcess] celery@baihe.lan ready.
```

#### 测试邮件发送
```
(mws) ➜  mws git:(personal/baihe/2020-03-22-celery-framework) python3
Python 3.6.6 (v3.6.6:4cf1f54eb7, Jun 26 2018, 19:50:54)
[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>>
>>>
>>> from celery_tasks.tasks import send_mail_task
>>> title = '访问百度'
>>> msg = '<a href="http://www.baidu.com/" target="_blank">访问百度</a>'
>>> email = 'b_aihe@163.com'
>>> send_mail_task.delay(title, email, msg)
<AsyncResult: dd90e491-aeed-44c9-b8f9-08785672d829>


# 查看队列服务输出

[2020-03-29 23:11:54,849: INFO/MainProcess] Received task: celery_tasks.tasks.send_mail_task[dd90e491-aeed-44c9-b8f9-08785672d829]

[2020-03-29 23:11:56,906: INFO/MainProcess] Task celery_tasks.tasks.send_mail_task[dd90e491-aeed-44c9-b8f9-08785672d829] succeeded in 2.05399569599831s: None
```

#### 启动命令参数说明

```
celery worker -A celery_tasks -Q for_email -l INFO -c 2  -f logs/message.log

参数1：celery 固定参数，用于启动celery
参数2：启动的celery组件，这里启动的是worker，用于执行任务
参数3、4：参数为一组，启动的task任务
参数5、6：参数为一组，用于指定消费队列，参数中，'-Q', 'message_queue'两个参数，是指定这个worker消费名为“message_queue”的队列
参数7、8：参数为一组，指定日志的级别，这里记录级别为info的日志
参数9、10：参数为一组，-c 2，指定启动多少个work进程
参数11、12：参数为一组，指定日志文件的位置，这里将日志记录在log/message.log

额外说明：当在高并发场景下，使用gevent或者event时，可以用-P 参数，-P eventlet。
```

### celery  其他相关命令
```
# 发布任务
celery -A celery_task beat
# 执行任务
celery -A celery_task worker -l info -P eventlet
# 将以上两条合并
celery -B -A celery_task worker
# 后台启动celery worker进程
celery multi start work_1 -A appcelery
# 停止worker进程，如果无法停止，加上-A
celery multi stop WORKNAME
# 重启worker进程
celery multi restart WORKNAME
# 查看进程数
celery status -A celery_task
```

### celery 监控软件flower

#### 安装flower
```
pip3 install flower
```

#### 启动flower
```
# 指定对外访问IP及redis为broker
celery flower --address=127.0.0.1 --broker='redis://localhost'

# basic_auth开启认证
celery flower --address=127.0.0.1 --broker='redis://localhost' --basic_auth=admin:123456
```

#### 查看页面
```
http://127.0.0.1:5555/
```

![](https://static.studytime.xin/image/articles/20200330000720.png)


### celery 和 supervisor 结合使用

#### supervisor 安装和使用

[supervisor 在centos下的安装和使用文档](https://www.studytime.xin/php/2020/02/27/php-supervisor-use.html)

#### celery 邮件队列服务在supervisor中的配置

```
cd /etc/supervisord.d

vim send-email-task.conf

[program:send-email-task]
process_name=%(program_name)s_%(process_num)02d
command=/root/.virtualenvs/vdw_mws/bin/python /root/.virtualenvs/vdw_mws/bin/celery worker -A celery_tasks -l INFO -c 8 -Q for_adv_finances_count
environment=PATH="~/.virtualenvs/vdw_mws/bin"
directory=/data/wwwroot/mws
autostart=true
autorestart=true
user=root
numprocs=1
redirect_stderr=true
stdout_logfile=/data/log/vdw_mws/celery-adv-finances-count.log
stopwatisecs=60
priority=994
stdout_logfile_maxbytes = 10MB
```

#### 重启supervisor服务
```
systemctl restart supervisord
```