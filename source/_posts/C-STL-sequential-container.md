---
title: C++ STL sequential container
date: 2019-11-10 12:37:41
tags:
 - C/C++
 - 容器
 - STL
categories: C/C++
---

## C++标准库中的顺序容器
### `vector`
- 内部数据结构为数组，可以自动增长
- 在后端插入和删除，push_back()和pop_back()，时间复杂度为$O(1)$
- 在中间和前段插入和删除，insert()和erase()，时间和空间复杂度是$O(n)$
- 分配连续内存，
- 支持随机数组存取，查找的时间复杂度$O(1)$
- 支持[]访问
- 头文件vector

### `list`
- 内部数据结构为双向环状链表
- 任意位置插入和删除的时间复杂度是$O(1)$
- 链式存储，非连续内存
- 不支持随机存取，查找的时间复杂度是$O(n)$
- 不支持[]访问
- 头文件list

### `forward_list`

### `deque`
- vector和deque的结合，使用若干个内存片段进行链接。兼有vector和list的好处。
- 内部数据结构为数组
- 头文件deque

### `array`
大小固定的容器，还需要指定元素类型。


## 顺序容器的操作
### 添加元素（增）
容器添加元素使用的是拷贝一份元素的值到容器中（非引用传参）。
- `c.push_back(t)`在容器尾部插入。除了`array`和`forward_list`，每个顺序容器和`string`都支持。
- `c.push_front(t)`在容器头部插入。`list`, `forwrad_list`和`deque`支持。
- `c.insert(p, t)`在任意位置插入。`vector`, `list`, `deque`, `string`都支持`insert`，`forward_list`有特殊的`insert`。将元素插入到`vector`,`deque`,`string`的任何位置都是合法的，但是非常耗时。
`insert`有多个版本，还可以直接插入一个范围。
如果通过一个迭代器指定插入位置，插入的元素会放在这个迭代器之前，`insert`的返回值是第一个新加入元素的迭代器，如果没有插入任何元素，返回第一个参数。
- `c.emplace(p, args)`是直接构造而不是拷贝元素。`emplace`，`emplace_front`，`emplace_back`分别对应`insert`, `push_front`和`push_back`。

### 删除元素（删）
- `c.pop_back()`，`forward_list`不支持。
- `c.pop_front()`，`vector`和`string`不支持。
- `c.erase(p)`，删除迭代器p指定的内容。
- `c.erase(b, e)`，删除迭代器b和e指定的范围。
- `c.clear()`，删除容器中的所有元素。

### 访问元素（改和查）
下面的四个操作返回的都是引用。
- `c.front()`，返回begin对应的元素。
- `c.back()`，返回end之前的元素，`forward_list`没有。
- `c[n]`，如果`n>c.size()`，无定义，只适用于`string`, `vector`, `deque`, `array`。
- `c.at(n)`，如果下标越界，抛出out of range异常，只适用于`string`, `vector`, `deque`, `array`。

### `forward_list`的操作
`forward_list`提供了`insert_after`, `emplace_after`和`erase_after`。

### 改变容器大小
将容器大小调整为n，n小于c.size()，将超过的舍去；n大于c.size()，使用值初始化或者指定一个元素t。
- `c.resize(n)`，
- `c.resize(n, t)`

### 迭代器失效
1. 容器添加元素之后
2. 从一个容器中删除元素之后

## `vector`的增长
不同的实现中，vector的增长速度也不同，有的是2，有的是1.多。可以使用`c.capaticy`查看vector的容量。capacity和size的区别在于，size指的是它已经保存的元素的数目，而capacity是在不分配新的内存空间的前提下最多可以保存多少元素。


## 容器适配器
适配器接收一种已有的容器类型，让它的行为看起来像另一种不同的类型。标准库中定义了下面三个适配器：
- `stack`
- `queue`
- `priority_queue`

每个适配器都有两个构造函数：
- 默认构造函数创建一个空对象
- 一个构造函数接收容器参数，拷贝该容器初始化适配器。

## 参考文献
1.《C++ Primer》第五版

