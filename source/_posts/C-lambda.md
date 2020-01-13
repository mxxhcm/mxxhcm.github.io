---
title: C++ lambda
date: 2020-01-08 19:50:10
tags:
 - C/C++
 - lambda
categories: C/C++
---

## lambda
lambda是一种可调用对象，它具有如下的形式：``` c++
[](param list)->return type {function body};
```

其中`[]`叫做capture list，(param list)是参数列表，return type是返回值类型，{function body}是函数体。
capture list和function body不能忽略，其他的部分都可以忽略。

## 向lambda传递参数
capture list和param list可以传递参数。capture list传递的是当前函数内的局部变量，表示lambda未来会使用这些变量，param list是lambda接受的参数。
capture list内可以是value capture，也可以是reference capture。

## 隐式capture
capture list中指定=或者&可以自动推断lambda中可能用到的局部变量。隐式capture和显式capture可以同时存在。

## 指定返回类型
如果lambda中包含超过一个return之外的语句，编译器假定它返回void，然后就不会return了。lambda可以指定返回类型：``` c++
[](param list)->return type {function body};
```

## 参考文献
1.《C++ Primer》第五版中文版
