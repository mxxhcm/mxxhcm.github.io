---
title: C++ STL iterator
date: 2019-12-19 20:00:45
tags:
 - C/C++
 - STL
categories: C/C++
---

## 迭代器
C++ 定义了许多容器，包括`vector`，`list`，`map`等等，这些容器中，只有少部分支持`[]`运算符，其他的都不支持，为了访问这些容器，C++ 提供了迭代器访问这些容器，迭代器和指针类似，提供了对对象的间接访问。
注意`string`不是容器，但是它支持很多和容器类型的操作。

## 迭代器和其他组件之间的关系
STL的核心思想，将数据容器和算法分开，彼此独立设计，然后再将他们组合在一起。
算法不依赖于容器，所以就不同的容器就可以使用同一个算法，迭代器就起到让不同的容器使用同一个算法的作用，相当于提供了一个统一的接口。
虽然算法不依赖于容器，但是它依赖于迭代器指向元素的类型。所以这也是为什么要有itreator_traits的原因，给出一个容器的迭代器，算法能够从中获得迭代器指向元素的类型。
iterator_traits的作用，STL的advance操作，对于不同的迭代器，可以使用不同的版本，如果没有iterator_category这个类型的话，只有在执行器进行判断（影响效率），可以利用iterator_traits实现重载。（或者总结一下，实现模板参数相同情况下的重载。）

## 迭代器范围
所有容器都支持迭代器，通过解引用迭代器访问容器中的元素。一个**迭代器范围**由一对迭代器构成，两个迭代器分别指向同一个容器中的元素或者是尾元素之后的位置，通常一个被称为begin，一个被称为end。如何获取迭代器：
    - `c.begin()`和`c.end()`，返回指向c的首元素和尾后元素的迭代器
    - `c.rbegin()`和`c.rend()`，返回指向c的尾元素和首元素之前的反向迭代器
    - `c.cbegin()`和`c.cend()`，返回指向c的首元素和尾后元素的const_iterator
    - `c.crbegin()`和`c.crend()`，返回指向c的尾元素和首元素之前的const_reverse_iterator

    当`auto`和`begin`，`end`结合使用时，获得的迭代器类型依赖于容器的类型。只有当容器本身是const时，才能够得到`const_iterator`。
而`auto`和`cbegin`和`cend`使用时，获得的迭代器类型和容器类型无关，一直都是`const_iterator`。


## 和迭代器相关的类型
迭代器有const和非const之分。此外，大多数容器还提供反向迭代器，两个类型相减得到`difference_type`类型。它们通过如下方式定义：
``` c
vector<int>::iterator it;   //非const迭代器，可读和可写
string::iterator it2;

vector<int>::const_iterator it3;    //const迭代器，只是可读
vector<int>::const_iterator it4;

vector<int>::difference_type count;
```

## `begin`和`end`获得迭代器
拥有迭代器的类型（`string`和容器等）也拥有返回迭代器的成员函数，如`begin`和`end`，`begin`返回指向容器第一个元素或`string`第一个字符的迭代器，而`end`返回容器或者`string`的最后一个元素的下一个位置，是一个根本不存在的元素，通常把这个迭代器称为尾后迭代器。
如果容器（`string`)为空，`begin`和`end`返回的是同一个迭代器，都是尾后迭代器。
如果对象是常量，`begin`和`end`返回`const_iterator`，否则返回`iterator`。
而`cbgein`和`cend`无论对象是否是常量，都返回`const_iterator`。

## 迭代器失效
1. 范围for语句会使迭代器失效
2. 任何一种可能改变`vector`对象容量的操作，如`push_back`，都会使得`vector`对象的迭代器失效。


## 迭代器运算符
迭代器支持的运算有：
- `*iter`，表示返回迭代器iter所指元素的引用
- `iter->num`，解引用`iter`并且获得该元素中名为`num`的成员，等价于`(*iter).mem`，即箭头运算符把解引用和成员访问两个操作结合在了一起。
- `++iter`，让`iter`指向容器中的下一个元素
- `--iter`，让`iter`指向容器中的上一个元素
- `iter1==iter2`
- `iter1!=iter2`，判断两个迭代器是否相等，即是否指向同一个元素或者指向同一个容器的尾后迭代器。

**注意，`forward_list`没有`--`操作。**
尽量使用`!=`而不是使用`<`，因为所有的标准容器和`string`都定义了`==`和`!=`，而只有部分容器和`string`定义了`<`，所以`==`和`!=`是比较通用的。

### 解引用和成员访问操作相结合
``` c
(*it).empty()
it->empty()
```

## 迭代器运算
`vector`和`string`, `deque`和``的迭代器提供了更多的运算符，这些运算被称为迭代器运算。如下所示：
- `iter+n`
- `iter-n`
- `iter+=n`
- `iter-=n`，
- `iter1-iter2`，
- `>, >=, <=, <`

`iter1-iter2`是两个迭代器之间的距离，`iter`和`iter2`必须是同一个容器中的迭代器。可以使用这些运算符进行二分搜索。

## insert iterator
## iostream iterator
## reverse iterator
## move iterator

## 迭代器是一种智能指针
迭代器是一种智能指针，而指针最常见的成员访问和解引用操作，迭代器一定要实现，所以迭代器一定要重载这两个运算符。
为什么每一个STL容器都要有一个专属迭代器？将容器的实现细节封装起来，不被使用者看到。比如说list节点中的next指针，在设计迭代器的时候就会暴露出来，而我们实际上使用list时，并不会知道next指针。

