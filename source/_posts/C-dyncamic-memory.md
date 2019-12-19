---
title: C++ dyncamic memory
date: 2019-11-10 12:39:41
tags:
 - C/C++
categories: C/C++
---

## 概念

## 概述
每一个C程序都把内存划分成静态内存，栈内存，堆内存（自由空间）。静态内存存放局部static对象，类的static数据成员以及定义在任何函数之外的变量。栈存放函数内的非static对象。堆内存是由程序员子自己负责管理（申请和释放）的内存。


## 动态内存和智能指针
C++ 中动态内存的管理是通过一对运算符`new`和`delete`实现的。`new`在动态内存中为对象分配空间，并且返回一个指向该对象的指针，可以选择对对象进行初始化；`delete`接收一个动态对象的指针，销毁该对象，释放和它相关的内存。

动态内存很难管理，有时候忘记释放内存，会产生内存泄露；有时候在有指针引用内存的情况下就释放了它，这种情况下产生非法引用的内存。
C++ 提供了两种智能指针，`shared_ptr`和`unique_ptr`管理动态对象。智能指针也是模板。因此，在创建智能指针的时候，需要提供类型信息。
`shared_ptr`允许多个指针指向一个对象，而`unique_ptr`则独占所指向的对象。
下面是`shared_ptr`和`unique_ptr`都支持的一些操作：
![smart_pointer](smart_pointer.png)

### `shared_ptr`

![shared_ptr](shared_ptr.png)

### `unique_ptr`
![unique_ptr](unique_ptr.png)

## 动态数组

### `new`

### allocate类

## 参考文献
