---
title: C++ STL associative container
date: 2019-11-10 12:38:19
tags:
 - C/C++
 - STL
 - 容器
categories: C/C++
---

## C++标准程序库中的关联容器
### 有序关联容器
`map`, `multimap`, `set`, `multiset`的底层实现是红黑树。

### 无序关联容器
`unordered_set`, `unordered_multiset`, `unordered_map`, `unordered_multimap`的底层实现哈希。

## `pair`
`pair`是一种标准库类型，定义在头文件`utility`中，一个`pair`保存两个数据成员，是一个生成特定类型的模板。创建一个pair时，需要提供两个类型名（可以相同，也可以不同）。
**pair的数据成员是public的，两个成员的名字分别为`first`和`second`**。pair支持的操作如下表。
![pair](pair.png)

## 关联容器的定义

## 关联容器的类型
除了所有容器都有的类型之外，关联容器还有一些特有的类型：
- `key_type`，关键字的类型。
- `mapped_type`，每个关键字关联的类型，只有map有。
- `value_type`，对于`set`，和`key_type`相同，对于`map`，和`pair<key_type, mapped_type>`相同。

## 添加元素（增）
可以使用以下几个函数进行插入操作：
- `c.insert(v)`
- `c.emplace(args)`
- `c.insert(b, e)`
- `c.insert(il)`
- `c.insert(p, v)`
- `c.emplace(p, args)`

对map和set进行insert时，insert的返回值是一个pair，pair的第一个元素是迭代器，第二个元素是个bool类型，之处是否插入成功。
而对multimap和multiset进行insert时，insert不需要返回bool值，因为插入总是成功的。

## 删除元素（删）
- `c.erase(p)`，删除迭代器p指定的元素。
- `c.erase(k)`，删除key为k的元素，返回值为删除的元素的数量。
- `c.erase(b, e)`

## 下标操作
- `c[k]`，返回关键字为k的元素。如果k不在c中，添加一个关键字为k的元素，进行值初始化。
- `c.at(k)`，访问关键字为k的元素，进行参数检查，如果k不在c中，抛出一个out_of_range异常。

## 查找操作（查）
- `c.find(k)`，
- `c.count(k)`，
- `c.lower_bound(k)`，不适用于无序容器，
- `c.upper_bound(k)`，不适用于无序容器，
- `c.equal_range(k)`


## 关联容器的无序版本
### bucket的管理

## 参考文献
1.《C++ Primer》第五版　
