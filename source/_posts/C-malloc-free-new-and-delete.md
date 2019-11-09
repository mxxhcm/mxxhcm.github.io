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

## malloc vs new
1. `malloc`是C语言中的函数，而`new`是C++的操作符；
2. 对于自定义类型而言，operator new首先调用operator new函数申请内存，然后调用类的构造函数进行初始化，最后返回自定义对象的指针；`malloc`只负责内存的分配而不会调用类的构造函数数。
3. `malloc`返回的是`void*`类型的指针，需要进行强制类型转换转换成我们需要的类型，而`new`返回的是对象类型的指针，类型和对象严格匹配，`new`是类型安全型操作符。
4. 在分配内存失败时，`malloc`会返回NULL，而`new`会throw on failure。
5. `malloc`需要指定申请的内存占多少个字节，而`new`不需要指定申请内存块的大小，编译器会根据类型计算需要的内存大小；
6. `malloc`和`new`都是申请heap上的内存；

几个问题：
1. 什么时候`malloc`和`new`会申请内存失败。
2. `new`操作符的两个步骤，一个是申请内存，一个是调用构造函数，`new`的申请内存和`malloc`的申请内存有什么区别。

## new
默认情况下，new分配的对象，不管是单个分配的还是数组中的，都是默认初始化的。这意味着内置类型或组合类型的对象的值是无定义的，而类类型对象将用默认构造函数进行初始化。
- 对于内置类型而言，new仅仅是分配内存，即使后面显示加(),相当于调用它的构造函数，```c
int *pi = new int; //pi指向一个动态分配的内存，没有初始化
```
- 对于自定义类型而言，只要一调用new，无论后面有没有加()，那么编译器不仅仅给它分配内存，还调用它的默认构造函数初始化。

## 参考文献
1.《C++ Primer第五版》
2.https://stackoverflow.com/questions/184537/in-what-cases-do-i-use-malloc-and-or-new
3.https://zhuanlan.zhihu.com/p/47089696?utm_source=wechat_session&utm_medium=social&utm_oi=687606928481730560
