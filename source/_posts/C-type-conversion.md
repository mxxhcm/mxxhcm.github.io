---
title: C++ type conversion
date: 2019-11-09 23:12:15
tags:
 - C/C++
categories: C/C++
mathjax: true
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
任何非函数，非数组类型T的glvalue可以隐式的转换成相同类型的prvalue。如果T是非类类型的话，这个转换还会去掉cv修饰。
在以下情况中，对象表示的glvalue是不可访问的：
- 转化发生在无法评估的上下文，比如`sizoef`, `decltype`或者静态形式的的`typid`。
- 


#### 数组到指针的转换

#### 函数到指针

### 算术提升
#### 整形提升
小整形（如`char`）的prvalue可以转换为大整形（如`int`）的prvalues。特别的，算术操作符不接受比`int`还小的类型作为参数，在使用了lvalue到rvalue的转换之后，自动的进行整形提升。这种转换不会丢失精度。
整形提升可以分为以下几类：
- `signed char`或者`signed short`可以被转换成`int`；
- `unsigned char`或者`unsigned short`被转化成`int`，如果超过`int`的表示范围，转换成`unsigned int`；
- `char`被转换成`int`还是`unsigned int`取决于它是`signed char`还是`unsigned char`；
- `w_char_t`, `char16_t`, `char32_t`可以被转换成下列第一个可以容纳它们整个取值范围的类型：`int`, `unsigned int`, `long`, `unsigned long`, `long long`, `unsigned long long`；（有个疑问，`wchar_t`等是有符号的还是无符号的？？）

总的来说，就是对于`bool`, `char`, `unisigned char`, `short`和`unsigned char`等类型来说，只要它们所有可能的值都能存在`int`里，就把它们提升成`int`类型，否则，提升成`unsigned int`类型。
并不是所有的转换都是提升，当`char`到`short`就是一个转换。


#### 浮点数提升
一个`float`类型的prvalue可以转换成`double`的prvalue。值不会改变，精度不会丢失。

### 算术转换
算术转换的含义是把一种算术类型转换成另外一种算术类型。算术转换可能会改变对象的值，丢失一定精度。

#### 整形转换
- 如果目标类型是无符号的，转换结果是source value对$2^n$取余得到的最小无符号整数，n是目标类型的位数。
- 如果目标类型是有符号的，如果source整数能够保存在目标类型中，对象的值不变。否则转换的结果是由实现定义的（注意和有符号整形算术溢出的区别，溢出是未定义行为）。
- 如果source type是`bool`，值`false`被转换为0，值`true`被转换成destination类型的值1（注意，如果destination类型是`int`的话，就变成了整形提升）。
- 如果目标类型是`bool`，这就是一个boolean转换

整形转换的实现，可以查看[]()。基本上所有的C/C++ 实现就是不改变二进制编码，分别使用无符号编码和补码形式进行解释这个二进制编码。
相同长度的无符号数和有符号数，在运算中有有符号数的话，就会把有符号数转换成无符号数。比如for循环中，`for(unsigned i = 10; i >= 0; --i)`，永远不会跳出for循环，因为当i=0时，`--i`就相当于得到了有符号数`-1`，而`-1`的补码形式和无符号数的$2^w -1 $的编码是一样的，而这里就是得到了$2^w -1$。

#### 浮点数转换
一个浮点数类型的prvalue可以被被转换成任何其他浮点数类型的prvalue。如果这个转换是浮点数提升，那么这就是一个浮点数提升而不是一个浮点数转换。
- 如果source value可以精确的表示在目标类型中，这个值不变。
- 如果source value在目标类型的两个可表示值之间（就是目标类型的精度比较低），则结果是其中的一个（由实现定义）。
- 否则是未定义行为。

#### 浮点数和整形之间的转换
- 一个浮点数类型的prvalue可以转换成一个整形的prvalue，小数部分被截断。如果这个值不适合目标类型，是未定义行为。
- 一个整形的prvalue或者没有范围的枚举类型可以被转换成浮点数类型的prvalue。如果这个值不能被正确表示，由实现定义选择。如果这个值不适合目标类型，是未定义行为。如果source type是`bool`，值`false`被转换成0，值`true`被转换成1。

