---
title: skills
date: 2019-05-29 12:24:41
tags:
 - python
categories: python
---

## 累加
### 简单介绍
今天在看Reinforcment Learning: an Introduction第五章的时候，写了figure_5_4的代码，然后跟github上作者写出来的效率差了太多。
最后对比了一下代码，发现了原因，是因为做了太多重复运算。

### 代码示例
``` python
import numpy as np
import time

# 目的，计算不断更新的两个列表的乘积的和
numbers = 100000
a = []
b = []
c1 = []

begin_time = time.time() 
for i in range(numbers):
    a.append(i-1)
    b.append(i+1)
    c1.append(np.sum(np.multiply(a, b)))
# print(c1)
end_time = time.time()
print("Total time: ", end_time - begin_time)

# 我用的是上面的代码，然后，，，，太多重复运算，效率惨不忍睹

a = []
b = []
c2 = []

begin_time = time.time() 
for i in range(numbers):
    a.append(i-1)
    b.append(i+1)
    c2.append(a[i]*b[i])
results = np.add.accumulate(c2)
# print(results)
end_time = time.time()
print("Total time: ", end_time - begin_time)

"""
在100000的量级上，差了几万倍出来。。。
Total time:  450.6051049232483
Total time:  0.03314399719238281
"""
```

## 参考文献
1.https://github.com/ShangtongZhang/reinforcement-learning-an-introduction/tree/master/chapter05
