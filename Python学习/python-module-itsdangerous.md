---
title: python模块：itsdangerous模块
permalink: python-module-itsdangerous
date: 2020-05-04 22:13:35
updated: 2020-05-04 22:13:36
tags: 
    - python
    - pycharm
    - itsdangerous
categories: python
toc: true
excerpt: 有时您只想将一些数据发送到不受信任的环境。但是如何安全地做到这一点？诀窍就是签名。只要知道一个密钥，您就可以对数据进行加密签名并将其移交给其他人。当您取回数据时，可以轻松确保没有人篡改数据。使用itsdangerous可以实现此种方案。
---


有时您只想将一些数据发送到不受信任的环境。但是如何安全地做到这一点？诀窍就是签名。只要知道一个密钥，您就可以对数据进行加密签名并将其移交给其他人。当您取回数据时，可以轻松确保没有人篡改数据。使用[itsdangerous](https://pythonhosted.org/itsdangerous/)可以实现此种方案。

### 安装
```
pip install itsdangerous
```

### 使用实例一
```python
import itsdangerous

salt='sdaf'#加盐
t=itsdangerous.TimedJSONWebSignatureSerializer(salt,expires_in=600)#过期时间600秒

info = {'username':'baihe','user_id':1}

# =========加密token============
res=t.dumps(info)
token=res.decode()#指定编码格式
print(token)
# eyJleHAiOjE1NzUwMDczNjgsImlhdCI6MTU3NTAwNjc2OCwiYWxnIjoiSFM1MTIifQ.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6InlhbmdmYW4ifQ.yUb3PW53V89ZX4Ci2qeaBJIiizt0JUAN_W9BBzg8QkIR1-uO7NQl6jizSUReOFGanWzfG19t7XFHCWv1JGMIZw


# =========解密token============

res = t.loads('eyJhbGciOiJIUzUxMiIsImV4cCI6MTU3NTAwNzM0MywiaWF0IjoxNTc1MDA2NzQzfQ.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6InlhbmdmYW4ifQ.k-Q1VyN2TOlQ4flHHoiOYEMRaUEiN5Ms2JgeRdnCZWbQB-WwQ1FScoBWxFGkCYEPoWVpAjQxDBQeBesmulZupQ')
# res = t.loads(token)
print(res)
# {'username': 'baihe', 'user_id': 1}


# 当超时或值有误则会报错
```

### 使用实例二
```python
import itsdangerous

salt='sdaf'#加盐
t=itsdangerous.TimedJSONWebSignatureSerializer(salt,expires_in=600)#过期时间600秒

info = {'username':'baihe','user_id':1}

# =========加密token============
res=t.dumps(info)
token=res.decode()#指定编码格式
print(token)
# eyJhbGciOiJIUzUxMiIsImlhdCI6MTU0MTgxOTcyMCwiZXhwIjoxNTQxODIwMzIwfQ.eyJ1c2VybmFtZSI6InlhbmdmYW4iLCJ1c2VyX2lkIjoxfQ.VjCgry9Sr-4iRsK_MHYThcn_O7js9BERrXzocc7BI1aavC3N3s3e0wWMsvq2-Qp-ol_WNMD23wxiYRrA1kwCbg

# =========解密token============

res = t.loads(token)
print(res)
# {'username': 'baihe', 'user_id': 1}


# 当超时或值有误则会报错
```

### 使用实例三
```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

salt='abcdefg'  # 这里就是配置加密的规则
serializer=Serializer(salt,expires_in=3600)  # 过期时间一小时，
info = {'confirm':1}
# 加密阶段
res=serializer.dumps(info)# 得到加密后的数据，会返回一个字节类型的数据
token=res.decode()  # 解码为str
print(token)
# 得到的数据如下，就是包含数据和盐值的token了，只有在知道盐值的时候才能被解密出来
#eyJhbGciOiJIUzUxMiIsImlhdCI6MTU2MjY0Nzg4NCwiZXhwIjoxNTYyNjUxNDg0fQ.eyJjb25maXJtIjo1fQ.93DtXu9vHQDW0lr7saJhDBt-dcBxNNh_IMTR-JhWnrT-ujQ9SwevSUyW0p2txLS-gtyRHPlH1eD9INksIWilkA

# 解密阶段
res=serializer.loads(token)
print(res)
# 返回的数据如下：
# {'confirm':1}
```

### 特殊说明
诚然，接收者可以破译内容，来看看你的包裹里有什么，但他们没办法修改你的内容，除非他们也有你的密钥。所以只要你保管好你的密钥，并且密钥足够复杂，一切就OK了。

itsdangerous内部默认使用了HMAC和SHA1来签名，基于 Django 签名模块。它也支持JSON Web 签名 (JWS)。这个库采用BSD协议，由Armin Ronacher编写，而大部分设计与实现的版权归Simon Willison和其他的把这个库变为现实的Django爱好者们。

