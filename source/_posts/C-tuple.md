---
title: C++ tuple
date: 2020-02-18 15:54:42
tags:
 - C/C++
categories: C/C++
---

## tuple
在头文件tuple中，是一个类模板。和pair很像，pair是具有两个不同成员的类模板，而tuple是具有不定个数的成员类型也不定的类模板。

## 创建tuple
1. 直接调用构造函数
2. 使用`make_tuple`函数模板。

## 访问tuple的成员
1. `get<i>(t)`获得tuple t的第i个元素。
2. `tuple_size<tupleType>`类模板，使用它的public成员value获得某个tuple的数据成员的个数。
3. `tuple_element<i, tupleType>`类模板，使用它的public成员type获得某个tupleType类型变量的第i个元素的类型。


## tuple的用处
1. 返回多个值。

## 参考文献
1.《C++ Primer》
