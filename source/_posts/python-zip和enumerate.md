---
title: python zip和enumerate
date: 2019-04-13 14:59:12
tags:
 - python
categories: python
---

## zip function
``` python
import numpy as np
a = np.zeros((2,2,2))
b = np.zeros((2,2,2))
c = np.zeros((2,2,2))
d = zip(a,b,c)   
print(list(d))        
d = list(zip(a,b,c))
e,f,g = d
```
这里d是一个什么呢，是多个tuple，数量是min(len(a),len(b),len(c))，每一个element是一个tuple，这个tuple的内容为(a[0],b[0],c[0])，....
打印出list(d)是一个list，这个list的长度为min(len(a),len(b),len(c))每一个element是一个tuple，tuple的形状是((2,2),(2,2),(2,2))
用zip的话，就是看一下它的len，然后在第一维上对他们进行拼接，形成多个新的元组。
例子
``` python
a = (2,3)
b = (3,4)
c = (4,5)
d = zip(a,b,c)
print(list(c))
```
> [(2,3),(3,4),(4,5)]    

相当于吧tuple a和tuple b分别当做一个list的一个元组，然后结合成一个新的tuple的list，

## enumerate(iterable, start=0)
``` python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
print(list(enumerate(seasons)))
print(list(enumerate(seasons, start=1)))
for i in enumerate(seansons):
   print(i)
```
> [(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
(0, 'Spring')
(1, 'Summer')
(2, 'Fall')
(3, 'Winter')


