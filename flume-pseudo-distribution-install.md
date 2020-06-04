---
title: Flume 学习之路（五）Flume 的部署安装
date: 2020-05-09 22:32:02
updated: 2020-05-09 22:32:03
tags: 
    - 大数据
    - flume
categories: flume
toc: true
excerpt: Flume 框架对 Hadoop 和 Zookeeper 的依赖只是在 jar 包上，并不要求 Flume 启动时必须将 Hadoop 和 Zookeeper 服务也启动。

---

Flume 框架对 Hadoop 和 Zookeeper 的依赖只是在 jar 包上，并不要求 Flume 启动时必须将 Hadoop 和 Zookeeper 服务也启动。

### Flume瞎子
[下载地址：http://flume.apache.org/download.html](http://flume.apache.org/download.html)
http://archive.cloudera.com/cdh5/cdh/5/

### Flume 解压
```
tar -zxvf apache-flume-1.7.0-bin -C ~/App
```

### 修改配置文件
```
cd ~/App/apache-flume-1.7.0-bin/conf

cp flume-env.sh.template flume-env.sh

vim flume-env.sh

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
```

### 配置环境变量
```
vim ~/.bash_profile

#FLUME
export FLUME_HOME=/Users/baihe/App/apache-flume-1.7.0-bin
export PATH=$PATH:$FLUME_HOME/bin

source ~/.bash_profile
```

### 查看版本
```
flume-ng version

➜  conf flume-ng version
Flume 1.7.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: 511d868555dd4d16e6ce4fedc72c2d1454546707
Compiled by bessbd on Wed Oct 12 20:51:10 CEST 2016
From source with checksum 0d21b3ffdc55a07e1d08875872c00523
```



