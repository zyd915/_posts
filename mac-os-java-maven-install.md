---
title: Mac Os 上如何配置Maven及进行IDEA Maven配置
date: 2020-05-05 17:14:50
updated: 2020-05-05 19:59:49
tags: 
    - mac
    - maven
    - java
categories: java
thumbnail: https://static.studytime.xin/article/20200505170646.png
toc: true
excerpt: Mac Os 上如何配置Maven及进行IntelliJ IDEA Maven配置
---

官方Maven下载地址：http://maven.apache.org/download.cgi

### 解压maven

`tar -zxvf apache-maven-3.6.0-bin.tar.gz -C /Users/baihe/Software/apache-maven-3.6.1`

### 配置全局环境变量
```
vim ~/.bash_profile

# 写入
export MAVEN_HOME=/Users/baihe/Software/apache-maven-3.6.1
export PATH=$PATH:$MAVEN_HOME/bin

# 更新环境变量配置
source ~/.bash_profile
```

### 测试maven
`mvn -v`

![](https://static.studytime.xin/article/20200505171900.png)

### 配置maven本地仓库
Maven很像一个很大仓库，装了很多的jar包，我们需要的时候就去拿，这就涉及到一个“本地仓库”的问题，默认情况，会在/user/name/.m2下，所以我们需要去更改一下。

```
# 创建仓库目录
cd /Users/baihe/Software/apache-maven-3.6.1
mkdir repository

# 修改自定义的本地仓库地址
vim /Users/baihe/Software/apache-maven-3.6.1/conf/settings.xml

# 修改本地的仓库地址
<localRepository> /Users/baihe/Software/apache-maven-3.6.1/repository </localRepository>

# 保存退出
:wq
```

### 中央仓库下载文件到本地库
`mvn help:system`

成功效果如下:
![](https://static.studytime.xin/article/20200505171843.png)

### 配置Intellij IDEA
打开IntelliJ IDEA：`Preference->Build->Build Tools->Maven`，按照图示配置。

![](https://static.studytime.xin/article/20200505171812.png)



