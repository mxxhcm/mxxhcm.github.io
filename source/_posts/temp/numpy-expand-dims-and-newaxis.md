---
title: numpy expand_dims and newaxis
date: 2019-07-12 20:08:59
tags:
 - python
 - numpy 
 - expand_dims
 - newaxis
categories: python
---

## expand_dims
在数组的某个维度增加一个为1的维度。

### 代码示例
[代码地址]()
``` python
import numpy as np


a = np.ones([2, 3, 4])
print(a.shape)

a1 = np.expand_dims(a, 0)
print(a1.shape)

a2 = np.expand_dims(a, 1)
print(a2.shape)

a3 = np.expand_dims(a, 2)
print(a3.shape)

a4 = np.expand_dims(a, 3)
print(a4.shape)
```

## newaxis
### 简介
newaxis就是None的一个alias
``` python
>>> np.newaxis is None
True
```

### 用途
常用于slicing。用于给数组增加一个维度。
### 代码示例
[代码地址]()
``` python
a = np.ones((2, 3, 4))
print(a.shape)
(2, 3, 4)

a1 = a[np.newaxis]
print(a1.shape)
(1, 2, 3, 4)

a2 = a[:, np.newaxis]
print(a2.shape)
(2, 1, 3, 4)

a3 = a[:, :, np.newaxis]
print(a3.shape)
(2, 3, 1, 4)

a4 = a[:, :, :, np.newaxis]
print(a4.shape)
(2, 3, 4, 1)

a5 = a[:, :, np.newaxis, np.newaxis, :]
print(a5.shape)
(2, 3, 1, 1, 4)
```

## 参考文献
1.https://stackoverflow.com/questions/29241056/how-does-numpy-newaxis-work-and-when-to-use-it
