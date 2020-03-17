---
title: C object model
date: 2020-03-10 20:41:06
tags:
 - C/C++
categories: C/C++
---

## C++对象模型
**C++ 对象模型：所有的nonstatic data member放在每一个类对象内。所有的static data member, staitc member function和member function都放在所有类对象之外。而对于虚函数，每一个类产生一堆指向虚函数表的指针，放在一个表中，叫做虚表。每一个类对象都会有一个虚指针指向虚表。如果加上继承的话，每一个类对象中还要放着一个虚基类对象的指针。**
指向不同类型指针的差异，既不在指针表示不同，也不再其内容（代表一个地址）不同，而是在其所寻址出来的object类型不同。指针类型起到的作用是告诉编译器如果解释某个特定地址中内存内容和其大小。
转型其实只是一种编译器指令，大部分情况下他并不改变一个指针所含的真正地址，它只影响被指向内存的大小和其内容的解释方式。
**一个指针或者引用之所以支持多态，是因为它们并不会引发内存中任何与类型有关的内存委托操作，会受到影响的只是它们所指向内存的“大小和内容解释方式”而已。**

## 构造函数语义学
为什么引入explicit关键字，避免将一个单一参数的构造函数当成一个转换运算符。 

### 默认构造函数
默认构造函数会在**编译器需要**的时候被编译器产生出来。注意，这是编译器的需要，不是程序的需要。程序的需要需要程序员进行负责。

如果没有任何用户声明的构造函数，在合适的时候编译器会暗中声明一个non-trivial的构造函数。什么时候会合成non-trivil的构造函数？
四种情况：
1. 类中包含具有默认构造函数的成员对象。需要调用成员对象。的默认构造函数。
2. 类继承了含有默认构造函数的基类对象。需要调用派生类的默认构造函数。
3. 含有虚函数的类。初始化类对象的vptr。
4. 含有虚基类的类。初始化虚基类对象。

在这四种情况之外的话并且没有声明任何构造函数的类，它们拥有implicit trivial default constructors，实际上并不会被合成出来。

### 拷贝构造函数
三种情况，会调用拷贝构造函数：
1. 明确的以一个object作为另一个object的初值。
2. pass by value。
3. 返回一个类对象而不是类对象的引用时。

#### 默认的memberwise的初始化
default memberwise initialization是把每一个内建的或者派生的数据成员（例如指针和数组，就是一个派生的数据成员）的值，从某个object拷贝到另一个object。对于成员类对象，以递归的方式执行memberwise initialization（其实就是深拷贝）。

C++ 标准同样把拷贝构造函数分为trivial和non-trivial的，只有nontrivial的构造函数才会被合成到程序中，决定一个拷贝构造函数是不是nontrivial的，取决于类是否会展现bitwise copy semantics时。bitwise copy其实就是浅拷贝。
什么时候不展现处bitwise copy semantics：
1. 类中包含含有拷贝构造函数的成员对象时，无论这个拷贝构造函数是设计者声明的，还是编译器合成的。
2. 类继承了含有拷贝构造函数的基类时，无论这个拷贝构造函数是设计者声明的还是编译器合成的。
3. 类声明了虚函数时。因为类对象中需要虚指针，没有bitwise semantics，编译器需要合成一个拷贝构造函数将vptr合适的初始化。当相同类型的对象相互赋值（基类对象赋值给基类对象，派生类对象赋值给偏生类对象），直接拷贝vptr是安全的，但是把派生类对象赋值给基类对象，vptr直接拷贝就不安全了，合成的拷贝构造函数会负责派生类的vptr的初始化。
4. 类直接或者间接继承了虚基类的时候，也会使得bitwise semantics失效。编译器需要让派生类中的虚基类子对象在执行器就准备妥当，还需要维护位置的完整性。
对于一个继承了虚基类的派生类，编译器会为派生类调用基类的默认构造函数（如果用户没有调用的话），还会将虚指针初始化，最后还会对基类的子对象进行定位（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）（bitwise copy可能会破坏这个位置）。

### 程序转化语义学
#### 函数参数的初始化
调用拷贝构造函数，构造一个参数传递给函数，并且把函数原型改成接受引用参数。

#### 返回值的初始化
函数返回值怎么获得？一般都是在函数内部声明一个临时对象，然后返回这个临时对象。编译器在处理的时候使用了：
双阶段转换：
1. 在函数声明上加了一个额外的引用参数，用来放置返回值。
2. 在return之前调用这个额外参数的拷贝构造函数拷贝函数内部处理的临时对象。