#### 指针转换
1. 指针的转换。常量0或者`nullpt`能转换成任意类型的指针。指向非常量的指针能转换成`void *`类型。指向任意对象的指针能转换成`const void *`。
2. 数组转换成指针。将数组转换成指向数组首元素的指针。注意，数组名表示的指针是不可修改的。当数组作为`decltyp`参数，或者作为取值地址符，`sizeof`或者`typeid`的运算对象时，上述转换不会发生。同样，使用一个应用来初始化数组，上述转换也不会发生。
3. 转换成常量。允许将指向非常量的指针转换成相应类型的指针常量的类型，对应引用也是如此。

#### 指针到成员转换

#### 布尔转换
1. 转换成bool类型。存在一种从算术类型或者指针类型到布尔类型自动转换的机制。如果指针或者算术类型的值是0，那么转换结果是false，否则是true。
2. 无符号类型的运算对象。如果一个运算对象是无符号类型，另一个运算对象是有符号类型。
当无符号类型大于带符号类型，那么带符号类型转换成无符号。比如，`int`和`unsigned int`，将`int`转换成`unsigned int`类型，如果`int`是负值，将它转换成正值。相当于`int`对`unsigned int`所能所示的最大值取余。
当无符号类型小于带符号类型时，如果无符号类型的值能存放在有符号类型中，将无符号类型转换成带符号类型。否则将带符号类型转换成无符号类型。

#### 其他隐式类型转换
类类型定义的转换


### Qualification转换
#### 函数指针转换

### bool变量的安全性问题

## 显式类型转换。
命名的强制类型转换。它的形式如下所示：```c
cast-name<type>(expression)
```
其中`cast-name`是`static_cast`,`dynamic_cast`, `const_cast`和`reinterpret_cast`的一种，`type`是转换的目标类型，而`expression`是要转换的值。

### `static_cast`
`static_cast`，只要不包含底层`const`，任何具有明确定义的类型转换，都可以使用`static_cast`。比如将一个运算对象强制转换成`double`类型执行浮点数出发。```c
double slope = static_cast<double>(j)/i;
```
可以把较大的算术类型转换成较小的算术类型。这个时候，强制类型转换告诉读者和编译器，我不在乎可能的精度损失。
它对于编译器无法自动执行的类型转换也非常有用。

### `const_cast`
0. `const_cast`只能改变运算对象的底层`const`，将常量对象转换成非常量对象，这种性质叫做去掉`const`性质。如果对象本身不是一个常量，使用强制类型转换获得写权限是一个合法的行为，如果对象是一个常量，使用`const_cast`执行写操作就会产生未定义的后果。`const_cast`还可以将一个非`const`对象变成`const`对象。```c
string s1("hello"); 
const string s2("world");

string *p1 = &s1;
const string *p2 = &s2;

string *p3 = const_cast<string *>(p2);  //去掉底层const，但是通过p3写它指向的东西是未定义行为。
const string *p4 = const_cast<const string*>(p1);   //将非底层const转换成底层const。
```
2. 只有`const_cast`能改变表达式的常量属性，使用其他类型的命名强制类型转换改变表达式的常量属性都会引发编译器的错误，注意不能使用`const_cast`改变表达式的类型。
3. 通常用于有函数重载的上下文。

### `reinterpret_cast`
通常用来进行指针类型的转换。
`reinterpret_cast`。尽量不使用强制类型转换，它干扰了正常的类型检查。在有重载的上下文中使用`const cast`无可厚非。但是在其他情况下使用`const_cast`也就意味着程序存在某种缺陷。其他的强制类型转换也不应该频繁使用。

### 旧式的类型转换。
旧式的类型转换分别具有和上述三种强制类型转换相同的结果。如果换成`const_cast`和`static_cast`也合法，其行为和对应的命名转换一样。如果替换后不合法，则旧式的强制类型转换执行和`reinterpret_cast`类似的功能。


### 总结
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
