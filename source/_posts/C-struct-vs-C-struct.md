---
title: C struct vs C++ struct
date: 2019-10-25 19:32:07
tags:
 - C/C++
categories: C/C++
---

1. 在C中，struct names也在定义的`struct`的namespace中，如果定义`struct Foo {};`类型，如果需要创建这种类型的变量，
在C中需要使用`struct Foo foo;`
而在C\+\+中只需要写 `Foo foo;` ，当然C那样子也是支持的。
而对于C语言程序员，可以使用 `typedef struct {} Foo;` ，然后像C\+\+那样创建变量。
2. 在C\+\+中，`struct`和`class`其实是相同的东西，除了`struct`的方法默认都是`public`的，`class`的方法默认都是`private`的。
而C中，结构体不能拥有method。
3. C中`struct`不能有static成员，而C++的`struct`可以
4. C中`struct`不能有构造函数，而C++的`struct`可以有构造函数
5. C中sizeof `struct Foo {};`是1，而C++中是0


## 参考文献
1.https://www.quora.com/What-is-the-difference-between-C-structure-C++-structure
2.https://www.geeksforgeeks.org/difference-c-structures-c-structures/
3.https://stackoverflow.com/questions/2242696/c-structure-and-c-structure