#### 使用者层面的优化

#### 编译器层面的优化（Named Return Value）
直接使用一个result参数代替函数内部的临时对象，少了一次拷贝构造，这个也叫作name return value(NRV)优化。
为什么没有拷贝构造函数就不能实行NRV优化？（可能是因为NRV优化就是针对拷贝构造函数进行的，你连优化对象都没有了，还优化个毛线。）
我在gcc下进行测试，默认是打开了NRV优化的，但是如果return 指令没有发生在top level就会失效（可看文章最后的代码）。

#### 要不要拷贝构造函数
如果某个类的拷贝构造函数被视为trivial（即没有带有拷贝构造函数的成员对象，或者基类对象，也没有虚函数和虚基类）。
那么memberwise的初始化会导致bitwisecopy，很快速很安全。
如果单从是否复制的角度来看，不用提供显式的拷贝构造函数。但是！！！如果需要大量的memberwise初始化操作，比如值传递，那么提供拷贝构造函数就可以使用NRV优化。

### 成员初始化列表
构造函数用来给函数设置初值，除了以下的四种情况，必须使用初始化列表进行初始化，其他情况下既可以在构造函数体内，还可以使用初始化列表。
1. 初始化一个引用成员
2. 初始化一个常量成员
3. 调用一个基类的有参数的构造函数时
4. 调用一个成员类的有参数的构造函数时

初始化列表中到底发生了什么？
编译器会一一操作初始化列表，用适当的顺序在构造函数内任何用户显式的代码之前安插初始化操作。
还有，就是初始化顺序和初始化列表中的顺序无关，和在类中声明的顺序相关。

## Data语义学
空类的大小为1，因为编译器在空类中安插了一个char字节，使得这两个对象在内存中的位置是独一无二的。标准规定空类的大小大于0。虚基类的大小是8字节，因为虚指针。
如果没有虚函数，但是有虚基类，派生类对一个空类进行继承，它的大小受到三个因素的影响（因为没有虚函数，所以没有虚指针）：
1. 语言本身的overhead，比如虚基类，在派生类对象中需要有指向虚基类对象的指针。它指向虚基类子对象，或者一个表格。
2. 编译器的优化处理。
3. 对齐。

