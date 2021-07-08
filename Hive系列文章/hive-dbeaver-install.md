---
title: 介绍一款Hive数仓可视化神器、Dbeaver的配置和使用方法
permalink: hive-dbeaver-install
date: 2021-06-09 01:15:20
updated: 2021-06-09 01:15:22
tags:
    - 大数据
    - hive
categories: [大数据,hive,Dbeaver]
toc: true
keywords: [大数据,hive客户端,Dbeaver下载,Dbeaver安装]
thumbnail: https://static.studytime.xin/article/kWlG922vxMM.jpg
excerpt: Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成。通过图形化界面的方式来操作我们的数据库与数据库表，类似于我们的sqlyog或者navicat。
---
Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成。通过图形化界面的方式来操作我们的数据库与数据库表，类似于我们的sqlyog或者navicat。

![](https://static.studytime.xin/article/20200607183920.png)

### 下载Dbeaver
![](https://static.studytime.xin/article/20200607183729.png)

- 我们可以直接从github上面下载我们需要的对应的安装包即可[dbeaver](https://github.com/dbeaver/dbeaver/releases)或官网[dbeaver](https://dbeaver.io/download/)
- 国内百度云Dbeaver地址  链接: https://pan.baidu.com/s/19TOxhU8ToaqvrjFCD0bHpA  密码: d2rc

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

####  新建hive连接，配置主机名、端口号、用户名等
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


### Dbeaver解决无法下载数据库驱动问题解决

Dbeaver专门用于与各种数据库的集成，通过Dbeaver我们可以与各种数据库进行集成，不同的数据库类型连接其需要下载不同的驱动，往往驱动包都是国外，因为都知道的原因会出现下载慢和无法下载驱动的问题，下面整理了解决驱动下载问题的方法。

#### 配置hive数据库连接，增加相关配置

![](https://static.studytime.xin//studytime/image/articles/E3NMby.png)

#### 点击测试连接按钮，此时会下载驱动，往往就在此时出现驱动慢或者无法下载问题。

#### 在Hive连接配置页面，编辑驱动设置
![](https://static.studytime.xin//studytime/image/articles/Dfh8gt.png)

#### 修改驱动所使用的的maven镜像为阿里云源的maven
```
alimaven aliyun maven http://maven.aliyun.com/nexus/content/groups/public/ central
```

![](https://static.studytime.xin//studytime/image/articles/RtlZLh.png)

![](https://static.studytime.xin//studytime/image/articles/66WfZW.png)

#### 驱动下载完成，测试数据库链接

![](https://static.studytime.xin//studytime/image/articles/3kiavP.png)


