---
title: numpy笔记
date: 2019-03-18 15:15:29
tags:
 - python
 - numpy
categories: python
---

## numpy数组初始化

### numpy.ndarray
#### attribute of the np.ndarray
ndarray.shape        #array的shape
ndarray.ndim            #array的维度
ndarray.size            #the number of ndarray in array
ndarray.dtype        #type of the number in array，dtype可以是'S',int等
ndarray.itemsize        #size of the element in array
``` python
array[array>0].size    #统计一个数组有多少个非零元素，不论array的维度是多少
```


### numpy.array(),numpy.zeros(),numpy.empty()

#### numpy.array()
> np.array(object,dtype=None,copy=True,order=False,subok=False,ndim=0)

#### numpy.zeros()
> np.zeros(shape,dtype=float,order='C')

例子
``` python
np.zeros((3,4),dtype='i')
``` 

#### numpy.empty()
> np.empty(shape,dtype=float,order='C')
例子
``` python
np.empty((3,4),dtype='f')
```

### numpy.random()

#### numpy.random.randn()
返回标准正态分布的一个样本
numpy.random.randn(d0,d1,...,dn)
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
```
> 0.9670298390136767
0.5472322491757223
0.9726843599648843

> 0.9670298390136767
0.5472322491757223
0.9726843599648843

> 0.9670298390136767
0.5472322491757223
0.9726843599648843

> 0.9670298390136767
0.5472322491757223
0.9726843599648843

> 0.9670298390136767
0.5472322491757223
0.9726843599648843

### others
#### numpy.arange()

#### numpy.linspace()

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
```
> [[[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]], [[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]], [[0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0]]]
3
4

## flatten ndarray

### reshape()
``` python
import numpy as np
a = np.zeros((3,4,5))
a.reshape(-1,1)
```

### flatten
``` python
import numpy as np
a = np.zeros((3,4,5))
a.flatten()
```


参考文献
1.https://stackoverflow.com/questions/47231852/np-random-rand-vs-np-random-random
