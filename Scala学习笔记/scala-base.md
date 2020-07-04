---
title: scala语言学习（一）、初识
permalink: scala-base
date: 2020-07-05 00:15:20
updated:  2020-07-05 00:15:22
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala是运行在 JVM 上的多范式编程语言，同时支持面向对象和面向函数编程。早期scala刚出现的时候，并没有怎么引起重视，随着Spark和Kafka这样基于scala的大数据框架的兴起，scala逐步进入大数据开发者的眼帘。scala的主要优势是它的表达性。
---
### scala简介
scala是运行在 JVM 上的多范式编程语言，同时支持面向对象和面向函数编程。早期scala刚出现的时候，并没有怎么引起重视，随着Spark和Kafka这样基于scala的大数据框架的兴起，scala逐步进入大数据开发者的眼帘。scala的主要优势是它的表达性。

### 官网地址 http://www.scala-lang.org

### 为什么要使用scala
* 开发大数据应用程序（Spark程序、Flink程序）
* 表达能力强，一行代码抵得上Java多行，开发速度快
* 兼容Java，可以访问庞大的Java类库

### scala 环境安装

#### 安装scala编译器以及开发工具

##### Java程序编译执行流程
![](https://static.studytime.xin/article/20200704234331.png)

##### Scala程序编译执行流程
![](https://static.studytime.xin/article/20200704234457.png)

#### 安装Java JDK

#### 安装scala SDK
scala SDK是scala语言的编译器，要开发scala程序，必须要先安装SDK

##### 下载安装包scala-2.11.8.zip
[https://www.scala-lang.org/download/2.11.8.html](https://www.scala-lang.org/download/2.11.8.html)

![](https://static.studytime.xin/article/20200704234712.png)

##### 解压文件且配置环境变量
```
vim ~/.bash_profile

export SCALA_HOME=/Users/baihe/Code/tools/scala-2.11.8
export PATH=$PATH:$SCALA_HOME/bin

source ~/.bash_profile
```

### IDEA的scala插件
IDEA默认是不支持scala程序开发，所以需要来安装scala插件来支持scala语言

### 点击File ，再点击Settings

![](https://static.studytime.xin/article/20200704234832.png)
![](https://static.studytime.xin/article/20200705003716.png)
#### 查询scala，选择install

### scala的REPL交互式解释器
Scala提供的最重要的一个工具是交互模式（REPL）。==REPL是一个交互式解释器==，可以即时编译、运行代码并返回结果，方便前期做学习和测试。
REPL： R(read)、E(evaluate) 、P（print）、L（loop）
要启动scala解释器，只需要打开控制台，`scala`命令即可。
使用`:quit`退出scala解释器
