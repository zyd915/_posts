---
title: 'pymongo 错误处理：IndexError: no such item for Cursor instance。'
permalink: python-pymongo-error-no-such-item
date: 2020-05-06 20:06:06
updated: 2020-05-06 20:59:59
tags: 
    - python
    - PyMongo
categories: python
toc: true
excerpt: '打印输出提示IndexError: no such item for Cursor instance，大意为：Cursor实例没有对应的节点，这样的问题该怎样解决呢？'
---

错误信息 `IndexError: no such item for Cursor instance` ，怎么解决呢？

### 产生原因

```python
result = collection.find({"id": "12345678"})
print result[0]['name']
```
打印输出提示`IndexError: no such item for Cursor instance`，大意为：`Cursor实例没有对应的节点`，这样的问题该怎样解决呢？使用怎样的`if`判定可以判定结果集是有效地呢？


### 解决办法
`result.count()`

### 代码实例
```python
result = collection.find({"id": "12345678"})
if result and result.count():
    print result[0]['name']
```