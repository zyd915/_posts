---
title: centos 下安装 supervisor及使用
date: 2020-05-07 00:23:49
updated: 2020-05-07 00:23:50
tags: 
    - supervisor
categories: tool
toc: true
excerpt: Supervisor 是一个进程控制系统。它是一个 C/S 系统 (注意：其提供 WEB 接口给用户查询和控制)。它允许用户去监控和控制在类 UNIX 系统的进程。它的目标与 launchd、daemontools 和 runit 有些相似。但是与它们不一样的是、它不是作为 init (进程号 pid 是 1) 运行。它是被用来控制进程、并且它在启动的时候和一般程序并无二致。
---

Supervisor 是一个进程控制系统。它是一个 C/S 系统 (注意：其提供 WEB 接口给用户查询和控制)。它允许用户去监控和控制在类 UNIX 系统的进程。它的目标与 launchd、daemontools 和 runit 有些相似。但是与它们不一样的是、它不是作为 init (进程号 pid 是 1) 运行。它是被用来控制进程、并且它在启动的时候和一般程序并无二致。

### 作用
你的 Nginx，Tomcat，memcache，Redis... 会崩么？
那你自己写的服务器监测脚本呢？
好吧、不要再纠结了、交给 Supervisor 吧！
它会帮你维护这些、即使它们不小心崩了、Supervisor 会帮你看住它们、维护它们。

### 名词说明
supervisor：要安装的软件的名称。
supervisord：装好supervisor软件后，supervisord用于启动supervisor服务。
supervisorctl：用于管理supervisor配置文件中program。

### 安装和配置

#### 前置条件安装前需要将用户切换为root下
#### 安装Linux的epel的yum源的命令，某些yum源会提示无supervisor源码包，此时可以使用此命令
```
yum install epel-release
```
#### 安装
```
yum install -y supervisor
```

#### 设置开启自启
```
systemctl enable supervisord
```

#### 启动supervisord服务
```
systemctl status supervisord
```

#### 查看supervisord服务状态
```
ps -ef|grep supervisord 
```

### 配置
```
vim /etc/supervisord.conf

# 调整增加引入配置文件路径，这个路径放置项目对应的 supervisor 配置文件
[include]
files = /etc/supervisor/*.conf 
```

#### 配置模版
```
[program:laravel-worker1]
process_name=%(program_name)s_%(process_num)02d
command=php /home/wwwroot/studytime.xin/artisan queue:work redis --sleep=3 --tries=3 --daemon
autostart=true
autorestart=true
user=root
numprocs=3
redirect_stderr=true
stdout_logfile=/home/baihe/worker1.log
```

#### 配置说明
```
;*为必须填写项
;*[program:应用名称]
[program:cat]
;*命令路径,如果使用python启动的程序应该为 python /home/test.py, 
;不建议放入/home/user/, 对于非user用户一般情况下是不能访问
command=/bin/cat
;当numprocs为1时,process_name=%(program_name)s
;当numprocs>=2时,%(program_name)s_%(process_num)02d
process_name=%(program_name)s
;进程数量
numprocs=1
;执行目录,若有/home/supervisor_test/test1.py
;将directory设置成/home/supervisor_test
;则command只需设置成python test1.py
;否则command必须设置成绝对执行目录
directory=/tmp
;掩码:--- -w- -w-, 转换后rwx r-x w-x
umask=022
;优先级,值越高,最后启动,最先被关闭,默认值999
priority=999
;如果是true,当supervisor启动时,程序将会自动启动
autostart=true
;*自动重启
autorestart=true
;启动延时执行,默认1秒
startsecs=10
;启动尝试次数,默认3次
startretries=3
;当退出码是0,2时,执行重启,默认值0,2
exitcodes=0,2
;停止信号,默认TERM
;中断:INT(类似于Ctrl+C)(kill -INT pid),退出后会将写文件或日志(推荐)
;终止:TERM(kill -TERM pid)
;挂起:HUP(kill -HUP pid),注意与Ctrl+Z/kill -stop pid不同
;从容停止:QUIT(kill -QUIT pid)
;KILL, USR1, USR2其他见命令(kill -l),说明1
stopsignal=TERM
stopwaitsecs=10
;*以root用户执行
user=root
;重定向
redirect_stderr=false
stdout_logfile=/a/path
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/a/path
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
;环境变量设置
environment=A="1",B="2"
serverurl=AUTO
```

#### 浏览器查看进程信息配置说明
```
[inet_http_server]         ; inet (TCP) server disabled by default
port=0.0.0.0:9001          ; (ip_address:port specifier, *:port for all iface)
username=user              ; 用户名 (default is no username (open server))
password=123               ; 密码 (default is no password (open server))
```


### 其他常用命令
```
# 读取有更新（增加）的配置文件，不会启动新添加的程序
supervisorctl reread 

# 重启配置文件修改过的程序
supervisorctl update 

# 启动 larashop-worker 程序
supervisorctl start larashop-worker:* 

# 查看状态
supervisorctl status 
```


### 常见问题以及说明

#### Error: Another program is already listening on a port that one of our HTTP servers is configured to use.
可能是端口占用，调整配置文件中的端口

#### python环境问题
python2场景下使用supervisord

#### 安装启动过程中提示无配置文件
```
easy_install supervisor
echo_supervisord_conf > /etc/supervisord.conf
```


