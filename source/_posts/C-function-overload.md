---
title: C++ function overload
date: 2019-12-06 15:34:29
tags:
 - C/C++
 - 函数
 - 重载
categories: C/C++
---

## 函数重载
如果同一个作用域内的几个函数名字相同但是形参列表不同，被称为重载函数。`main`函数不能被重载。

## 定义重载函数
如下示例，定义了几个重载函数。重载函数的区别在于形参的类型和个数不同。形参可以有名字可以没有名字，也可以有不同的名字，但只要类型和个数相同，就是同一个函数。
``` c++
void print(const int i);
void print(const string &s);
void print(const vector<string> &vs);
void print(const set<int> &si);
void print(const map<string, int> &msi);
```

## 重载和`const`形参
顶层`const`不影响传入函数的对象
1. 一个拥有顶层`const`的形参无法和另一个没有顶层`const`的形参区分开来。
2. 如果形参是某种类型的指针或引用，即形参是底层`const`，区分其指向的是常量对象还是非常量对象可以实现函数重载。

## `const_cast`和重载
`const_cast`在重载函数的情景中有最有用。如下函数：``` c++
const string& stringCompare(const string &s1, const string &s2);
```
函数的形参是底层`const`，可以接收常量或者非常量的实参。但是返回的都是`const string`的引用，如果输入是两个`string`的引用，返回一个`const string`的引用显然是不合理的，这时候就可以使用`const_cast`了。对上述代码做一个改进：```c++
string& stringCompare(string &s1, string &s2)
{

    auto &r = stringCompare(static_cast<const string &>(s1), static_cast<const string &>(s2)a);
    return static_cast<string &>(r);
}

```

## 调用重载函数
函数匹配是指将函数调用功能和重载函数中的某一个关联起来。编译器根据调用的实参和重载集合中每个函数的形参作比较。调用功能重载函数有三种可能结果：
1. 找到一个与实参完全匹配的函数，生成调用功能该函数的代码。
2. 找不到任何一个函数和调用的实参匹配，这个时候编译器发出无匹配的错误
3. 多于一个函数可以匹配，但是每一个都不是最佳选择，这种错误叫做二义性调用。

## 重载和作用域
正常来说，将函数声明放在局部作用域内是一个不明智的选择。如果真的这么做了，编译器会首先在内层作用域寻找函数的声明，如果找到了，它会隐藏外部作用域的所有同名声明。

## 参考文献
1.《C++ Primer》第五版

