---
title: numpy笔记
date: 2019-03-18 15:15:29
tags:
 - python
 - numpy
categories: python
mathjax: true
---

## numpy.ndarray
### attribute of the np.ndarray
ndarray.shape        #array的shape
ndarray.ndim            #array的维度
ndarray.size            #the number of ndarray in array
ndarray.dtype        #type of the number in array，dtype可以是'S',int等
ndarray.itemsize        #size of the element in array
``` python
array[array>0].size    #统计一个数组有多少个非零元素，不论array的维度是多少
```
### 改变数组数据类型
将整形数组改为字符型
``` python
a = numpy.zeros((3,4),dtype='i')
a.astype('S')
```

### 将numpy转为list
``` python
a = np.zeros((3,4,5))
b = a.tolist()
print(b)
print(len(b))
print(len(b[0]))
# [[[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]], [[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]], [[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]]]
# 3
# 4
```

### reshape
``` python
import numpy as np
a = np.zeros((3,4,5))
a.reshape(-1, 1)
```

### flatten
``` python
import numpy as np
a = np.zeros((3,4,5))
a.flatten()
```

## numpy数组初始化
- numpy.array()
- numpy.zeros()
- numpy.empty()
- numpy.random()

### numpy.array()
#### API
``` python
np.array(
    object,
    dtype=None,
    copy=True,
    order=False,
    subok=False,
    ndim=0
)
```

### numpy.zeros()
#### API
``` python
np.zeros(
    shape,
    dtype=float,
    order='C'
)
```

#### 代码示例
[代码地址]()
``` python
np.zeros((3, 4),dtype='i')
``` 

### numpy.empty()
#### API
``` python
np.empty(
    shape,
    dtype=float,
    order='C'
)
```
#### 代码示例
[代码地址]()
``` python
np.empty((3, 4),dtype='f')
```

### numpy.random
#### numpy.random.randn()
返回标准正态分布的一个样本
numpy.random.randn(d0, d1, ..., dn)
例子
``` python
np.random.randn(3,4)
```
> array([[ 0.47203644, -0.0869761 , -1.02814481, -0.45945482],
       [ 0.34586502, -0.63121119,  0.35510786,  0.82975136],
       [-2.00253326, -0.63773715, -0.82700167,  1.80724647]])

#### numpy.random.rand()
创建一个给定shape的数组，从区间[0,1)上的均匀分布中随机采样
> create an array of the given shape and populate it with random samples from a uniform disctribution over [0,1)

numpy.random.rand(d0,d1,...,dn)
例子
``` python
np.random.rand(3,4)
```

#### numpy.random.random()
返回区间[0.0, 1.0)之间的随机浮点数
> return random floats in the half-open interval [0.0,1.0)

numpy.random.random(size=None)
例子
``` python
np.random.random((3,4))
```
##### Note
注意，random.random()和random.rand()实现的功能都是一样的，就是输入的参数不同。见参考文献[1]。

#### numpy.random.ranf()
我觉得它和random.random()没啥区别

#### numpy.random.randint()
> return random integers from low(inclusive) to high(exclusive),[low,high) if high is None,then results are from [0,low)

numpy.random.randint(low,high=None,size=None,dtype='l')
例子
``` python
np.random.randint(3,size=[3,4])
np.random.randint(4,6,size=[6,2])
```

#### numpy.random.RandomState()
> class numpy.random.RandomState(seed=None)

这是一个类，给定一个种子，它接下来产生的一系列随机数都是固定的。每次需要重新产生随机数的时候，就重置种子。
通过一个例子来看：
``` python
import numpy as np
rdm = np.randrom.RandomState()
for i in range(3):
   rdm.seed(3)
   print(rdm.rand())
   print(rdm.rand())
   print(rdm.rand())
    print("\n")
# 0.9670298390136767
# 0.5472322491757223
# 0.9726843599648843

# 0.9670298390136767
# 0.5472322491757223
# 0.9726843599648843

# 0.9670298390136767
# 0.5472322491757223
# 0.9726843599648843

# 0.9670298390136767
# 0.5472322491757223
# 0.9726843599648843

# 0.9670298390136767
# 0.5472322491757223
# 0.9726843599648843
```

