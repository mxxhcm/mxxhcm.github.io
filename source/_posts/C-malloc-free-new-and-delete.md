---
title: C/C++ malloc(alloc) free new and delete
date: 2019-11-05 10:37:48
tags:
 - malloc
 - alloc
 - free
 - new
 - delete
 - C/C++
categories: C/C++
---

## malloc

## new
默认情况下，new分配的对象，不管是单个分配的还是数组中的，都是默认初始化的。这意味着内置类型或组合类型的对象的值是无定义的，而类类型对象将用默认构造函数进行初始化。
- 对于内置类型而言，new仅仅是分配内存，即使后面显示加(),相当于调用它的构造函数，```c
int *pi = new int; //pi指向一个动态分配的内存，没有初始化
```
- 对于自定义类型而言，只要一调用new，无论后面有没有加()，那么编译器不仅仅给它分配内存，还调用它的默认构造函数初始化。

## malloc vs new
0. `malloc`和`new`都是申请heap上的内存
1. `malloc`是C语言中的函数，而`new`是C++的操作符
2. `malloc`返回的是`void*`类型的指针，而`new`返回的指针是有类型的
3. `malloc`只负责内存的分配而不会调用类的构造函数，而`new`会分配内存，自动调用类的构造函数。

## 参考文献

