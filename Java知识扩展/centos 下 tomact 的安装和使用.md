---
title: centos 下 tomact 的安装和使用
date: 2020-05-07 20:30:05
updated: 2020-05-07 20:32:05
permalink: java-tomcat-install
tags: 
    - centos
    - java
    - tomcat
categories: java
toc: true
excerpt: 整理汇总  centos 下 tomact 的安装和使用。
---

整理汇总  centos 下 tomact 的安装和使用。

### 前置条件 java sdk已安装 
```
➜  ~ java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

### 下载Tomact

[Tomcat下载链接](https://tomcat.apache.org/download-80.cgi)

![](https://static.studytime.xin/image/articles/spring-boot20190824183211.png)

### 解压文件

```
tar -zxvf apache-tomcat-8.5.45.tar.gz -C ~/App
```

### 配置环境变量

```
vim  ~/.bash_profile

#tomcat
export PATH=$PATH:/Users/baihe/App/apache-tomcat-8.5.45/bin

#保存退出 :wq

source ~/.bash_profile
```

### 启动 Tomcat
```
startup.sh // 启动 tomcat

➜  App startup.sh
Using CATALINA_BASE:   /Users/baihe/App/apache-tomcat-8.5.45
Using CATALINA_HOME:   /Users/baihe/App/apache-tomcat-8.5.45
Using CATALINA_TMPDIR: /Users/baihe/App/apache-tomcat-8.5.45/temp
Using JRE_HOME:        /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
Using CLASSPATH:       /Users/baihe/App/apache-tomcat-8.5.45/bin/bootstrap.jar:/Users/baihe/App/apache-tomcat-8.5.45/bin/tomcat-juli.jar
Tomcat started.

```

如果出现`permision denied`，表示没有权限，需要给予相应权限。执行如下命令：

```
sudo chmod 755 /Users/baihe/App/apache-tomcat-8.5.45/bin
```


### 访问webUI查看是否配置正常

```
http://localhost:8080/
```

![](https://static.studytime.xin/image/articles/spring-boot20190824183820.png)

### 关闭Tomcat命令

```
shutdown.sh 
```

### 查看tomcat版本号

```
➜  conf catalina.sh version
Using CATALINA_BASE:   /Users/baihe/App/apache-tomcat-8.5.45
Using CATALINA_HOME:   /Users/baihe/App/apache-tomcat-8.5.45
Using CATALINA_TMPDIR: /Users/baihe/App/apache-tomcat-8.5.45/temp
Using JRE_HOME:        /Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
Using CLASSPATH:       /Users/baihe/App/apache-tomcat-8.5.45/bin/bootstrap.jar:/Users/baihe/App/apache-tomcat-8.5.45/bin/tomcat-juli.jar
Server version: Apache Tomcat/8.5.45
Server built:   Aug 14 2019 22:21:25 UTC
Server number:  8.5.45.0
OS Name:        Mac OS X
OS Version:     10.14.6
Architecture:   x86_64
JVM Version:    1.8.0_191-b12
JVM Vendor:     Oracle Corporation
```

### tomcat 的目录结构介绍

```
drwxr-x---@ 27 baihe  staff   864B  8 15 06:27 bin
drwx------@ 13 baihe  staff   416B  8 24 19:00 conf
drwxr-x---@ 28 baihe  staff   896B  8 15 06:24 lib
drwxr-x---@  8 baihe  staff   256B  8 24 11:15 logs
drwxr-x---@  3 baihe  staff    96B  8 15 06:24 temp
drwxr-x---@  9 baihe  staff   288B  8 24 11:32 webapps
drwxr-x---@  3 baihe  staff    96B  8 24 11:15 work
```

-  bin:存放tomcat命令
-  conf:存放tomcat配置信息,里面的server.xml文件是核心的配置文件
-  lib:支持tomcat软件运行的jar包和技术支持包(如servlet和jsp)
-  logs:运行时的日志信息
-  temp:临时目录
-  webapps:共享资源文件和web应用目录
-  work:tomcat的运行目录.jsp运行时产生的临时文件就存放在这里

### 默认端口号为8080,修改默认端口，自行确定是否需要

服务器的默认端口是8080,也可以将其改成自定义的端口,为了避免与系统端口冲突,必须设置为1024以上,例如设置为80808

```
cd ~/App/apache-tomcat-8.5.45/conf

vim server.xml

找到
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
修改端口号为
<Connector port="80808" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />

重启后生效

shutdown.sh
startup.sh

```

### 密码设置，部分管理功能，需要密码才可正常

```
cd ~/App/apache-tomcat-8.5.45/conf

vim tomcat-users.xml

<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script"/>
</tomcat-users>

重启服务，查看server
```

查看服务状态，无账号情况提示认证失败
![](https://static.studytime.xin/image/articles/spring-boot20190824190941.png)

输入用户名、密码后
![](https://static.studytime.xin/image/articles/spring-boot20190824190911.png)

![](https://static.studytime.xin/image/articles/spring-boot20190824191013.png)



