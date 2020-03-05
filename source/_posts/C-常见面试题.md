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

## const和宏常量的区别
const常量有数据类型，而宏常量没有。编译器可以对前者进行安全检查，对后者只能进行字符替换。

## inline[10,11]
1. 为什么引入inline？减少函数调用的开销。
2. inline只适合函数体代码简单的函数使用，不能有循环等语句。
3. 用inline声明的函数并不一定会inline，取决于编译器。
4. 建议inline函数的定义放在头文件中。
5. 类内定义的函数是隐式的inline函数。
6. inline关键字应该和实现放在一块，只在声明中说明没有用。
7. 慎用内联。

## inline和宏函数的区别
1. 宏函数是由预处理器对宏进行替换，而内联函数是通过编译器控制的。
2. 内联函数是函数，只不过在调用的时候像宏一样展开，减少函数调用。


## 指针函数和函数指针
指针函数是一个函数，返回类型是一个指针而已。
函数指针是一个指针，是一个指向函数的指针。

## 数组和指针
1. 数组是一块连续存放的空间，指针是一个地址，指针变量是一个变量。
2. 初始化。
3. 大小。
4. 指针数组和数组指针。
5. 作为函数参数。

## 指针和引用的区别
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

## 哈希冲突

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

## 虚函数和虚函数表[12]

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

## RTTI[13]
RTTI是运行时类型识别。

## `cast`
### `static_cast`
和C中提供的强制类型转换一样。

### `const_cast`
只能改变运算对象的底层const，可能去掉也可以添加底层const。

### `reinterprect_cast`
为运算对象的bit mode提供新的解释。比如可以把一个整数解释成字符串。

### `dynamic_cast`
`dynamic_cast`实现安全的动态转换。
它可以可以安全实现的执行downcast，但是需要是含有虚函数的类。

## 智能指针
### shared_ptr实现
引用计数放在指针成员中。``` c++
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

    T& operator*() 
    {
        return *p;
    }
    T& operator*() const 
    {
        return *p;
    }

private:

    T *pd;
    size_t *count;

};
```
### unique_ptr实现
``` c++
template<typename T>
class unique_ptr
{
public:
    unique_ptr():p(nullptr)
    { }
    // 构造函数
    explicit unique_ptr(T *data): p(data)
    { }

    //拷贝构造
    unique_ptr(const unique_ptr&) = delete;

    //拷贝赋值
    unique_ptr& operator=(const unique_ptr &rhs) = delete;

    //析构
    ~unique_ptr()
    {
        if(p)
            delete p;
    }

    T* release()
    {
        T *q = p;

        p = nullptr;
        return q; 
    }

    T* reset(T *data)
    {
        if(p)
            delete p;
        p = data;
    }

    T& operator*() {return *p;}
    //指针本身是个常量
    // const 修饰this指针，常量指针，指向不能改变，指针指向的值可以改变。为什么这里返回值不是常量。这个指针可不可以指向常量对象，可以。
    T& operator*() const {return *p};

private:
    T *p;

};
```

### shared_ptr和unique_ptr
1. 它们管理保存指针的策略，前者可以共享指针，而后者则独占指针。
2. 它们允许用户重载deleter的方式。只要在创建或者reset指针时给shared_ptr提供一个deleter即可。但是，deleter是unique_ptr对象的一部分。
shared_ptr的deleter是运行时绑定，所以需要使用指针，开销大，但是用户重载更方便。它的执行方式可能如下：`del?del(p): delete p`。
而unique_ptr的deleter是编译时绑定，避免了间接调用deleter的运行时开销。
3. shared_ptr不能动态管理数组，而unique_ptr可以动态管理数组。

### 智能指针和数组
1. shared_ptr不支持管理动态数组。需要提供自定义的delete（只要是一个可调用对象就行）。
2. unique_ptr支持动态管理数组，需要在类型后面加上一对方括号。当unique_ptr销毁时，会自动使用delete[]。不能使用成员访问运算符（点和箭头），但是可以使用下标访问运算符。


