---
title: Linux环境变量全局有效的设置方法
date: 2021-01-01 21:20:50
updated: 2021-01-01 21:23:50
permalink: linux-profile
tags: 
    - linux
categories: 
toc: true
excerpt: Linux环境变量全局有效的设置方法
---

### 修改/etc/profile文件

用来设置系统环境参数，比如$PATH. 这里面的环境变量是对系统内所有用户生效。使用bash命令，需要`source  /etc/profile`一下。


### 修改~/.bashrc文件
针对某一个特定的用户，环境变量的设置只对该用户自己有效。使用bash命令，只要以该用户身份运行命令行就会读取该文件。

### 把/etc/profile里面的环境变量追加到~/.bashrc目录
```
cat /etc/profile >> ~/.bashrc
```



