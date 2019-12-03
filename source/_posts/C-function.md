---
title: C++ function
date: 2019-11-10 12:25:56
tags:
 - function
 - C/C++
categories: C/C++
---

## 函数
函数是一个命名了的代码块，可以通过调用函数执行相应的代码。函数可以有0个或者多个参数，通常会产生一个结果。C++可以重载函数，一个名字可以对应几个不同的函数。

### 调用运算符
通过**调用运算符**执行函数，调用运算符的形式是一对圆括号，它作用于一个表达式，该表达式是函数或者指向函数的指针，圆括号内是一个用逗号分隔开来的实参列表，我们用实参初始化函数的形参。调用功能表达式的类型就是函数的返回值类型。

### 函数调用和`return`
函数的调用主要完成两项工作，一个是用实参初始化函数的形参，一个是将控制权从主调函数（calling function）转交给被调用函数。执行函数的第一步是隐式的而定义并初始化它的形参。遇到一条`return`语句时函数结束执行过程。函数调用一样，`return`语句也完成两项工作，一个是返回`return`语句中的值，一个是将控制权从被调用函数转交给主调函数。函数的返回值用于初始化调用表达式的结果，完成调用所在表达式的声誉部分。

### 形参和实参
实参是形参的初始值。实参的类型必须和对应的形参相匹配，或者可以经过隐式的类型转换转换成形参的类型。

### 函数的形参列表
隐式的定义空形参列表```c
void f1()
```
显式的定义空形参列表```c
void f2(void)
```

### 函数返回值类型
1. 函数的返回值不能是数组类型或者函数类型，但是可以是指向数组或者函数的指针。
2. 函数的返回值可以是一种特殊类型`void`。

## 局部对象
### 作用域和生命周期
在C++中，名字有作用域，对象具有生命周期：
- 名字的作用域是程序文本的一部分。
- 对象的生命周期是程序执行过程中该对象存在的一段时间。

函数体是一个语句块，块构成一个新的作用域，可以在其中定义变量。形参和函数体内的变量统称为局部变量。它们对于函数而言是局部的，仅在函数的作用域内可见，同时局部变量还会隐藏在外层作用域中所有同名的其他声明。
定义在所有函数体之外的对象存在于程序执行的整个过程中。这类对象在程序启动时被创建，到程序结束时被销毁。

### 自动对象
对于普通局部变量对应的对象来说，函数的控制流经过变量定义语句时创建该对象，到达定义所在的块末尾时销毁它，**只存在于块执行期间的对象称为自动变量**。当块的执行结束时，块中创建的对象的值也就变成未定义的了。
形参是一种自动对象，函数开始时为形参申请存储空间，一旦函数终止，形参就被销毁。
对于局部变量对应的自动对象来说，如果变量本身有初始值，使用这个初始值进行值初始化；否则，执行默认初始化。这就意味着内置类型的未初始化的局部变量具有未定义的值。

### 局部静态对象
局部静态对象在程序的执行路径第一次经过对象定义语句时初始化，直到程序终止时被销毁，在此期间即使对象所在的函数结束执行也不会对它有影响。
如果没有执行局部静态变量的值，执行值初始化，内置类型的静态局部变量被初始化为0。

### 在头文件中进行函数声明
和变量在头文件中声明，在源文件中定义类似。函数也应该在头文件中声明而在源文件中定义。定义函数的源文件应该把含有函数声明的头文件包含进来，编译器负责验证函数的定义和声明是否匹配。

### 分离式编译
如何编译和链接多个头文件。查看[C/C++分离式编译]()。

## 参数传递
形参初始化的机理和变量初始化一样。形参的类型决定了形参和实参交互的方式。如形参是引用类型，它将绑定到对应的实参上；否则，将实参的值拷贝后赋给形参。
当形参是引用类型时，我们说它对应的实参被**引用传递**，或者函数被**传引用调用**。和其他引用一样，引用形参也是它绑定的对象的别名；也就是说，引用形参是它对应的实参的别名。
当实参的值被拷贝给形参时，形参和实参是两个相互独立的对象。我们说这样的实参被**值传递**或者函数被**传值调用**。

因为C中没有引用，所以C中传递参数的方式只有值传递，而C++中多引用，不仅有值传递，还有引用传递。

## `return`和返回值类型
### 无返回值

### 有返回值

### 返回数组指针

## 函数重载

## 特殊函数

## 函数匹配

## 函数指针

## 参考文献
1.