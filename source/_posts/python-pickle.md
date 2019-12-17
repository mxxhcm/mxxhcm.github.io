---
title: python pickle
date: 2019-10-08 17:56:48
tags:
 - python
categories: python
---

## 简介
pickle是一个序列化模块，它能将python对象序列化转换成二进制串再反序列化成python对象。

## 常用函数
### pickle.dump()
#### API
``` python
pickle.dump(obj, file, [,protocol])
```
#### 作用
将python对象obj以二进制字符串形式保存到文件file中，使用protocol。
#### 示例
``` python
import pickle

dictionary = {"name": "mxx", "age": 23}

with open("test.txt", 'wb') as f:
    pickle.dump(dictionary, f)
```

### pickle.load()
#### API
``` python
pickle.load(file)
```
#### 作用
从文件file中读取二进制字符串，将其反序列成python对象。
#### 示例
``` python
import pickle

with open("test.txt", 'rb') as f:
    b = pickle.load(f)

print(b)
print(type(b))
```

### pickle.dumps()
#### API
``` python
pickle.dumps(obj, [,protocol])
```

#### 作用
将python对象obj转化成二进制字符串，返回一个字符串

#### 示例
``` python
import pickle

dictionary = {"name": "mxx", "age": 23}

s = pickle.dumps(dictionary)
print(s)
print(type(s))
# 输出
# b'\x80\x03}q\x00(X\x04\x00\x00\x00nameq\x01X\x03\x00\x00\x00mxxq\x02X\x03\x00\x00\x00ageq\x03K\x17u.'
# <class 'bytes'>
```

### pickle.loads()
#### API
``` python
pickle.loads(string)
```

#### 作用
从二进制字符串中返回序列化前的python obj对象。

#### 示例
``` python
import pickle

dictionary = {"name": "mxx", "age": 23}

s = pickle.dumps(dictionary)
b = pickle.loads(s)
print(b)
print(type(b))
```

## 参考文献
1.https://www.jianshu.com/p/cf91849064e3
