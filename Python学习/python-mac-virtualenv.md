---
title: Max Os 下安装和配置Python virtualenv虚拟环境
permalink: python-mac-virtualenv
date: 2020-05-04 20:07:59
updated: 2020-05-04 20:59:52
tags: 
    - python
categories: python
toc: true
excerpt: virtualenv 官方文档对virtualenv的解释是:virtualenv is a tool to create isolated Python environments。 用它可以创建一个独立的 Python 环境，每个项目都可以有一个专属环境，避免了不同各种包安装冲突以及版本要求问题，可以让你更方便快捷的切换不同 Python 环境，更高效的开发。
---


### 简介
virtualenv 官方文档对virtualenv的解释是:virtualenv is a tool to create isolated Python environments。
用它可以创建一个独立的 Python 环境，每个项目都可以有一个专属环境，避免了不同各种包安装冲突以及版本要求问题，可以让你更方便快捷的切换不同 Python 环境，更高效的开发。

### 安装virtualenv
```
pip install virtualenv
```

### 安装 virtualenvwrapper
```
sudo pip install virtualenvwrapper
```

### 查找virtualenvwrapper.sh文件的路径
```
sudo find / -name virtualenvwrapper.sh

# 默认一般在 /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh
```

### 配置环境变量
```
# 打开~/.bash_profile
vim ~/.bash_profile

# 加入配置信息
export WORKON_HOME=$HOME/.virtualenvs
VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3
source /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh

source ~/.bash_profile
```

### 创建虚拟环境
```
mkvirtualenv djangodevenv(虚拟环境名称)
```

### 使用(进入)虚拟环境
```
workon djangodevenv(虚拟环境名称)
```

### 退出虚拟环境
```
deactivate
```

### 查看所有虚拟环境
```
workon 两次tab键
```

### 删除虚拟环境
```
rmvirtualenv djangodevenv(虚拟环境名称)

# 删除虚拟环境前需要先退出方可
先退出：deactivate
再删除：rmvirtualenv py_django
```

### 导出和安装依赖包

#### 安装依赖包,须在虚拟环境中

`pip install -r requirements.txt`

如果此处报Could not open requirements file: [Errno 2] No such file or directory: './requirements.txt'，直接进行下一步命令

#### 生成依赖包,须在虚拟环境中
```
pip freeze > requirements.txt
```

至此，Python虚拟环境Virtualenv安装流程完毕，你可以在你自己的虚拟环境下随意安装各种包，不同项目间也不会相互影响了。