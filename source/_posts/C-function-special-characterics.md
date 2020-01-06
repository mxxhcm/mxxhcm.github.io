---
title: C++ function special characterics
date: 2019-12-07 15:53:38
tags:
 - C/C++
 - 函数
categories: C/C++
---

## 特殊语言特性
这一节介绍三种函数相关的语言特性，它们分别是默认实参，内联函数和`constexpr`函数，以及程序调用过程中的一些常用功能。

## 默认实参
### 默认实参是什么
默认实参是函数声明时就指定形参的值。一般某个形参被赋予了默认值，它后面的所有形参都必须要有默认值。
尽量让不怎么使用默认值的形参出现在前面，让那些经常使用默认值的形参出现在后面。

### 默认实参的声明
一般情况下，一个函数只声明一次，但是声明多次也是可以的。但是在一个定的作用域内，每一个形参只能被赋予一次默认实参。``` c++
typedef string::size_type sz;
string screen(sz , sz w, char='*'); //指定形参char的默认实参
string screen(sz h=80, sz w=80, char); //指定形参h, w的默认实参。
```

### 默认实参初始值
局部变量不能当做实参的初始值，除此之外，只要表达式的类型都够转换为形参的类型，这个表达式就可以当做默认实参。

## 内联函数
函数调用的代价要比表达式的代价高。一次函数调用（生成一个新的函数栈帧）包含一系列寄存器的保存和恢复，以及可能的实参拷贝，程序跳转到一个新的位置执行等等。
但是函数调用要比表达式有一些好处：
1. 阅读和理解函数调用要比等价的表达式容易
2. 函数可以重复利用，不用重复编写，使用函数可以确保行为的统一，修改也比较方便（不用找到所有等价表达式出现的地方进行替换）。

所以就有了内联函数，在函数名字前加上`inline`关键字就可以声明一个内联函数。内联函数可以避免函数调用的开销还有了函数的优势，通常就是将它在每个调用点上展开，相当于直接用内联函数的定义替换这个函数的名字。

## `constexpr`函数
`constexpr`函数是指能用于常量表达式的函数。**定义`constexpr`函数的方法和其他函数一样，但是函数的返回值以及所有形参的类型都必须是字面值类型，而且函数中必须有且只有一条`return`语句。**
`constexpr`函数的返回值不一定是常量表达式。如果把它的返回值用在需要常量表达式的地方，编译器会进行检查。比如：```c++
```的

**通常将内联函数和`constexpr`函数定义在头文件中。**
内联函数允许多次定义，但是每次的定义必须一致，为什么？什么是内联函数，在每个调用点上将函数内联的展开。如果有两个文件foo.cpp和bar.cpp都需要调用一个相同的内联函数myinline，在编译这两个头文件的时候，需要将函数myinline进行展开，所以它们都需要定义myinline，但是在生成目标文件的时候，因为函数是强类型，就会出错，所以内联函数允许多次一致的定义。而最简单的方法就是将内联函数定义在头文件中，然后使用它的源文件包含它即可。


## `assert`预处理宏
`assert`定义在`<cassert>`头文件中。它使用一个表达式作为它的条件：```
assert(expr);
```
对`expr`求值，如果为真，什么也不做，否则输出信息，终止程序执行。它用来检查某种错误条件，即当`expr`满足条件时什么也不做，当它出错时，就结束。

## NODEBUG预处理变量
`assert`的行为依赖于一个名为`NDEBUG`的预处理器变量的状态。如果定义了`NDEBUG`，那么`assert`什么也不做。默认情况下没有定义`NDEBUG`，`assert`执行检查。

## 参考文献
1.《C++ Primer》第五版