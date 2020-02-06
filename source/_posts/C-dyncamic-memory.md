---
title: C++ dyncamic memory
date: 2019-11-10 12:39:41
tags:
 - C/C++
categories: C/C++
---

## 概述
每一个C程序都把内存划分成静态内存，栈内存，堆内存（自由空间）。静态内存存放局部static对象，类的static数据成员以及定义在任何函数之外的变量。栈存放函数内的非static对象。堆内存是由程序员自己负责管理（申请和释放）的内存。

## 动态内存和智能指针
C++ 中动态内存的管理是通过一对运算符`new`和`delete`实现的。`new`在动态内存中为对象分配空间，并且返回一个指向该对象的指针，可以选择对对象进行初始化；`delete`接收一个动态对象的指针，销毁该对象，释放和它相关的内存。

动态内存很难管理，有时候忘记释放内存，会产生内存泄露；有时候在有指针引用内存的情况下就释放了它，这种情况下产生非法引用的内存。
C++ 提供了两种智能指针，`shared_ptr`和`unique_ptr`管理动态对象。智能指针也是模板。因此，在创建智能指针的时候，需要提供类型信息。
`shared_ptr`允许多个指针指向一个对象，而`unique_ptr`则独占所指向的对象。
下面是`shared_ptr`和`unique_ptr`都支持的一些操作：
![smart_pointer](smart_pointer.png)

## `shared_ptr`
![shared_ptr](shared_pointer.png)

1. `shared_ptr`的声明和创建
2. `make_shared`创建一个指针。
3. `shared_ptr`的拷贝和赋值，引用计数。修改引用计数的几种情况：
    - 拷贝一个`shared_ptr`，比如用一个`shared_ptr`初始化另一个，值传参，返回值等情况，引用计数增加。
    - 给`shared_ptr`赋一个新值，引用计数减少。
    - shared_ptr被销毁时，引用计数减少。
4. 通过析构函数自动销毁它管理的对象。

## `new`

### 初始化
1. 默认初始化。`new`后面加类型，没有小括号，也没有花括号。
默认情况下，new分配的对象，不管是单个分配的还是数组中的，都是默认初始化的。这意味着内置类型或组合类型的对象的值是无定义的，而类类型对象将用默认构造函数进行初始化。
2. 值初始化。类型名字后加()即可，对于内置类型的变量，初始化为0，对于类类型的变量，调用默认构造函数。
3. 直接初始化。使用初始化列表加对象值，或者小括号加对象值。

对于自定义类型而言，只要一调用new，无论后面有没有加()，那么编译器不仅仅给它分配内存，还调用它的默认构造函数初始化。

还有auto初始化器。

### 内存耗尽
使用placement new（定位new），当内存耗尽时，防止new抛出bad_alloc异常。

## `shared_ptr`和`new`
1. 定义和改变shared_ptr的其他方法，shared_ptr对象还有其他几个构造函数，分别接收内置指针，unique_ptr，以及内置指针和内置指针的删除器，shared_ptr对象和它的删除器这几类参数。**接收内置指针参数的shared_ptr的构造函数是explict的，也就是必须显式调用构造函数，不能使用隐式转换将一个内置指针转换成shared_ptr。**
2. **当把一个shared_ptr指针绑定到一个普通指针时，接下来不应该再使用内置指针访问这部分内存了。**
3. **也不要使用shared_ptr的get()函数返回的指针初始化另一个智能指针或者为智能指针赋值。同时也不能使用delete删除get()返回的指针，否则会发生二次delete。**
4. p.reset()函数可以用来重置指针p，如果p是指向shared_ptr的唯一对象，会将p原本指向的对象释放；如果没有传入参数，p置为空；如果传入了参数q，让p指向q；还可以传递一个d，表示调用d而不是delete释放q。
5. copy on write可以通过shared_ptr实现？？？使用unique()函数检测是否自己是当前的shared_ptr的唯一用户，不是的话，调用reset()函数拷贝一个新的。


## 智能指针和异常
### 智能指针和哑类

### 使用自己的释放操作
shared_ptr假设它们指向的内容是动态内存，当它被销毁时，调用delete。我们可以自己定义一个删除器，取代delete的调用。``` c++
void end_connection(connection *p)
{
    disconnect(*p);
}
void func()
{
    connection c = connect(&d);
    shared_ptr<connection> p(&c, end_connection);
    // 创建一个connection类型的智能指针，构造函数的参数是一个内置的指针类型和一个调用函数。
}
```


## `unique_ptr`
![unique_ptr](unique_pointer.png)

## `weak_ptr`
`weak_ptr`不控制指向对象的生存周期，它指向一个shared_ptr管理的对象，将一个weak_ptr绑定到shared_ptr不增加引用计数。
![weak_ptr](weak_ptr.png)

## `new`和数组
### 初始化
1. `new int[]`，默认初始化。
2. `new int[]()`，值初始化。但是不能在括号中给出初始化器，也就是不能用auto分配数组。
3. `new int[10]{1, 2, 3}`，列表初始化。

### 动态分配空数组
动态分配一个空数组是允许的，但是不能解引用。
而定义一个长度为0的数组是不允许的。

### 智能指针和动态数组
`unique_ptr`有一个可以管理new分配的数组的版本。而shared_ptr没有相应的版本，如果想要使用`shared_ptr`管理数组，需要提供自己定义的删除器。

## allocate类

## 参考文献
1.《C++ Primer》第五版
