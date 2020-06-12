---
title: python模块：JSON模块
permalink: python-module-json
date: 2020-05-04 22:13:29
updated: 2020-05-04 22:13:35
tags: 
    - python
    - pycharm
    - json
categories: python
toc: true
excerpt: 在日常开发中，对数据进行序列化和反序列化，是常备的操作。而在Python标准库中提供了json模块对JSON数据的处理功能。
---


在日常开发中，对数据进行序列化和反序列化，是常备的操作。而在Python标准库中提供了json模块对JSON数据的处理功能。


### 什么是json?
JSON（JavaScript Object Notation）是一种使用广泛的轻量数据格式，相对于XML而言更简单，也易于阅读和编写，机器也方便解析和生成，Json是JavaScript中的一个子集。

Json 模块提供了四个方法： dumps、dump、loads、load。

Python的Json模块序列化与反序列化的过程分别是 encoding和 decoding
encoding：把一个Python对象编码转换成Json字符串
decoding：把Json格式字符串解码转换成Python对象


对于简单数据类型（string、unicode、int、float、list、tuple、dict），可以直接处理。

### JSON 序列化
JSON 序列化，是把一个Python对象编码转换成Json字符串。
具体的操作为：`json.dumps()`

```
import json
Json_str = [
        {"name":"Tom", "aga":20},
        {"name":"Jack", "aga":16}
    ]
student =json.dumps(Json_str)

print(type(student))
#<class 'str'>
上述list已经序列化为JSON字符串

print(student)
#[{"name": "Tom", "aga": 20}, {"name": "Jack", "aga": 16}]
```

### JSON 反序列化
JSON 反序列化，是把Json格式字符串解码转换成Python类型。
具体的操作为：`json.loads()`

python中将json反序列化list：
```
import json
Json_str = '[{"name":"Tom", "aga":20},{"name":"Jack", "aga":16}]'
student = json.loads(Json_str)

print(type(student))
#<class 'list'>
print(student)
#[{'aga': 20, 'name': 'Tom'}, {'aga': 16, 'name': 'Jack'}]
```

python中将json反序列化dict字典：

```
import json
Json_str = '{"name":"Tom", "aga":20, "sex":"female"}'
注意上述字符串要加引号：双引号，数字不用加，布尔值不用加
整个字符串可以用单引号包装；

student = json.loads(Json_str)
print(type(student))
#输出 <class 'dict'>
print(student)
#输出 {'name': 'Tom', 'sex': 'female', 'aga': 20}

访问JSON的成员
print(student['name'])
```


### 数据类型转换
在默认实现中, json.dumps可以处理的Python对象, 及其所有的属性值, 类型必须为dict, list, tuple, str, float或者int。

默认实现中, JSON和Python之间的数据转换对应关系如下表:

| JSON | Python |
| --- | --- |
| object | dict |
| array | list |
| string | str |
| number | int/float |
|  true/false | True/False |
|  null | None |