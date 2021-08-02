---
title: Max Os 上配置Pycharm连接远程服务器并实现使用远程服务器Python解释器
date: 2020-05-05 20:38:45
updated: 2020-05-05 20:38:46
permalink: mac-os-python-pycharm-server-develop
tags: 
    - mac
    - pycharm
categories: python
toc: true
excerpt: 本文将介绍如何使用公司运行服务器进行开发调试，以及使用远程服务器python解释器，整理了对应的配置流程。
---

本文将介绍如何使用公司运行服务器进行开发调试，以及使用远程服务器python解释器，整理了对应的配置流程。

###  进入配置页面

Pycharm菜单栏，如下图所示，依次点击 `Tools -> Deployment -> Configration…`

![](https://static.studytime.xin/image/articles/20200317221801.png?x-oss-process=image/resize,h_600)



### 开始配置连接服务器

1. Connection 选项设置：

![](https://static.studytime.xin/image/articles/20200317221928.png?x-oss-process=image/resize,h_1000)


具体参数说明：

- name 代码服务器配置名称
- Type 协议类型，协议最好选择 SFTP
- Host、User name、Password 服务器配置相关信息
- Root Path 为服务器项目运行的上级目录，特殊说明结束符不要带/
- Send Keep alive messages eash 调整为10，代表同步频率
- 编码类型如果不是UTF-8需要设置成



2. Mapping 选项设置，主要设置本地项目目录与远程目录的映射

![](https://static.studytime.xin/image/articles/20200317222001.png)


到此，本地和远程服务器的连接同步已经配置完成。可以进行本地和远程服务器代码的上传、下载或者对比。

![](https://static.studytime.xin/image/articles/20200317223224.png)



## 配置使用远程服务器 Python 解释器

使用服务器调试 Python 程序的前提时在服务器上安装了Python解释器，如果没安装，请先安装。具体可以参考作者文章


在菜单栏，File -> Settings… -> Project ×× -> Project Interpreter，点击右侧 Add按钮，添加解释器。


![](https://static.studytime.xin/image/articles/20200317222129.png)


选择SSH Interpreter，选择上一步中设置的服务器。当然也可以填写服务器的 Host 地址，端口Port，用户名Username，填好后，下一步Next。

![](https://static.studytime.xin/image/articles/20200317222214.png)


选择远程服务器上Python解释器的位置，服务器上的远程同步文件夹Sync folders，可以选择多个。如果不知道Python安装在哪，可以远程连接服务器后，使用 命令 which python 找到Python安装位置。


![](https://static.studytime.xin/image/articles/20200317222330.png)


![](https://static.studytime.xin/image/articles/20200317222310.png)

![](https://static.studytime.xin/image/articles/20200317222439.png)

![](https://static.studytime.xin/image/articles/20200317222919.png)

Finish，配置结束。该项目现在使用的就是远程服务器上的Python解释器了。以后的项目若想/不想使用该解释器，手动更改解释器即可。
