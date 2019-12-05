---
title: C++ type conversion
date: 2019-11-09 23:12:15
tags:
 - C/C++
categories: C/C++
---

## 类型转换
在C++类型中，某些类型之间有关联。如果说两种类型有关联，当程序需要其中一种类型的运算对象时，可以使用另一种类型关联的对象或者值来替代。或者说如果两种类型可以相互转换，那么他们就是关联的。
C++提供了两种类型相互转换的规则：
1. 隐式类型转换，隐式类型转换指的是编译器自动执行的，无需程序员的介入的转换。隐式类型转换包含上下文转换，值转换，算术提升，算术转换等。
2. 显式类型转换。

## 隐式类型转换
[13.3.3.1 Implicit conversion sequences]
什么时候会发生隐式类型转换？当一个类型为T1的表达式用在需要类型为T2的上下文时，会发生类型转换。比如：
- 函数声明中参数是T2类型的，传入参数是T1类型的；
- 将T1类型的表达式用作需要T2类型操作数的操作符。
- 用T1类型的表达式初始化一个T2类型的新对象，包含函数的返回语句。
- 当表达式被用在一个`switch`语句的条件中时（T2是整型）
- 表达式用于`if`的条件或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时或者循环语句的条件时（T2是`bool`类型）

只有存在从T1到T2的没有歧义的隐式类型转换时，这个程序才能编译成功。
 
### 转换的顺序
隐式类型转换序列如下所示：
1. 0或1个stand convesion sequence;
2. 0或1个用户自定义的转换;
3. 0或1个stand convesion sequence;

一个stand conversion sequence由以下顺序组成：
1. 0或1个左值转换
2. 0或1个数值提升或者数值转换
3. 0或1个函数指针转换（从C++17开始）
4. 0或1个qualification adjustment。

一个用户定义的转换由0个或者一个非显式的单个参数构造函数或者非显式的转换函数调用。
> An expression e is said to be implicitly convertible to T2 if and only if T2 can be copy-initialized from e, that is the declaration T2 t = e; is well-formed (can be compiled), for some invented temporary t. Note that this is different from direct initialization (T2 t(e)), where explicit constructors and conversion functions would additionally be considered.


### 上下文转换
在下面列出来的上下文中，期望获得一个`bool`类型的变量，如果声明语句`bool t(e)`能够编译成功，那么就进行隐式转换。这些表达式`e`被称为上下文转换成`bool`。
- 控制流`if`, `while`和`for`；
- 逻辑操作符`!`, `&&`, `||`的操作数；
- 条件操作符`?:;`的第一个操作数；

在下列上下文中，需要一个和上下文相关的类型`T`。
...

### 值转换
每一个表达式都有一个非引用类型和值类别，值转换发生在表达式的值类别上。无论何时当表达式的值类别时和操作符需要的操作数值类别不同时，发生这类转换。

#### 左值到右值转换


#### 数组到指针的转换

#### 函数到指针

### 算术提升
#### 整形提升
1. 整形提升。把整数类型转换成大整数类型。对于`bool`, `char`, `unisigned char`, `short`和`unsigned char`等类型来说，只要它们所有可能的值都能存在`int`里，就把它们提升成`int`类型，否则，提升成`unsigned int`类型。
#### 浮点数提升


### 算术转换
算术转换的含义是把一种算术类型转换成另外一种算术类型。

#### 整形转换

#### 浮点数转换

#### 浮点数到整形转换

#### 指针转换
2. 指针的转换。常量0或者`nullpt`能转换成任意类型的指针。指向非常量的指针能转换成`void *`类型。指向任意对象的指针能转换成`const void *`。
1. 数组转换成指针。将数组转换成指向数组首元素的指针。注意，数组名表示的指针是不可修改的。当数组作为`decltyp`参数，或者作为取值地址符，`sizeof`或者`typeid`的运算对象时，上述转换不会发生。同样，使用一个应用来初始化数组，上述转换也不会发生。
4. 转换成常量。允许将指向非常量的指针转换成相应类型的指针常量的类型，对应引用也是如此。

#### 指针到成员转换

#### 布尔转换
1. 转换成bool类型。存在一种从算术类型或者指针类型到布尔类型自动转换的机制。如果指针或者算术类型的值是0，那么转换结果是false，否则是true。
2. 无符号类型的运算对象。如果一个运算对象是无符号类型，另一个运算对象是有符号类型。
当无符号类型大于带符号类型，那么带符号类型转换成无符号。比如，`int`和`unsigned int`，将`int`转换成`unsigned int`类型，如果`int`是负值，将它转换成正值。相当于`int`对`unsigned int`所能所示的最大值取余。
当无符号类型小于带符号类型时，如果无符号类型的值能存放在有符号类型中，将无符号类型转换成带符号类型。否则将带符号类型转换成无符号类型。

#### 其他隐式类型转换
类类型定义的转换

## Qualification转换
### 函数指针转换

## bool变量的安全性问题

## 显式类型转换。
1. 命名的强制类型转换。它的形式如下所示：```c
cast-name<type>(expression)
```
其中`cast-name`是`static_cast`,`dynamic_cast`, `const_cast`和`reinterpret_cast`的一种，`type`是转换的目标类型，而`expression`是要转换的值。
2. `static_cast`，只要不包含底层`const`，任何具有明确定义的类型转换，都可以使用`static_cast`。比如将一个运算对象强制转换成`double`类型执行浮点数出发。```c
double slope = static_cast<double>(j)/i;
```
可以把较大的算术类型转换成较小的算术类型。这个时候，强制类型转换告诉读者和编译器，我不在乎可能的精度损失。
它对于编译器无法自动执行的类型转换也非常有用。
3. `const_cast`，它只能改变底层`const`，将常量对象转换成非常量对象，这种性质叫做去掉`const`性质。如果对象本身不是一个常量，使用强制类型转换获得写权限是一个合法的行为，如果对象是一个常量，使用`const_cast`执行写操作就会产生未定义的后果。
`const_cast`只能改变表达式的常量值，使用其他类型的命名强制类型转换改变表达式的常量属性都会引发编译器的错误。同样的，也不能使用`const_cast`改变表达式的类型。
通常用于有函数重载的上下文。
4. `reinterpret_cast`。尽量不使用强制类型转换，它干扰了正常的类型检查。在有重载的上下文中使用`const cast`无可厚非。但是在其他情况下使用`const_cast`也就意味着程序存在某种缺陷。其他的强制类型转换也不应该频繁使用。
5. 旧式的类型转换。旧式的类型转换分别具有和上述三种强制类型转换相同的结果。如果换成`const_cast`和`static_cast`也合法，其行为和对应的命名转换一样。如果替换后不合法，则旧式的强制类型转换执行和`reinterpret_cast`类似的功能。

总的来说，`static_cast`可以进行不包含底层`const`的具有明确定义的类型转换。
`cosnt_cast`可以去掉底层`const`的`const`，但是不能改变表达式类型，也不能对去掉const的常量表达式执行写操作。



## 参考文献
1.《C++ Primer》第五版
2.https://en.cppreference.com/w/cpp/language/implicit_conversion
3.https://en.cppreference.com/w/cpp/language/explicit_cast
4.https://en.cppreference.com/w/cpp/language/cast_operator
5.https://en.cppreference.com/w/cpp/language/const_cast
6.http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3242.pdf
7.https://en.cppreference.com/w/cpp/language/value_category
