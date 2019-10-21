---
title: python 指定长度list初始化
date: 2019-06-18 15:26:32
tags:
 - python
 - list
categories: python
---

## 指定长度list初始化
想要找到初始化指定长度list最快的方法。
方法一：
``` python
length = 10
array = [[]] * length
```

方法二
``` python
length = 10
array = [[] for _ in range(length)]
```

事实上，只有第二种方法是对的。第一种方法中，arrary中的10个[]都指向了同一个对象。。

## 示例
``` python
length = 10
v1 = [[]]*length
for i in range(length):
    v1[i].append(i)
print(v1)
# [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]]

v2 = [[] for _ in range(length)]
for i in range(length):
    v2[i].append(i)

print(v2)
# [[0], [1], [2], [3], [4], [5], [6], [7], [8], [9]]
```


