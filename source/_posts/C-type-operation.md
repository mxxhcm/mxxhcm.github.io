---
title: C++ type operation
date: 2019-11-13 14:19:46
tags:
 - auto
 - typedef
 - decltype
 - C/C++
categories: C/C++
---

## 类型操作
### 别名`typedef`和`using`
``` c
typdef dobuel wages;    //类型别名
using SI = Sales_imte;  //别名声明
```

#### 指针，常量和类型别名
```c
typedef char *pstring;
const pstring cstr = 0; //基本数据类型是char *，即指针类型
const pstring *p;
```
不能简单的把`pstring`用`char *`替换，如果替换了变成下式：
```c
const char *cstr = 0;   //基本数据类型是const char，
```
很容易把`*`看成是声明符的一部分，即`*cstr`的一部分，但是实际上`*`是和`const char`在一起的。`const char *cstr`是**指向`char`常量的指针**，而`const pstring cstr`是指向`char`的**常量指针**。

### `auto`关键字
编译器自动分析表达式的类型，`auto`定义的变量必须有初值。
使用一条`auto`语句可以声明多个变量，多个变量的基本数据类型必须一样。
```
auto i= 0, *p = &i; //正确
auto i = 1, d = 3.14;   //错误
```

#### 复合类型，常量和`auto`
1. 编译器使用`auto`推断出来的值和初始值类型有时候不完全一样。比如使用引用其实使用的是引用对象的值。
2. `auto`会忽略顶层`const`，保留底层`const`。如果希望`auto`推断出的是顶层`const`，需要显式的加一个`const`，即`const auto = ...`。
3. 将引用的类型设为`auto`也可以保留初始值中的顶层`const`属性，
4. 如果给初始值绑定一个引用，并且设为`auto`类型，这个对象就不是顶层`const`了。```c
int i = 0, &r = i;
auto a = r; // a是一个int,而不是int &

const int ci = i, &cr = ci;
auto b = ci;    //b是一个int
auto c = cr;    //c是一个int, cr是ci别名，ci是顶层const
auto d = &i;    //d是int *
auto e = &ci;   //e是一个指向const的指针。

const auto f = ci;  //f是const int
auto &g = ci;   // g是int &

const auto &j = 43;//这个j就不是顶层const了，类型是const int &，它是底层const。
```
### `decltype`类型
`decltype`和`auto`的区别：
1. 它只返回表达式的类型。
2. 它能识别顶层`const`和引用类型``` c
int i = 0, &ri = i, *pi = &i;
decltype(ri);       //是int &
decltype(ri+0);    //是int
const int ci = 0, &cj = ci;
decltype(ci) p = 0; //const int,
decltype(cj) q = 0; //const int &，顶层常量引用可以初始化成字面值
decltype(ci) x; //错误，常量必须初始化
decltype(cj) y; //错误，引用必须初始化
```
引用从来都是作为它所指对象的同义词出现，只有在`decltype`处是例外。

#### `decltype`和引用
1. 如果表达式的内容是解引用操作，使用`decltype`将会得到引用类型。
2. `decltype((variable))`的结果永远是引用，而`decltype(variable)`的结果只有在真的是引用的时候才会返回引用。``` c
int i = 0;
decltype((i)) d;
decltype(i) e;
```

## 参考文献
1.《C++ Primer第五版》
