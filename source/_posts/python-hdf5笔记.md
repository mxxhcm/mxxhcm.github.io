---
title: python h5py笔记
date: 2019-03-18 15:12:03
tags:
 - python
categories: python
---

## python包安装
~$:pip install h5py

## 简介

### 创建和打开h5py文件
f = h5py.File("pathname","w")
w     create file, truncate if exist   
w- or x  create file,fail if exists
r         readonly, file must be exist r+        read/write,file must be exist
a        read/write if exists,create othrewise (default)

### 删除一个dataset或者group
``` python
del group["dataset_name/group_name"]
```

## dataset

### 什么是dataset
datasets和numpy arrays挺像的

### 创建一个dataset
``` python
f = h5py.File("pathname","w")
f.create_dataset("dataset_name", (10,), dtype='i')
f.create_dataset("dataset_name", (10,), dtype='c')
```
第一个参数是dataset的名字, 第二个参数是dataset的shape, dtype参数是dataset中元素的类型。

### 如何访问一个dataset
```python
dataset = f["dataset_name"]                           # acess like a python dict
dataset = f.create_dateset("dataset_name")  # or create a new dataset
```

### dataset的属性
dataset.name        #输出dataset的名字
dataset.tdype        #输出dataset中elements的type
dataset.shape        #输出dataset的shape
dataset.value
dataset doesn't hava attrs like keys,values,items,etc..

### 给h5py dataset复制numpy array
``` python
array = np.zero((2,3,4)
h['array'] = array        # in h5py file, you need't to explicit declare the shape of array, just assign it an object of numpy array
```

## group 

### 什么是group
group和字典挺像的

### 创建一个group
``` python
group = f.create_group("group_name")    #在f下创建一个group
group.create_group("group_name")        #在group下创建一个group
group.create_dataset("dataset_name")    #在group下创建一个dataset
```

### 访问一个group(the same as dataset)
``` python
group = f["group_name"]                           # acess like a python dict
group = f.create_dateset("group_name")  # or create a new group
```

### group的属性和方法
group.name        #输出group的名字
以下内容分为python2和python3版本
#### python 2 版本
group.values()    #输出group的value
group.keys()        #输出gorup的keys
group.items()    #输出group中所有的item，包含group和dataste
#### python 3 版本
list(group.keys())   
list(group.values())
list(group.items())

## 属性

### 设置dataset属性
dataset.attrs["attr_name"]="attr_value"    #设置attr
print(dataset.attrs["attr_name"])                #访问attr

### 设置group属性
group.attrs["attr_name"]="attr_value"    #设置attr
print(group.attrs["attr_name"])                #访问attr

## numpy and h5py
``` python
f = h5py.File(pathname,"r")

data = f['data']    # type 是dataset
data = f['data'][:] #type是numpy ndarray
f.close()
```

## 参考文献
1.http://docs.h5py.org/en/latest/index.html
2.https://stackoverflow.com/questions/31037088/discovering-keys-using-h5py-in-python3
