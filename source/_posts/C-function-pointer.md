---
title: C/C++ function pointer
date: 2019-12-06 15:34:54
tags:
 - C/C++
 - 指针
 - 函数
categories: C/C++
---

## 函数指针
函数指针指向的是函数而不是对象。和其他指针一样，函数指针指向某种特定类型。函数的类型由它的返回值和形参共同决定，和函数名无关。例如：``` c++
bool lengthCompare(const string &, const string &);
```
这个函数的类型是`bool(const string &, const string &)`，要想声明一个指向该函数的指针，只需要使用指针代替函数名字即可：``` c++
bool (*pf)(const string &, const string &);
```
从声明符中的变量名字开始，`pf`前面有个`*`，所以`pf`是个指针，右侧是形参列表，左侧是函数的返回值类型。因此，`pf`是一个指向函数的指针，函数的参数是两个`const string`的引用，返回值是`bool`类型，指针类型是`boo(*)(const string &, const string &)`

## 使用函数指针
**当我们把函数名作为一个值使用时，这个函数自动的转换成指针。**如下所示的两个语句是等价的：``` c++
pf = lengthCompare;
pf = &lengthCompare;
```
同时，可以直接使用指向函数的指针调用函数，而不需要进行解引用运算。下面的三个语句是等价的：``` c++
bool b1 = pf("hello", "world");
bool b2 = (*pf)("hello", "world");
bool b3 = lengthCompare("hello", "world");
```

指向不同函数类型之间的指针不存在转换规则，但是可以为函数指针赋值`nullptr`或者0，表示它不指向任何函数。

## 重载函数的指针
当函数重载之后，上下文必须能够告诉编译器到底选择哪个函数，指针必须和重载函数中的某一个精确匹配。

## 函数指针形参
和数组类似，虽然不能定义函数类型的形参，但是形参可以是指向函数的指针。这个时候，形参看起来是函数类型，实际上是当成指针使用。
可以直接把函数作为实参使用，这个时候它会自动的转换成指针：``` c++
//第三个参数是函数类型，它会自动的转换成指向函数的指针
void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &));
//第三个参数是显式声明的指向函数的指针
void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &));
// 函数lengthCompare会被自动的转换成函数指针
useBigger(s1, s2, lengthCompare);
```

### 使用`typedef`和`delctype`简化函数指针
可以使用下列语句定义函数指针：``` c++
//下面两个是函数类型：
typedef bool Func(const string&, const string&);
typedef delctype(lengthCompare) Func2; 


//下面两个是函数指针类型：
typedef bool (*FuncP)(const string&, const string&);
typedef delctype(lengthCompare) *FuncP2; 
```

需要注意的是，`decltype`返回函数类型，不会将函数自动转换成指针类型，只有在前面加上`*`才能得到指针。


## 返回指向函数的指针
和数组类似，虽然不能返回一个函数，但是可以返回指向函数类型的指针。**然后，我们必须把返回类型手动写成指针，编译器不会自动的将函数返回类型当成对应的指针类型处理。**最好的办法是使用类型别名：``` c++
using F = int(int*, int);   //F是函数，不是指针
using PF = int(*)(int*, int);   //PF是指针类型
```
必须注意的是，和函数类型的形参不一样，返回类型不会自动的转换成指针，我们必须显式的将返回类型指定为指针。

## `auto`和`decltype`作用于函数指针
将`decltype`作用于某个函数时，它返回函数类型而非指针类型，需要我们显式的加上`*`表示我们需要返回指针，而非函数本身。当`decltype`作用于函数指针时，它返回的是函数指针类型。

## 参考文献
1.《C++ Primer》第五版