### weak_ptr
`weak_ptr`可以解决循环引用问题，比如双向链表中的循环引用。[9]


## 类模板和函数模板的区别
类模板用来生成类，函数模板的参数是由编译器推导的，而类模板的参数必须指定。

## 模板特化和偏特化
模板特化的本质是模板实例化，定义特化版本时，我们实际上是接替了编译器的工作。
函数模板只能全特化。
类模板可以全特化也可以偏特化。偏特化既有类型的偏特化，也有个数的偏特化。

## 为什么模板的声明和实现要写在一起而类的实现和声明要分开[7]
C++ 采用分离式编译。可以不用将所有的代码写在一个文件中，可以将他们分解为更小，更好管理的木块，可以独立的修改和编译这些模板。当我们改变这些模板时，只需要重新的编译它，并重新链接，不必重新编译其他文件。更多的内容可以查看CSAPP上介绍的。
对于普通对象和函数而言，类的声明写在头文件中，实现写在cpp文件中。cpp文件是可以单独被编译的。
但是模板函数的实现并不能直接编译成二进制代码，因为只有在真正使用模板的时候，才会知道模板参数的值。编译器和链接器的某一个部分，可以去掉模板的多重定义。

## 模板元编程

## 哈希表
哈希表冲突严重的话会退化成单链表。这个时候哈希表的各种操作的时间复杂度都提升了一个量级，会占用大量CPU时间，降低系统响应时间，可以用来做DDos攻击。
解决冲突的方法：

## malloc和new
### new
new一个空数组是和合法的，但是不能解引用！！！可以把它当做尾后迭代器来使用。

### malloc, calloc和realloc
1. malloc分配指定字节的数组，初值不定。
2. calloc分配nobj个size字节的对象，每一位都初始化为0。
3. realloc在以前分配的区域的基础上扩大为新的容量。如果原来的区域后面有足够大的空间，就在那里，否则复制到一个新的空间中。

### ptmalloc实现

### placement new
重载operator new函数。placement new可以为分配函数提供额外的信息。
Placement new允许我们将对象构造在已经分配好的内存上，分配好的内存不一定是operator new分配的内存，甚至可以是静态内存。

## 对齐
gcc 默认对齐是4字节对齐。在结构体中要注意。
malloc是16字节对齐。

## cmake和makefile
make用来执行makefile，调用makefile中用户指定的命令进行编译和链接。
makefile是文件，包含了怎么样进行编译和链接。
makefile可以手动写，但是它不跨平台（即不同平台的makefile不同），而且工程大很麻烦。
cmake可以根据CMakeList.txt跨平台自动生成makefile。
CMakeLists.txt是我们自己写的。

## gdb调试

## void *
可以接收任意类型的赋值，无需强制类型转换。
经过类型转换可以赋值给任意类型的变量。

## lambda表达式
1. 总共有四种可调用对象，函数，函数指针，重载了函数运算符的类和lambda表达式。
2. lambda表达式的声明。
auto f = [local variable](int x){return a;};
3. 对于那种只在一两个地方使用的简单操作，lambda表达式是有用的。当在多个地方使用的话，通过应该使用一个函数，或者需要很多语句的话，也是函数比较好。
4. capture list中引用和值的作用。
5. 可以和STL中的算法进行交互。

## 面向对象的几大原则
1. 单一职责原则。解耦，增强内聚，即高内聚低耦合。
2. 开放封闭原则。对扩展开放，对修改关闭。
3. 里氏替换原则。子类可以替换父类。
4. 依赖倒置原则。依赖于抽象而不是依赖细节。
5. 接口分离原则。不需要让客户程序依赖它们不需要的方法。一个接口应该只提供一种对外的功能。
6. 合成复用原则。
7. 迪米特原则。

## 设计模式
1. 单例模式
2. reactor模式

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
10.https://www.cnblogs.com/fnlingnzb-learner/p/6423917.html
11.https://isocpp.org/wiki/faq/inline-functions
12.https://isocpp.org/wiki/faq/virtual-functions
13.https://www.cnblogs.com/findumars/p/6358194.html
