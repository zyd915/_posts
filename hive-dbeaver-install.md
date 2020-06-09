---
title: hive 学习之路（六)、Hive可视化工具Dbeaver配置使用
date: 2020-06-09 01:15:20
updated: 2020-06-09 01:15:22
tags:
    - 大数据
    - hive
categories: hive
toc: true
thumbnail: https://static.studytime.xin/article/kWlG922vxMM.jpg
excerpt: Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成。通过图形化界面的方式来操作我们的数据库与数据库表，类似于我们的sqlyog或者navicat。
---
Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成。
通过图形化界面的方式来操作我们的数据库与数据库表，类似于我们的sqlyog或者navicat。
![](https://static.studytime.xin/article/20200607183920.png)

### 下载Dbeaver
![](https://static.studytime.xin/article/20200607183729.png)

- 我们可以直接从github上面下载我们需要的对应的安装包即可[dbeaver](https://github.com/dbeaver/dbeaver/releases)

- 或者官网[dbeaver](https://dbeaver.io/download/)

### 常见报错
```
报错信息：
Could not establish connection to jdbc:hive2://172.16.250.240:10000/test: Required field 'client_protocol' is unset! Struct:TOpenSessionReq(client_protocol:null, configuration:{use:database=test})

Required field 'client_protocol' is unset! Struct:TOpenSessionReq(client_protocol:null, configuration:{use:database=test})
Required field 'client_protocol' is unset! Struct:TOpenSessionReq(client_protocol:null, configuration:{use:database=test})
```

### 安装dbeaver

#### 根据下载的dbeaver.dmg，双击安装

#### 然后启动dbeaver图形化界面
![](https://static.studytime.xin/article/20200609011902.png)

#### 新建hive连接，配置主机名、端口号、用户名等
![](https://static.studytime.xin/article/20200607184019.png)

#### 配置修改驱动
```
下载需要的驱动
[hadoop@node03 ~]$ cd /opt/module/hive-1.1.0-cdh5.14.2/lib/
[hadoop@node03 lib]$ sz hive-jdbc-1.1.0-cdh5.14.2-standalone.jar
```

#### 删除默认驱动配置
![](https://static.studytime.xin/article/20200607184057.png)

#### 加载下载的驱动配置
![](https://static.studytime.xin/article/20200607184228.png)

### 测试连接
![](https://static.studytime.xin/article/20200609011401.png)