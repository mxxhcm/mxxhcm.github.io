---
title: C/C++ program in memory
date: 2019-10-19 20:41:02
tags:
 - C/C++
 - memeory
categories: C/C++
---

## C语言程序在内存中的组成部分
C语言程序在内存中的构成如下所示：
![c_program_in_memory](c_in_memory.jpg)
即一个C语言程序在内存中由text segment, data segment，heap和stack组成

## Text segment （代码段）
Text Segment中存放的是代码，只读，固定大小。代码段也可能会放在heap或者stack下面，当它们溢出时就会覆盖代码段。

## Data segment （数据段）
数据段分为两部分bss和data两部分。data中存放的是初始化数据，bss中存放的是未初始化数据。bss和data都是静态内存。

## Heap （堆）
堆是动态分配的内存，从低地址往高地址增长，存放的是用户手动分配的内存，即malloc()和calloc()等函数生成的数据存放地址。

## Stack （栈）
栈也是动态分配的内存，从高地址向低地址增长，栈中存放的是临时数据：函数参数，返回地址，局部变量等等。可以用来保存函数调用的现场，在递归调用中，如果调用次数太多，就可能会有stack overflow问题。


## 参考文献
1.https://www.geeksforgeeks.org/memory-layout-of-c-program/
2.https://www.tutorialspoint.com/memory-layout-of-c-programs