## 迭代器相关的类型
在使用迭代器的时候，可能会用到迭代器指向对象的类型，比如算法中声明一个迭代器指向对象类型的变量。
而我们不能直接获得一个对象的类型，比如find：``` c++
template<typename InputIterator, class T>
InputIterator find(InputIterator first, InputerIterator last, const T&value);
```
在find中，我们想要声明一个InputIterator指向类型的变量（**不一定是T!!!它们没关系，可能碰巧一样**）。怎么做到？函数模板的参数推导？可以做到，但是需要做一个转换。
如果要用InputIterator指向的类型作为返回值类型怎么办？
这样子很麻烦，而且常用的类型有以下五种：
1. value type
2. pointer
3. reference
4. difference type
5. iterator_category

需要一个更全面的方法。

## Traits
使用Traits来获得上面介绍的五种类型！
traits是怎么被发明的呢？一步一步来？
如果想要获得迭代器指向对象的类型，最直接的想法是声明内嵌：``` c++
template <typename T>
class MyIter
{
public:
    typedef T value_type;   //类型声明

};
```
其他人想要使用迭代器指向对象的类型时，可以使用以下方式进行访问：``` c++
template <typename I>
typenme I::value_type func(I ite);
```
在使用的时候，要加上typename，因为I是一个模板参数，I::value_type到底是个类型，成员函数，还是数据成员。加上typename告诉编译器这是一个类型。
**但是并不是所有迭代器都是class type，比如说指针也是迭代器，但是它不是类类型，所以就没有办法为它定义内嵌类型，但是STL必须能够把指针当做迭代器。**这时就可以使用偏特化（个数的偏，类型的偏）解决问题，比如：``` c++
template<typename I>
struct iterator_traits
{
    typedef typename I::value_type value_type;
}

// 注意这里为什么加上const，如果不加const，会推导出顶层const，使得我们声明的变量，无法被修改。
template<typename I>
struct iterator_traits<const I*>
{
    typedef typename I value_type;
}

// I可以是指针，也可以是迭代器。。。
template<class I>
typename iterator_traits<I>::value_type func(I ite)
{
    return *ite;
}
```

最常用的迭代器的类型有五种：value_type, reference, pointer, difference_type，iterator_category。如果要定义我们自己的迭代器，最好要定义这五个类型，这样子能够和算法相结合。``` c++
template<typename T>
struct iterator_traits
{
    typedef T::value_type value_type;
    typedef T::pointer pointer;
    typedef T::reference reference;
    typedef T::difference_type difference_type;
    typedef T::iterator_category iterator_category;
};
```
此外，需要对指针和指向常量的指针进行偏特化。

#### value_type
迭代器指向对象的类型。

#### difference_type
表示两个迭代器之间的距离。

#### reference
如果C++函数要返回左值，需要返回引用。

#### pointer
``` c++
// Item &是引用，而Item*是指针。
Item& operator*() const {return *ptr;}
Item* operator->() const {return ptr;}
```

#### iterator_category
根据迭代器的移动特性和操作，可以把它们分成五类：
- 输入迭代器，不允许外界改变，只读。
- 输出迭代器，只写
- 前向迭代器，允许读写操作。
- 双向迭代器，可双向移动。
- 随机访问迭代器，随机存储。

![iterator_category](iterator_category.png)
这五种迭代器之中，支持的指针算术运算的能力不同。前三种支持operator++，第四种还支持operator--，第五种涵盖了所有指针运算p+n, p1 < p2，p[n]等。
如果算法中能使用上面层次中的迭代器，那么下面层次中的迭代器也一定能使用，但是效率不一定最好，这里说的效率不一定最好，指的是下层迭代器本来可以访问的更快，偏偏让它一步一步走。所以有一些函数会针对于每一类迭代器实现一个版本，然后根据iterator_category调用相应的版本。
**为什么要定义这个类型？因为我们想要根据不同的迭代器类型，判断要执行哪个函数，而且不是运行期执行判定，而是编译器判断！！！这样速度快。通过引入几个空的类型tag，让编译器可以进行识别，从而根据函数重载调用相应的函数。**
**为什么用类定义迭代器的分类标签？**一方面可以促成重载机制的成功运作，执行重载解析，另一方面不用再写做传递调用的函数。

STL的算法以它能接受的最低阶的迭代器类型，为迭代器类型参数命名。


## 继承iterator类
为了确保所有编写的迭代器都能有这个类型定义，和STL其他组件兼容，可以让新设计的迭代器继承iterator类。
这个类除了类型定义之外，没有任何其他操作。此外，还需要继承一个类型的tag。

## `__type_traits`
`__type_traits`前面加了双下划线，SGI使用双下划线前缀表示这是SGI STL内部使用的东西，不在标准范围内。
`iterator_traits`萃取了迭代器的特性，而`__type_traits`萃取type的特性，比如这个type是否有non-trivial的构造，拷贝构造，拷贝赋值，和析构函数。如果没有这些non-trivial的操作，就可以采取最有效的策略（比如不调用那些函数，直接进行内存操作），获得最高的效率。
编译器只有面对class object形式的参数时，才会做参数推导。


## 参考文献
1.《C++ Primer第五版》
