---
title: C/C++ problems
date: 2019-12-10 19:09:03
tags:
 - C/C++
 - 问题
categories: C/C++
---

## 问题1-C中为什么`const int`不能当做array size
> The const qualifier really means ``read-only''; an object so qualified is a run-time object which cannot (normally) be assigned to. The value of a const-qualified object is therefore not a constant expression in the full sense of the term, and cannot be used for array dimensions, case labels, and the like. (C is unlike C++ in this regard.) When you need a true compile-time constant, use a preprocessor #define (or perhaps an enum).

在C语言中，`const`修饰只代表只读，被`const`修饰的对象是一个运行时对象，通常不能被赋值。因此它不是一个常量表达式，这点和C++ 不一样。可以使用`#define`或者`enum`。
什么是常量表达式，在编译时就能获得它的值，而不是运行时。
> A constant expression can be evaluated during translation rather than runtime, and accordingly may be used in any place that a constant may be.[ISO C11 Sec 6[ISO C11 Sec 6.6]

## 问题2-cc1和gcc的关系
C语言程序从ASCII转换到可执行目标文件分别经历了预处理器cpp，编译器cc1，汇编器as，以及链接器ld的处理，一个编译器驱动程序包含所有这几个部分，gcc是GNU编译系统上的编译驱动程序。
我在ubuntu 18.04上没有找到cc1，我的做法：``` shell
which gcc       # 查看gcc存放位置
#我的输出是: 
#/usr/bin/gcc
gcc --version # 查看当前gcc 版本
#我的输出是: 
#gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0,,,
cd /usr
sudo find . -name "cc1"
#我的输出是：
#./lib/gcc/x86_64-linux-gnu/7/cc1
#./lib/gcc/x86_64-linux-gnu/6/cc1
#然后根据gcc版本7.4，进入第一个目录

cp cc1 /usr/bin # 将它拷贝到和gcc相同的目录
```


## 参考文献
问题1
1.https://stackoverflow.com/a/44268465/8939281
2.https://stackoverflow.com/a/18848583/8939281
3.http://c-faq.com/ansi/constasconst.html
问题2
4.https://unix.stackexchange.com/questions/77779/relationship-between-cc1-and-gcc
5.https://stackoverflow.com/a/51218063/8939281

