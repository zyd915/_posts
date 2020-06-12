---
title: Pycharm 安装使用 black 提升python代码质量
permalink: python-pycharm-black-tool
date: 2020-05-04 22:13:29
updated: 2020-05-04 22:13:30
tags: 
    - python
    - pycharm
    - tool
    - black
categories: python
toc: true
excerpt: 针对代码风格不一致问题，导致的维护成本过高，针对性的镇定代码风格统一标准，是很有必要的。目前市面上用的比较多的python代码格式化工具有YAPF、Black。Black，号称不妥协的代码格式化工具，它检测到不符合规范的代码风格直接就帮你全部格式化好，根本不需要你确定，直接替你做好决定。从而节省关注代码规范的时间和精力，关注编程。
---

针对代码风格不一致问题，导致的维护成本过高，针对性的镇定代码风格统一标准，是很有必要的。目前市面上用的比较多的python代码格式化工具有YAPF、Black。

Black，号称不妥协的代码格式化工具，它检测到不符合规范的代码风格直接就帮你全部格式化好，根本不需要你确定，直接替你做好决定。从而节省关注代码规范的时间和精力，关注编程。

### 安装 black 扩展
```
pip3 install black
```

### 安装目录
```
➜  servers which black
/Library/Frameworks/Python.framework/Versions/3.6/bin/black
➜  servers where black
/Library/Frameworks/Python.framework/Versions/3.6/bin/black
```

### 测试命令行使用
```
➜  mws git:(feature/datawarehouse-v3.0.0) ✗ python3 -m black celery_tasks/__init__.py
All done! ✨ 🍰 ✨
1 file left unchanged.
```

### pycharm 中 集成 black

#### 在 PyCharm 中打开`External tools`
```
# On macOS:
PyCharm -> Preferences -> Tools -> External Tools

# On Windows / Linux / BSD:
File -> Settings -> Tools -> External Tools
```

![](https://static.studytime.xin/image/articles/20200402001716.png)

#### 添加一个新的扩展工具
配置信息如下：
```
Name: Black
Description: Black is the uncompromising Python code formatter.
Program: /Library/Frameworks/Python.framework/Versions/3.6/bin/black
Arguments: "$FilePath$"
Working directory: $ProjectFileDir$
```

![](https://static.studytime.xin/image/articles/20200402001904.png)

#### 如何使用 black 格式化代码

1. 通过选择 `Tools -> External Tools -> black` 来格式化代码。
2. 在代码区域按鼠标右键，选择`External Tools`中的`black`

### 设置快捷键执行 black 格式化代码

打开 `Preferences or Settings -> Keymap -> External Tools -> External Tools - Black`，

![](https://static.studytime.xin/image/articles/20200402003403.png)


### black + File Watchers 自动格式化
确保 File Watchers插件可用，一般而言安装Pycharm应该会默认安装。

选择 `Preferences or Settings -> Tools -> File Watchers` 添加一个新的`watcher`

```
Name: Black
File type: Python
Scope: Project Files
Program: /Library/Frameworks/Python.framework/Versions/3.6/bin/black
Arguments: $FilePath$
Output paths to refresh: $FilePath$
Working directory: $ProjectFileDir$
Uncheck "Auto-save edited files to trigger the watcher"
```

![](https://static.studytime.xin/image/articles/20200402003921.png)
![](https://static.studytime.xin/image/articles/20200402004011.png)

### 特殊说明
两个设置中的参数Program使用 `which black` 查询出来的路径信息。




