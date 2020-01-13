---
title: C++ generic algorithm
date: 2019-11-10 12:57:52
tags:
 - C/C++
 - generic algorithm
categories: C/C++
---


## 泛型算法
头文件`<algorithm>`定义了许多通用算法，而头文件`<numeric>`定义了一组数值泛型算法。
**算法并不依赖于容器，即算法不直接操作容器，而是运行于迭代器之上，执行迭代器的操作。因此算法永远不会改变底层容器的大小，但是它可能改变容器中保存的元素，也可能在容器内移动元素，但永远不会直接添加或者删除元素。**
**标准库定义了一类特殊的迭代器，叫做插入器(inserter)，插入器可以向容器添加元素。给插入器赋值的时候，它们会在底层的容器上执行插入操作。因此，当一个算法操作这样的迭代器时，迭代器可以完成向容器添加元素的效果，但是算法自身不会直接操作容器。**

绝大部分算法的都对一个范围内的元素进行操作，这个范围被称为输入范围，接收输入范围的算法总是使用前两个参数表示这个范围。可以将它们分为：
- 只读算法，如find, find_if, count, count_if accumulate，equal等。
- 写容器元素的算法，如fill,fill_n，copy,replace,replace_copy等。
- 重排容器元素的算法，如sort, unique等。

## 定制操作
这一节主要介绍了如何向算法传递可调用对象。总共有三种方法：
1. predicate。predicate是一个可调用的表达式，返回结果是一个能用作条件的值。C++ 使用了unary predicate和binary predicate。
2. lambda表达式，具体的可以查看[]()。
3. bind 绑定参数。

## 再谈迭代器
这一节介绍了四种特殊的迭代器：
1. insert iterator
2. iostream iterator
3. reverse iterator
4. move iterator

## 迭代器类别


## 算法形参模式

## 算法命令规范

## 特定容器算法
list和forward_list单独定义了sort, merge, remove, reverse和unique。
因为通用的sort需要使用random_access_iterator，所以不能用于list和forward_list。而上述的其他算法的通用版本可以用于list和forwrad_list，但是代价太高了。这些算法需要交换输入序列中的算法，list和forward_list可以仅仅通过改变元素之间的链接而不是真的交换它们的值实现更快的交换，这样的性能比通用的版本要更好一些。list和froward_list还有一个特殊的splice算法。
![]()
![]()


**注意，链表特有版本的算法会改变底层的容器。**

## 参考文献
1.《C++ Primer》第五版中文版