### 创建bool类型数组
np.ones([2, 2], dtype=bool)
np.zeros([2, 2], dtype=bool)

### others
#### numpy.arange()

#### numpy.linspace()

## np.random.binomial
### API
``` python
numpy.random.binomial(
	n, 
	p, 
	size=None
)
```
### 介绍
二项分布，共有三个参数，前两个是必选参数，第三个是可选参数。$n$是实验的个数，比如同时扔三枚硬币，这里就是$n=3$,$p$是为$1$的概率。$size$是总共进行多少次实验。
返回值是在每次试验中，trival成功的个数。如果是一个scalar，代表$size=1$，如果是一个list，代表$size\gt 1$。

### 代码示例
``` python
import numpy as np

for i in range(10):
    rand = np.random.binomial(2, 0.9)
    print(rand)
# 可以看成扔2个硬币，每个硬币正面向上的概率是0.9,最后有几个硬币正面向上。
# 2
# 2
# 2
# 2
# 2
# 2
# 2
# 1
# 1
# 2

rand = np.random.binomial(3, 0.9, 5)
print(rand)
# 可以看成扔3个硬币，每个硬币正面向上的概率是0.9,最后有几个硬币正面向上。一共进行5次实验。
# [2 2 3 3 2]
```

## np.random.choice
### API
``` python
numpy.random.choice(
    a,  # 1d array或者int，如果是一个数组，从其中生成样本；如果是一个整数，从np.arange(a)中生成样本
    size=None,  # output shape，比如是(m, n, k)的话，总共要m*n*k个样本，默认是None,返回一个样本。
    replace=True,   # 是否使用replacement，设置为False的话所有元素不重复。
    p=None  # 概率分布，相加必须等于1，默认是从一个均匀分布中采样。
)
```

### 代码示例
[代码地址]()
``` python
import numpy as np

a0 = np.random.choice([8, 9, -1, 2, 0], 3)
print(a0)

# 从np.arange(5)从使用均匀分布采样一个shape为4的样本
a1 = np.random.choice(5, 4)
print(a1)

a2 = np.random.choice(5, 8, p=[0.1, 0.2, 0.5, 0.2, 0])
print(a2)

# replace 设置为False，相当于np.random.permutation()
a3 = np.random.choice([1, 2, 3, 8, 9], 5, replace=False)
print(a3)
```

## np.random.permutation
### API
``` python
np.random.permutation(
    x   # int或者array，如果是int，置换np.arange(x)。如果是array，make a copy，随机打乱元素。
)
```
### 简介
对输入序列进行排列组合，如果输入是多维的话，只会在第一维重新排列。

### 代码示例
[代码地址]()
``` python
import numpy as np


a1 = np.random.permutation(9)
print(a1)

a2 = np.random.permutation([1, 2, 4, 9, 8])
print(a2)

a3 = np.random.permutation(np.arange(9).reshape(3, 3))
print(a3)
```

## np.random.normal
### API
```
numpy.random.normal(loc=0.0, scale=1.0, size=None)  
loc:float，正态分布的均值，对应着整个分布的center
scale:float，正态分布的标准差，对应于分布的宽度，scale越大越矮胖，scale越小，越瘦高
size:int or tuple of ints，输出的shape，默认为None，只输出一个值
np.random.randn(size)相当于np.random.normal(loc=0, scale=1, size)
```

## np.argsort
### API
numpy.argsort(a, axis=-1, kind='quicksort', order=None)
axis:对哪个axis进行排序，默认是-1

### 功能
将数组排序后（默认是从小到大排序），返回排序后的数组在原数组中的位置。




参考文献
1.https://stackoverflow.com/questions/47231852/np-random-rand-vs-np-random-random
2.https://stackoverflow.com/questions/21174961/how-to-create-a-numpy-array-of-all-true-or-all-false
3.https://docs.scipy.org/doc/numpy/reference/generated/numpy.random.choice.html
4.https://docs.scipy.org/doc/numpy/reference/generated/numpy.random.permutation.html
