---
title: python数组初始化
date: 2019-04-13 14:49:35
tags:
 - python
 - numpy
categories: python
---

## 数组初始化initialize(numpy library)

### array initialize
array_one_dimension =  [ 0 for i in range(cols)]
array_multi_dimension  = [[0 for i in range(cols)] for j in range(rows)]

### numpy.ndarray(numpy.array(),numpy.zeros(),numpy.empty())
#### np.ndarray属性
ndarray.shape        #array的shape
ndarray.ndim            #array的维度
ndarray.size            #the number of ndarray in array
ndarray.dtype        #type of the number in array
ndarray.itemsize        #size of the element in array
array[array > 0].size    #统计一个数组有多少个非零元素，不论array的维度是多少

#### numpy.array()
np.array(object,dtype=None,copy=True,order=False,subok=False,ndim=0)

#### numpy.zeros()
np.zeros(shape,dtype=float,order='C')

#### numpy.empty()
np.empty(shape,dtype=float,order='C')

#### numpy.random.randn(shape)
np.random.randn(3,4)

#### numpy.arange()

#### numpy.linspace()

