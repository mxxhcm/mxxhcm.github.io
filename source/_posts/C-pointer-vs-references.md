---
title: C pointer vs references
date: 2020-02-22 01:06:24
tags:
 - C/C++
categories: C/C++
---

1. 指针可以被重新赋值。而引用一旦被绑定就不能修改。
2. 指针本身是一个变量，有它自己的内存地址，以及在栈上的大小。而引用的内存地址和它绑定的对象一样，看起来是一样的，但是实际上是不一样的（编译器做了一些操作），可以把引用当做它引用对象的别名。
3. 可以定义指针的指针，但是不能（直接）定义引用的引用（模板参数推导的时候可以）。
4. 指针可以为空，引用不能。
5. 指针变量可以进行运算，而引用不能。
6. 指针必须被解引用才能使用，而引用不需要。
7. const引用可以指向临时变量，而指针不能。
8. 指针本身可以是const的，而引用不能。底层const和顶层const。

## 参考文献
1.https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable-in