在菱形继承中，X是空的虚基类，A和B派生自X，没有自定义的数据，Y派生自A和B。
在没有优化的时候派生类（A和B）的大小是虚基类指针大小，一个char，一个对齐，也就是8+1+3 = 12字节。A的大小是(虚基类指针大小，一个char，和派生自A和B指向，最后加起来总共是1+8+8+3 = 20。

空虚基类(empty virtual base class)，它提供了一个虚接口，一个空的虚基类被看成派生类的最开始的一部分，也就是说派生类不再是空类了。所以派生自虚基类的空类的大小就是一个指针的大小。在这里，A和B的大小都是一个指向虚基类指针的大小，是8字节。而Y是16字节。


### 数据成员的绑定
两个防御性规则
1. 把所有的数据成员都放在class声明的开始处。
2. 把所有的inline函数都放在class声明的外部。如果一个inline函数体，在整个类的声明没有完全被看到之前，是不会被评估求值的。但是对于成员函数的参数列表并不是这样的，参数列表中的类型还是会在第一次遇到时被求值。

这一节最重要的就是为什么要把类型的typedef放在类的最前面。虽然对于函数的定义来说，可以使用任何类中声明的对象，但是对函数的声明是按照类中的声明顺序进行解析的！所以如果把类型的typedef放在最后，在解析函数声明的时候就可能出错（如果在函数声明出现之前没有找到某个类型，就会查找类外部的作用域）。


### 数据成员的布局
1. 静态数据成员存放在程序的data segment中。和单个的类对象无关。
2. 非静态数据在类对象中的排列顺序和其被声明的顺序一致。
C++标准要求任何在access访问块中，只要满足较晚出现的的members在类对象中有较高的地址即可，它们不一定连续，中间可能会有padding。
3. 还有编译器内部使用的数据成员，用来支持整个对象模型。比如vptr，所有编译器都会把它安插在含有虚函数的类对象之内。一般放在类对象的最前端，有的也放在最后。

### 数据成员的存取
#### static data member的存取
1. **每一个静态数据成员都只有一个实体，存放在程序的data segment**。C++ 中通过指针和对象存取成员，结果完全相同的唯一情况就是，就是对静态数据成员的存取。实际上，它们的存取都没有经过类对象，因为静态数据成员不在类中。
2. 如果取一个static data member的地址，会得到一个指向其数据类型的指针，而不是一个指向class data member的指针，因为static data member并不含在类对象中。
3. 如果不同的类都声明了同名的静态数据成员，会导致命名冲突，编译器的方法是暗中对每一个static data member进行编码，从而区分它们。

#### nonstatic data member
**非静态数据成员直接存放在每一个类对象中，只能通过class object（不管是直接的还是简介的）访问。**比如在member function中，访问nonstatic data member，实际上是经过一个隐式的this指针进行的。
通过指针和.运算访问nontstaic data member，访问的data member是一个struct member，一个clas member，单一继承或者多重继承的情况下效率都相同。
对一个nonstatic data member进行存取操作，编译器需要把类对象的起始地址加上偏移量。每一个nonstatic data member的偏移量在编译时就知道，即使它属于一个基类子对象。因此，存取一个nonstatic data member的效果和存取C结构体或者一个费派生类的成员是一样的。
但如果某个data member是虚基类的成员，指针的存取速度会慢一些。

### 继承和data member
在C++ 继承模型中，一个派生类对象表现出来的，是自己的members加上基类data member的总和。它们的排列顺序没有要求，在大部分编译器中，base class member总是出现在上面。属于虚基类的data member除外。

#### 单继承不含虚函数

#### 单继承和虚函数

#### 多重继承

#### 虚继承
对于虚继承来说，无论一个虚基类在继承体系中出现了多少次，派生类中只会含有一个虚基类子对象。


## 函数语义学
类的member function有三种：static member function, nonstatic member function和virtual function。
static member funciton的主要特性是没有this指针，它的其他特性统统来自主要特性：
1. 不能直接存取非non static data member。
2. 不能声明为非const，volatile, virtual。
3. 不需要经过class object进行访问。


### member的调用方式
#### nonstatic member functions
nonstatic member function的访问效率必须和一般的nonmember function效率相同。这个是怎么实现的呢？编译器通过将nonstatic member function函数实体转换成对等的nonmember function函数实体：
1. 向nonstatic member function添加一个额外的参数（this指针），如果是const nonstatic member function，是通过this指针是一个指向常量对象的指针。this指针本身就是一个常量指针，即它的指向不能改变。
2. 使用this指针对每一个nonstatic data member进行存取。（这就是上一节介绍的东西）。
3. 将member function重新写成一个nonmemeber functon，就是对它起一个新名字。

#### virtual member functions
对于一个virtual函数来说，比如```c++
Point3d obj, *ptr = &obj; 
ptr->normalize();   //normalized虚函数
```
会被转化成：``` c++
(*ptr->vptr[1])(ptr);
```
其中第一个ptr表示指针ptr，vptr是虚指针，指向一个虚表，1是normalize()虚函数在虚函数表中的位置，ptr表示传递给this指针的实参。
在一个虚函数内调用另一个虚函数，会比较快。
而对于通过成员访问运算符调用的虚函数，因为它不支持多态，所以会把虚函数当做普通函数进行解析。

### static member function
对于static memeber function来说，通过指针或者成员访问运算符调用，编译器会将它们转换成一般的nonmember function调用。

如果取一个static member funciton的地址，获得的是一个地址。因为static member function没有this指针，所以地址类型不是一个指向class member function的地址，而是一个nonmember function指针。

### virtual member function
virtual function的一般实现模型：每一个class有一个虚表，包含该class中的virtual function地址，每个对象都有一个vptr，指向virtual table的所在。


## 代码
NRV被关闭：``` c++
#include <iostream>

using std::cout;
using std::endl;


class X
{

public:
    X():_val(0)
    {
        cout << "X()" << endl;
    }

    X(int val): _val(val)
    {
        cout << "X()" << endl;
    }



    X(const X& x)
    {
        cout << "X(const X& x)" << endl;
    }

private:
    int _val;
};


X bar(int a)
{
    if(a % 2)
    {

        X x = X(a);
        return x;
    }

    X x;
    return x;
}

int main()
{

    X t = bar(3);
    return 0;
}
```
上述代码会调用一次构造函数，一次拷贝构造。注意，这个构造函数是临时对象x调用的。t调用的是拷贝构造函数。

## 参考文献
1.《深入探索C++对象模型》
