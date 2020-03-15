---
title: C STL allocator
date: 2020-03-08 22:33:20
tags:
 - C/C++
categories: C/C++
---

## allocator
一般来说，C++ 对象的分配都是通过new expression来实现的，new expression通过调用operator new函数分配空间，然后调用相应的构造函数。同理释放的时候，通过delete，先调用operator delete，然后调用相应的析构函数。

STL allocator的把这两部分给分开，调用allocate()分配空间，然后调用construct()进行构建，construct()的话一般来说是负责调用构造函数。
而allocate的工作就要多一些，当然也可以少一些，比如直接调用operator new或者malloc分配。这样子的话可能效率要低一下，有内存碎片，overload比较大。
所以就有了下面的分配器。

## alloc
SGI alloc allocator的实现是一个两级空间配置器，第一级配置器负责分配128字节以上的空间，以及异常处理。第一级配置器使用malloc而不是operator new进行内存分配的原因，可能是C++没有realloc，因为没有使用new，所以就不能使用new_handler，需要自己实现。需要客户端自己设置一个oom_handler。
第二级分配器采用内存池的方式，维护16个链表，每个链表负责分配一个小于128字节的8字节整数倍（8, 16, ..., 128个字节）的小区块。初始时是它们都是空指针。（采用嵌入式指针节约空间）
接下来，如果要分配1个64字节大小对象的空间。它会向负责64字节的链表要空间，如果链表上没有足够的空间。它会向内存池要空间，而且要的不止一个，设置为20。如果内存池中有足够的空间，就返回，没有的话，满足一个也行，就返回一个对象。如果一个也没有的话，就向malloc要空间。malloc也没有的话，检测大于64字节的那些链表，是不是有空间。如果还没有的话，就调用第一级空间配置器，调用oom异常处理对象。


## 其他内存处理工具
这里使用到了type_traits进行重载。（还有哪里使用了type_traits？）
uninitialized_copy(InputForwardIterator, InputIterator, ForwardIteartor);
uninitialized_fill(ForwardIterator, ForwardIterator, const T& x);
uninitialized_fill_n(ForwardIterator , size_type n, const T& x )
这三个函数，会负责对相应的位置进行初始化，通过type_traits判断是否是POD类型，执行内存，或者是调用构造函数。

## 参考文献
1.《STL源码剖析》
