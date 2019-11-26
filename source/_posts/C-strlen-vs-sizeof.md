---
title: C/C++ strlen vs sizeof
date: 2019-11-26 10:07:37
tags:
 - C/++
categories: C/++
---


1. `strlen`不计算字符串数组的null字节，而`sizeof`会计算null字节所占的字节。
2. `strlen`是一个函数，使用时需进行一次系统调用。而`sizeof`会在编译时计算。


## 参考文献
1.《APUE》第三版8.3节
