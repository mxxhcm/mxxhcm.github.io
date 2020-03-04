---
title: C 常见面试题
date: 2020-02-29 21:43:10
tags:
 - 面试
categories: 面试
mathjax: true
---

## static关键字[4]
1. 隐藏全局变量和函数的作用域。这一个已经被标准抛弃了。
2. 修饰局部变量，改变变量的生命周期。
3. 修饰类的数据成员和成员函数。

## const关键字
1. 顶层const和底层const。主要是常量指针和常量引用。
2. const修饰形参和const修饰返回值。
3. 和类相关。
修饰成员变量，必须使用初始化列表修饰。
const修饰成员函数。
修饰类对象，必须调用const类型的函数。
const可以用来区分重载。

## 指针函数和函数指针
指针函数是一个函数，返回类型是一个指针而已。
函数指针是一个指针，是一个指向函数的指针。

## 数组和引用的区别
1. 可以定义指针的指针，但是不能（直接）定义引用的引用（模板参数推导的时候可以）。
2. 指针可以被重新赋值。而引用一旦被绑定就不能修改。
3. 指针可以为空，引用不能。
4. 指针本身是一个变量，有它自己的内存地址，以及在栈上的大小。而引用的内存地址和它绑定的对象一样，看起来是一样的，但是实际上是不一样的（编译器做了一些操作），可以把引用当做它引用对象的别名。
5. 指针必须被解引用才能使用，而引用不需要。
6. 指针变量可以进行运算，而引用不能。
7. const引用可以指向字面值常量，而指针不能（除了字符数组）。
8. 指针本身可以是const的，而引用不能。底层const和顶层const。

## vector扩容
### 为什么不是常数
成倍增长可以保持常数时间复杂度的插入操作。
常数增长的插入时间复杂度是O(n)。
为什么？假设要插入n个元素，成倍增长的倍数是p，常数增长的大小是q。
成倍增长总共的复制开销：
$$\sum\_{i=1}^{log_p^n } p^i $$
常数增长总共的复制开销：
$$\sum\_{i=1}^{n/q} qi $$

### 为什么是1.5或者2
为什么是1.5？可以复用之间的空间。
为什么是2？实现简单。

## strlen和sizeof的区别
1. sizeof是运算符。它的值在编译时就已经确定了，所以不能用来获得动态分配的内存空间的大小。
2. strlen是函数，在运行时计算的。
3. 静态数组传递给strlen就会退化成指针，在传递给sizeof的时候还是数组。strlen是数组中内容的长度，而sizeof是静态数组声明时的大小。
4. 动态数组(malloc)分配的，strlen能获得数组中内容的大小，而sizeof只能获得指针的大小。
5. strlen不会计算空字符，而sizeof会。

## override和overload
只有虚函数能被ovrride。
override是用来实现多态的，函数的参数类型和声明和可以父类中完全一样。
而overload中，函数的参数列表必须不同。它的作用我觉得可能是方便记忆，方便理解。实现同一个功能的函数具有同样的名字。

## 空类的大小为1字节
标准规定空类的大小为1，因为两个不同的对象需要不同的地址表示。标准规定完整对象的大小大于0。

## 虚函数和虚函数表

## 虚析构函数
一个基类总是需要虚析构函数。

## 虚继承
为什么要有虚继承？
派生类可能多次继承同一个类。默认情况下，派生类会含有继承链上每个类对应的子部分，如果某个类在派生过程中出现了多次，派生类中将包含该类的多个子对象。比如iostream之类的类，肯定不行。
声明成虚继承的继承体系中，无论虚基类在继承体系中出现了多少次，派生类中都只含有一个共享的虚基类子对象。

## 多态
C++的多态分为两种，一种是静态多态，通过函数重载和模板实现。
一种是动态多态，虚函数实现。

C实现C++的多态[5,6]。

## RTTI

## `cast`
### `static_cast`
### `const_cast`
### `reinterprect_cast`

### `dynamic_cast`
`dynamic_cast`可以安全的执行downcast，但是需要是含有虚函数的类。

## 智能指针
shared_ptr，引用计数放在指针成员中。``` c++
template<typename T>
class shared_ptr
{

public:
    shared_ptr():pd(nullptr)
    { }
    shared_ptr(T *data): pd(data), new size_t(1)
    { } 

    // 拷贝构造函数
    explicit shared_ptr(const shared_ptr<T> &rhs)
    {
        pd = rhs.pd; 
        count = rhs.count;

        ++*rhs.count;
    }

    shared_ptr<T>& operator=(const shared_ptr<T> &rhs)
    {
        if(count && --*count == 0)
        {
            delete pd;
            delete count;
        }

        pd = rhs.pd;
        count = rhs.count;
        if(rhs.count)
            ++*count;

        return *this;
    }

    ~shared_ptr()
    {
        if(count && --*count == 0)
        {
            delete pd; 
            delete count;
        }
        
    }

private:

    T *pd;
    size_t *count;

};
```

`weak_ptr`可以解决循环引用问题，比如双向链表中的循环引用。[9]

## 模板特化和偏特化

## 为什么模板的声明和实现要写在一起而类的实现和声明要分开[7]
C++ 采用分离式编译。可以不用将所有的代码写在一个文件中，可以将他们分解为更小，更好管理的木块，可以独立的修改和编译这些模板。当我们改变这些模板时，只需要重新的编译它，并重新链接，不必重新编译其他文件。更多的内容可以查看CSAPP上介绍的。
对于普通对象和函数而言，类的声明写在头文件中，实现写在cpp文件中。cpp文件是可以单独被编译的。
但是模板函数的实现并不能直接编译成二进制代码，因为只有在真正使用模板的时候，才会知道模板参数的值。编译器和链接器的某一个部分，可以去掉模板的多重定义。

## 哈希表

## malloc和new

## ptmalloc实现

### placement new
重载operator new函数。placement new可以为分配函数提供额外的信息。
Placement new允许我们将对象构造在已经分配好的内存上，分配好的内存不一定是operator new分配的内存，甚至可以是静态内存。

## 对齐
gcc 默认对齐是4字节对齐。
malloc是16字节对齐。

## 参考文献
1.https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable-in
2.https://blog.csdn.net/dengheCSDN/article/details/78985684
3.https://www.cnblogs.com/carekee/articles/1630789.html
4.https://stackoverflow.com/a/943303/8939281
5.https://stackoverflow.com/questions/524033/how-can-i-simulate-oo-style-polymorphism-in-c
6.https://blog.csdn.net/dumpling5232/article/details/52632060
7.https://blog.csdn.net/uestclr/article/details/51372780
8.https://stackoverflow.com/questions/10068653/separate-compilation-in-c
9.https://blog.csdn.net/qq_34992845/article/details/69218843
