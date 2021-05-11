---
title: MacOs使用国内数据源安装brew
permalink: mac-os-install-brew
tags:
    - Blog
categories: [博客, hexo]
date: 2021-03-10 23:10:43
updated: 2021-03-10 23:10:22
toc: true
excerpt: 本文主要整理了MacOs使用国内数据源安装brew，希望对大家有所帮助。
---


### 终端输入命令
```shell
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 此时会报错，是因为安装brew需要翻墙
![rzJWOF](https://static.studytime.xin//studytime/image/articles/rzJWOF.jpg)

### 解决办法
```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

![Buswa4](https://static.studytime.xin//studytime/image/articles/Buswa4.png)

![g0nI7z](https://static.studytime.xin//studytime/image/articles/g0nI7z.png)


