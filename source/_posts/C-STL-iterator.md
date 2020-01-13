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

## 迭代器类别
- 输入迭代器
- 输出迭代器
- 前向迭代器
- 双向迭代器
- 随机访问迭代器

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

## 参考文献
1.《C++ Primer第五版》
