---
title: C `char*` vs `char**`
date: 2019-11-13 13:41:37
tags:
 - 指针
 - 指针数组
 - 数组指针
 - C
 - C/C++
categories: C/C++
---

##  char\*
1. `char*`是一个指针，指向一个`char`，理论上来说，它并不是一个数组。
2. `char *ptr;`并不为它指向的内容分配内存，而是分配一个`char *`的内存存放`ptr`。

## char\*\*
1. `char**`是一个指针，指向一个　指向`char`的指针。


## char arr[]
1. `char arr[10];`是一个数组，不是一个指针，`char *`和`char[10]`不是一个类型。
2. `char arr[10];`分配10个`char`的空间，`arr`的地址等于第一个字符的地址。

## 参考文献
1.
