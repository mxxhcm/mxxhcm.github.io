---
title: C++ expression
date: 2019-11-10 12:15:22
tags:
 - C/C++
 - unary operator
 - binary operator
categories: C/C++
---

## 表达式(expression)
### 什么是表达式
由一个或者多个运算对象组成，对表达式求值将得到一个结果，字面值和变量是最简答的表达式，其结果就是字面值和变量的值。把一个运算符和多个运算对象组合起来可以生成比较复杂的表达式。

### 运算类型转换
简单来说，类型转换的规则大多复合情理。比如整形转换成浮点型，浮点型转换成整形。但是指针不能转换成浮点型。小整数（如`bool`, `char`, `short`）通常会被提升成较大的整数类型，通常是`int`。
在后面的小节会详细总结一下规则。

### 运算符重载
C++定义了运算符作用于内置对象和复合类型的运算对象时所执行的操作。当运算符作用于类类型的时候，用户可以自行定义它的含义。这种自定义的过程事实上是为了已存在的运算符附上了另外一层含义，所以称为运算符重载。

IO库中的>>和<<运算符以及`string`对象，`vector`对象和迭代器使用的运算符都是重载的运算符。


### 左值和右值
当对象被用作右值的时候，用的是对象的值。当对象被用作左值的时候，使用的是对象的身份（在内存中的位置）。

## 算术运算符
1. 算术类型的运算对象和求值结果都是右值。
2. 算术运算符包含加，减，乘，除，取余和正负号。
3. 算术运算符能够作用于任何算术类型以及任何能转换成算术类型的类型。
4. 一元正号运算符，加法运算符和减法运算符都能作用于指针。当一元正号运算符作用于一个指针或者算术值时，返回运算对象值的一个提升后的副本。
5. 在表达式求值前，小整数的类型会被提升成较大的整数类型，所有运算对象最后会转化成同一种类型。
6. 如果把一个整数赋值给bool对象，只有当这个数是0是才是false，否则都是真，比如-1给一个bool对象也是真。**所以布尔对象不应该参与运算。**
7. 整数相除还是整数。如果两个运算对象符号相同，则商为正，否则为负。
8. 取余运算的运算对象都必须是整数。取余结果的符号和被除数相同。


## 逻辑运算符和关系运算符
逻辑运算符：与，或，非

关系运算符：等于，不等于，大于，小于，大于等于，小于等于
逻辑运算符和关系运算符的求值结果都是布尔值。

## 赋值运算符
赋值运算的结果是它的左侧运算对象，是一个左值。
赋值可以使用初始化列表。
对于内置类型来说，初始值列表只能包含一个值，而且该值即使转换的话其所占空间也不应该大于目标类型的空间。
对于类类型来说，赋值运算的细节由类本身决定。
对于vector来说，vector模板重载了赋值运算符，并且可以接收初始值列表，当赋值发生时用右侧运算对象的元素替换左侧运算对象的元素。
无论左侧运算对象的类型是什么，初始化列表都可以为空。这个时候使用值初始化进行赋值。

## 递增和递减运算符
这两种运算符的运算对象必须是左值运算对象，前置版本将对象本身作为左值返回，后置版本将对象原始值的副本作为右值返回。

## 成员访问运算符
箭头运算符作用于一个指针类型的对象时，结果是一个左值。

## 条件运算符
当条件运算符的两个表达式都是左值的时候或者能转换成同一种左值类型时，运算的结果是左值，否则运算的结果是右值。

## 位运算符

## `sizeof`运算符
sizoef运算符返回一条表达式或者一个类型名字所占的字节数。sizeof运算符满足右结合律，所得的值是一个`size_t`类型的常量表达式。

`sizeof`运算的结果部分的依赖于它作用的类型。
- 对`char`或者类型为`char`的表达式执行`sizeof`运算符，得到1
- 对引用类型执行`sizeof`运算得到被引用对象所占空间的大小
- 对指针执行`sizeof`运算得到指针本身所占空间的大小。
- 对解引用指针执行`sizeof`运算得到指针指向对象所占空间的大小，指针不需要有效。
- 对数组执行`sizeof`运算得到整个数组所占空间的大小，等价于对数组中所有的元素各执行一个`sizeof`运算所得结果之和。`sizeof`并不会把数组转换为指针进行运算。
- 对`string`或者`vector`对象执行`sizeof`运算只返回该类型固定部分的大小，不会计算对象中的元素占用了多少空间。

## 逗号运算符

## 强制类型转换
在C++类型中，某些类型之间有关联。如果两种类型有关联，那么当程序需要其中一种类型的运算对象时，可以使用另一种类型关联的对象或者值来替代。或者说，如果两种类型可以相互转换，那么他们就是关联的。

隐式类型转换，一些类型转换是自动执行的，无需程序员的介入，有时甚至不需要程序员了解，所以叫做隐式类型转换。在发生以下情况时，编译器会自动的转换运算对象的类型：
- 比int小的整型值首先提升为较大的整形值值。
- 在条件中，非bool类型转为成bool类型。
- 初始化过程中，初始化值转换成变量的类型。在赋值语句中，右侧对象转换成左侧运算对象的类型。
- 如果算术运算或者逻辑运算的对象有多种类型，需要转换成同一种类型。
- 函数调用时有时候也会发生类型转换。

不同的类型之间是有可能进行相互转换的，并且有不同的规则进行类型转换，主要有以下几种规则：
1. 算术转换，针对于内置类型
2. 其他隐式类型转换
3. 显式类型转换。


### 算术转换，针对于内置类型
算术转换的含义是把一种算术类型转换成另外一种算术类型。

1. 整形提升。把整数类型转换成大整数类型。对于`bool`, `char`, `unisigned char`, `short`和`unsigned char`等类型来说，只要它们所有可能的值都能存在`int`里，就把它们提升成`int`类型，否则，提升成`unsigned int`类型。
2. 无符号类型的运算对象。如果一个运算对象是无符号类型，另一个运算对象是有符号类型。
当无符号类型大于带符号类型，那么带符号类型转换成无符号。比如，`int`和`unsigned int`，将`int`转换成`unsigned int`类型，如果`int`是负值，将它转换成正值。相当于`int`对`unsigned int`所能所示的最大值取余。
当无符号类型小于带符号类型时，如果无符号类型的值能存放在有符号类型中，将无符号类型转换成带符号类型。否则将带符号类型转换成无符号类型。


### 其他隐式类型转换
1. 数组转换成指针。将数组转换成指向数组首元素的指针。注意，数组名表示的指针是不可修改的。当数组作为`decltyp`参数，或者作为取值地址符，`sizeof`或者`typeid`的运算对象时，上述转换不会发生。同样，使用一个应用来初始化数组，上述转换也不会发生。
2. 指针的转换。常量0或者`nullpt`能转换成任意类型的指针。指向非常量的指针能转换成`void *`类型。指向任意对象的指针能转换成`const void *`。
3. 转换成bool类型。存在一种从算术类型或者指针类型到布尔类型自动转换的机制。如果指针或者算术类型的值是0，那么转换结果是false，否则是true。
4. 转换成常量。允许将指向非常量的指针转换成相应类型的指针常量的类型，对应引用也是如此。
5. 类类型定义的转换


### 显式类型转换。
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

## 运算符的优先级表

## 参考文献
1.《C++ Primer》第五版

