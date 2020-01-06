---
title: C++ STL container
date: 2019-11-10 12:34:56
tags:
 - C/C++
 - STL
 - 容器
categories: C/C++
mathjax: true
---

## 什么是容器
一个**容器**就是一些特定类型对象的集合。C++标准库中提供了两类容器，一类是顺序容器，一类是关联容器。
关联容器和顺序容器有着根本上的不同：
关联容器的元素是按照关键字保存和访问的；而顺序容器是按照他们在容器中的位置顺序保存和访问的，这种顺序不依赖于元素的值，而跟元素加入容器时的位置相对应。关联容器中的许多行为和顺序容器相同，但是他们的不同之处反映了关键字的作用。顺序容器和关联容器共享公共的接口，不同容器可以按照不同的方式对其进行扩展，这个公共接口使得容器学习起来更容器，基于某个容器学习的内容可以扩展到其他容器上。
关联容器支持高效的关键字查找和访问，两个主要的关联容器是`map`和`set`，`map`中的元素是一些关键字－值(key-value)对，关键字索引，而值表示和索引相关的数据。而`set`中的每个元素只有一个关键字。


## 顺序容器概述
关于顺序容器更详细的内容可以查看[C++ sequential container](https://mxxhcm.github.io/2019/11/10/C-sequential-container/)。
### 顺序容器种类
所有C++标准库中的顺序容器包括：
- vector: 可变大小数组，
- list：双向链表，
- forward_list：单向链表，
- deque：双端队列
- array：固定大小数组
- string：与vector类似，但是专门用于字符操作，

### 定义顺序容器
``` c++
#include <array>
#include <vector>
#include <list>
#include <forward_list>
#include <deque>

using std::array;
using std::vector;
using std::list;
using std::forward_list;
using std::deque

#include <string>
using std::string;

array<string, 1000> c1;
vecotr<string> c2;
list<string> c3;
forward_list<string> c4;
deque<string> c5;
```

### 存取时间复杂度和存储策略
所有的顺序容器都提供了快速顺序访问元素的能力。但是，这些容器在随机访问或者增删元素上的能力上做了不同的性能折中：
- `string和`vector`：支持$O(1)$时间的随机访问；尾部增删是$O(1)$，在尾部之外的位置插入或者删除可能很慢$O(n)$。`string`和`vector`都存储在连续的的内存空间中，在中间增删需要移动增删位置之后所有的元素；在任何位置添加一个元素时，如果当前的存储空间不够，还需要分配新的存储空间，需要将所有的元素都移动到新的存储空间去。
- `list`和`forward_list`：`list`是双向链表，`forward_list`是单向链表，它们都不支持随机访问，在寻找某一个元素时，只能以$O(n)$的时间复杂度遍历整个链表，`list`支持双向顺序访问$O(n)$，`forward_list`只支持单向顺序访问$O(n)$；它们在任何位置插入或者删除的时间复杂度都是$O(1)$。
和`vector`,`deque`,`array`相比，链表需要存放指针记录前(后)节点的信息。此外`forward_list`没有`size()`和`back()`成员，因为`forward_list`的设计目标是和手写的单向链表性能相似，`size`操作会增大计算开销，对于其他容器而言，`size`是一个$O(1)$的操作。
- `deque`：`deque`支持$O(1)$时间的随机访问；在中间位置增删都是$O(n)$的开销，但是在`deque`两端增删是$O(1)$的事件开销。
- `array`：$O(1)$时间复杂度的随机访问；不支持增删操作。`array`和内置数组一样，大小固定，不支持增删，但是更安全。

### 顺序容器的选择
1. 通常使用`vector`是最好的选择，除非有更好的理由。
2. 程序有很多小的元素，而且空间额外开销很重要，不要用`list`或者`forward_list`。
3. 要求支持随机访问元素，使用`vector`或者`deque`。
4. 在容器中间插入或者删除，使用`list`或者`forward_list`。
5. 程序需要在容器头尾增删，而不会在中间增删，使用`deque`。
6. 如果即需要随机存取，又需要在容器中间增删，这个时候根据存取和增删的操作数量进行选择，哪种操作占据主导地位，就使用相对应的容器。
7. 如果不确定到底应该使用`vector`还是`list`，那么就只使用它们都支持的操作，不使用下标运算，使用迭代器，避免随机访问。

## 关联容器概述
关于关联容器更详细的内容可以查看[C++ associative container](https://mxxhcm.github.io/2019/11/10/C-associative-container/)。
标准库共有8个关联容器，他们在三个维度上有差异
1. `set`还是`map`，map存放key-value，set只存放key，或者说key=value。
2. 关键字是否可以重复，是否容器名字中包含multi
3. 元素顺序无序还是有序，容器名字是否包含unordered

### 关联容器种类
具体如下：
- `map`，关联数组，保存key-value
- `set`，只保存key，或者说key=value
- `unordered_map`，无序`map`，底层用hash实现
- `unordered_set`，无序`set`，底层用hash实现
- `multimap`，key可以重复出现的`map`
- `multiset`，key字可以重复出现的`set`
- `unordered_multimap`，key可以重复出现的无序`map`，底层用hash实现
- `unordered_multiset`，key可以重复出现的无序`set`，底层用hash实现

关联容器不支持顺序容器和位置相关的操作，如`push_back`，`push_front`等，因为关联容器中元素是根据关键字存储的，这些操作对关联容器没有意义。此外，关联容器也不支持接收一个元素值和一个数量值的插入操作和构造函数。
不过关联容器支持一些顺序容器不支持的操作。关联容器的迭代器都是双向的。

### 定义关联容器
``` c++
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>
#include <string>

using std::set;
using std::map;
using std::multiset;
using std::multimap;

using std::unordered_set;
using std::unordered_map;
using std::unordered_multiset;
using std::unordered_multimap;

set<string> c1;
map<long, string> c2;
multiset<string> c3;
multimap<long, string> c4;

unordered_set<string> c5;
unordered_map<long, string> c6;
unordered_multiset<string> c7;
unordered_multimap<long, string> c8;
```

## 顺序容器和关联容器的公共操作
1. 每一个容器都定义在一个头文件中，文件名和类型相同（除了multiset和multimap以及unordered_multiset, unordered_multimap。
2. **容器类型成员**。每个容器都定义了多个类型：
    - `iterator`，容器的迭代器类型
    - `const_iterator`，无法修改元素的迭代器类型
    - `reverse_iterator`，反向迭代器
    - `const_reverse_iterator`，不能修改元素的反向迭代器
    - `size_type`，无符号整数，足够保存容器的最大大小。
    - `difference_type`，有符号整数，足够保存两个迭代器之间的距离
    - `value_type`，元素类型
    - `reference`，元素的左值类型
    - `const_reference`，元素的`const`左值类型。 

3. **迭代器和迭代器范围。**所有容器都支持迭代器，通过解引用迭代器访问容器中的元素。一个**迭代器范围**由一对迭代器构成，两个迭代器分别指向同一个容器中的元素或者是尾元素之后的位置，通常一个被称为begin，一个被称为end。如何获取迭代器：
    - `c.begin()`和`c.end()`，返回指向c的首元素和尾后元素的迭代器
    - `c.rbegin()`和`c.rend()`，返回指向c的尾元素和首元素之前的反向迭代器
    - `c.cbegin()`和`c.cend()`，返回指向c的首元素和尾后元素的const_iterator
    - `c.crbegin()`和`c.crend()`，返回指向c的尾元素和首元素之前的const_reverse_iterator

    当`auto`和`begin`，`end`结合使用时，获得的迭代器类型依赖于容器的类型。只有当容器本身是const时，才能够得到`const_iterator`。
而`auto`和`cbegin`和`cend`使用时，获得的迭代器类型和容器类型无关，一直都是`const_iterator`。
关于迭代器的具体内容可以查看[]()。
4. **容器定义和初始化。**
    - `C c;`，默认构造
    - `C c1(c2);`或者`C c1=c2;`，拷贝构造，直接拷贝容器
    - `C c(b, e);`，拷贝构造，通过迭代器范围进行拷贝，将迭代器b和e指定范围的元素拷贝到c，不适用于array。这种方式不要求容器类型相同，只要能将要拷贝的对象转化为要初始化的容器的元素类型即可。
    - `C c{a, b, c...};`或者`C c={a, b, c...}`，列表初始化c，元素类型必须相同，同时显式的指定了容器的大小

    只有顺序容器（除了`array`）的构造函数才能接收大小参数。
    - `C seq(n);`，进行值初始化，不适用于`string`
    - `C seq(n, t);`，seq是包含n个初始值为t的元素

    总结一下：
    1. 将一个容器初始化为另一个容器的拷贝时，两个容器的容器类型和元素类型必须相同
    2. 使用迭代器拷贝构造容器时，不需要容器类型和元素类型相同，只需要待拷贝对象能够转换成要初始化的元素对象即可。
    3. 对于顺序容器（除了array）来说，它还有另一个构造函数，它的参数是容器大小和一个可选的元素初始值。如果不提供元素初始值，标准库会创建一个值初始化器，内置类型，如int，取0，`string`等类类型，由类进行默认初始化。即当如果元素是内置类型，或者具有默认构造函数的类类型，那么可以只提供一个容器大小参数。如果没有默认构造函数，就必须指定显式的元素初始值。
    4. 标准库`array`具有固定大小，定义`array`时，除了指定元素类型，还要指定元素个数。一个默认构造的`array`是非空的，这些元素都被默认初始化。如果进行列表初始化，初始值如果小于`array`大小，剩余的元素执行值初始化。对于类类型来说，不论是默认初始化还是值初始化，都需要类有一个默认构造函数。

6. **赋值, `assign`和`swap`**。
    - `c1 = c2`，将c1的元素用c2的元素替换，c1和c2类型必须相同
    - `c1 = {a, b, c, ...}`，将c1中的元素替换为列表中元素，不适用于array
    - `a.swap(b)`，交换a和b的元素
    - `swap(a,b)`，和`a.swap(b)`相同。

    还有不适用于关联容器和`array`的`assign`操作，
    - `seq.assign(b, e);`，将seq中元素替换为迭代器b和e中的元素，迭代器b和e不能指向seq中的元素
    - `seq.addign(il);`，将seq中的元素替换成初始化列表il中的元素
    - `seq.assign(n, t);`，将deq中的元素替换成n个值为t的元素

    1. 赋值号左右两边的运算对象必须具有相同的类型，而assign不需要两个容器的类型相同，只需要元素类型相容即可。`array`允许直接赋值`array`，但是不支持`assign`操作，也不允许用花括号包围的值列表进行赋值，因为右面运算对象的大小可能和左面运算对象的大小不同（见C++ primer第五版302页），而`array`的大小是不可变的。
    2. 赋值操作会让指向左边容器内部的迭代器，引用和指针失效。而swap交换容器内容不会使得指向容器的迭代器，指针和引用失效（容器类型为array和string除外）。
    3. swap交换`array`时，两个`array`的大小必须相同，类型相同。`swap`交换两个`array`会真正交换两个`array`的元素。
    4. swap交换除了`string`之外的容器时，指向容器的迭代器，引用和指针都不会失效，即访问的还是未交换之前的对象，但是这些对象所属的容器变了。

7. **大小。**
    - `size()`，容器当前容纳的元素个数，不支持`forward_list`
    - `max_size()`，容器所能容纳的最大元素个数
    - `empty()`，容器是否为空

8. **关系运算符。**
    1. `==`和`!=`，所有的容器都支持的运算符。
    2. `<=`,`<`,`>=`, `>`，关系运算符（无序关联容器不支持），关系运算符两侧的容器类型必须一样，容器类的元素类型也必须一样。
    3. 容器的相等运算实际上是使用的元素的`==`运算实现的，而容器的关系运算实际上是使用元素的`<`运算实现的。对于类类型来说，必须对相应的操作符重载，才能进行相应的关系运算，否则就无法进行。
    4. 两个容器比较大小的规则：
两个容器大小相等，对应元素相等，这两个容器相等。
两个容器大小不同，但是较小元素中每个元素都等于较大容器中的对应元素，较小容器小于较大容器。
如果两个容器都不是另一个容器的前缀子序列，则他们的结果取决于第一个不相等的元素的比较结果。

9. **增删元素（不适用于`array`）。**注意，在不同的容器中，操作的接口都不同
    - c.insert(args)，将args中的元素拷贝进c
    - c.emplace(inits)，使用inits构造c中的一个元素
    - c.erase(args)，删除args指定的元素
    - c.clear()，删除c中所有元素，返回void
2. 顺序容器几乎可以保存任意类型的元素，但是某些容器对于元素类型有特殊的要求，我们可以为不支持特定操作的类型定义容器，但是使用只用那些没有特殊要求的容器操作了。
顺序容器构造函数的一个版本接受容器大小参数，它使用元素类型的默认构造函数，但是有些类没有默认构造函数，这时候我们可以定义这种类型的容器，但是需要传入一个元素的初始化器。例如：```c
vector<noDefault> v1(10, init); //正确，
vector<noDefault> v2(10);   //错误，因为没有默认构造函数
```


## 参考文献
1.《C++ Primer》第五版
