---
title: Dbeaver解决无法下载数据库驱动
permalink: hive-dbeaver-install-question
date: 2020-06-10 01:15:20
updated: 2020-06-10 01:15:22
tags:
    - 大数据
    - hive
    - dbeaver
categories: [大数据,hive]
toc: true
thumbnail: https://static.studytime.xin//studytime/image/articles/GcOZJe.jpg
excerpt: Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成，不同的数据库类型连接其需要下载不同的驱动，往往驱动包都是国外，因为都知道的原因会出现下载慢和无法下载驱动的问题，下面整理了解决驱动下载问题的方法。
---

Dbeaver是一个图形化的界面工具，专门用于与各种数据库的集成，通过dbeaver我们可以与各种数据库进行集成，不同的数据库类型连接其需要下载不同的驱动，往往驱动包都是国外，因为都知道的原因会出现下载慢和无法下载驱动的问题，下面整理了解决驱动下载问题的方法。

### 配置hive数据库连接，增加相关配置

![](https://static.studytime.xin//studytime/image/articles/E3NMby.png)

### 点击测试连接按钮，此时会下载驱动，往往就在此时出现驱动慢或者无法下载问题。


### 在Hive连接配置页面，编辑驱动设置
![](https://static.studytime.xin//studytime/image/articles/Dfh8gt.png)


### 修改驱动所使用的的maven镜像为阿里云源的maven

```
alimaven aliyun maven http://maven.aliyun.com/nexus/content/groups/public/ central
```
![](https://static.studytime.xin//studytime/image/articles/RtlZLh.png)

![](https://static.studytime.xin//studytime/image/articles/66WfZW.png)

### 驱动下载完成，测试数据库链接
![](https://static.studytime.xin//studytime/image/articles/3kiavP.png)


