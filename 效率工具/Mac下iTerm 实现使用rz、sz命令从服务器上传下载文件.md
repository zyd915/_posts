---
title: Mac OS 下 iTerm 实现使用 rz/sz 命令从服务器上传下载文件
date: 2020-05-03 17:59:47
permalink: mac-os-iterm-lrzsz
tags: 
    - iTerm2
    - Mac
categories: tool
updated: 2020-05-03 17:59:49
toc: true
excerpt: 在 windows 下通常可以用 xshell、secureCRT 等工具，只要在服务端装好 lrzsz 工具包就可以实现简单方便的文件上传下载。但是在 Mac Os 上用 iTerm 的时候发现 iTerm 原生不支持 rz/sz 命令，也就是不支持 Zmodem 来进行文件传输，下面就整理了怎么处理这种情况。
---

在 windows 下通常可以用 xshell、secureCRT 等工具，只要在服务端装好 lrzsz 工具包就可以实现简单方便的文件上传下载。但是在 Mac Os 上用 iTerm 的时候发现 iTerm 原生不支持 rz/sz 命令，也就是不支持 Zmodem 来进行文件传输，下面就整理了怎么处理这种情况。

### Mac os 下 安装 lrzsz
```
brew install lrzsz
```

### 下载 Iterm2 使用 lrzsz 脚本
```
cd /usr/local/bin

# 克隆下载插件脚本
git clone https://github.com/aikuyun/iterm2-zmodem

# 赋予脚本执行权限
chmod +x cd /usr/local/bin/iterm2-zmodem
```

### 设置 Iterm2 的配置
打开 iTerm2 的 Preferences-> Profiles -> Default -> Advanced -> Triggers 的 Edit 按钮。

输入如下配置信息：
```
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-zmodem/iterm2-send-zmodem.sh
Instant: checked

Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-zmodem/iterm2-recv-zmodem.sh
Instant: checked
```

![](https://static.studytime.xin/2020-05-04-100807.png)

![](https://static.studytime.xin/2020-05-04-100830.png)

到此整个安装流程就结束了，服务器端安装 lrzsz工具包，便可以通过rz、sz命令实现上传下载文件。