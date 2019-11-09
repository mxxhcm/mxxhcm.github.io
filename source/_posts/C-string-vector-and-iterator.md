---
title: C++ string vector and iterator
date: 2019-11-06 16:55:51
tags:
 - string
 - vector
 - iterator
 - C/C++
categories: C/C++
---

## `string`标准库
`string`是标准库的一部分，提供了可变长的字符串，使用时需要包含`string头文件。``` c
#include <string>
using std::string;
```


## `string`对象的初始化
1. 默认初始化，就是简单的声明。```c
string s1;
```
2. 拷贝初始化，使用等号就是拷贝初始化
```c
string s2(s1);
string s3("value");
string s4(10, 'c');
```
3. 直接初始化，不使用等号的初始化。
```c
string s2 = s1;
string s3 = "value";
```

当初始值只有一个的时候，使用直接初始化或者拷贝初始化都行，但是当初始化用到的值有多个时，只能使用直接初始化的方式。

## `string`对象的操作
1. 读写
```c
string s;
cin >> s;   //从遇到的第一个非空白符开始读入，遇到空白符就停止，且不会读入这个空白符
cout << s;
getline(cin, s);    //遇到换行符停止，读入换行符，但是不会将换行符写入s，并且遇到的第一个换行符就算。
```
2. `string`的大小
`empty()`返回`string`是否为空，
`size()`返回`string`的长度。
3. `string::size_type`
`size`返回一个无符号类型`string::size_type`的值，能够容纳放下任何`string`对象的大小。
4. 比较`string`对象
如果两个不同长度的`string`，如果短的`string`和长的`string`的共同部分完全一样，短的小。
如果两个`string`对象在某些位置上的字符不一样，按照字典序排大小，字典序在前的小。
5. `string`赋值
直接赋值字符串常量，
给字符串赋值字符串对象
6. `string`相加
7. 字面值和`string`对象相加
加号两边至少有一个对象是`string`对象

## `string`对象中字符的处理
标准库`cctype`提供了操作`string`中字符的函数。

### 范围`for`(range for)语句
1. 输出每个字符的值
```c
for( auto c: str)
    cout << c << endl;
```
2. 修改每个字符的值，加上引用
```c
for( auto & c: str)
    c = toupper(c);
```

### `[]`下标运算符
1. 迭代访问每个元素

## vector
`vector`表示对象的集合，所有的对象类型都相同，因为`vector`容纳着其他对象，所以`vector`也叫容器。
`vecotr`在头文件`vector`中，使用前需要包含该头文件，并进行`using`声明：
``` c
#include<vector>

using std::vector;
```
`vector`是一个类模板而不是类型。模板本身不是类或者函数，可以将模板看成编译器生成类或者函数编写的一份说明，编译器根据模板创建类或者函数的过程称为实例化，使用模板时需要指出编译器应该把类或者函数实例化成什么类型。
在模板名字后面跟上一对尖括号，括号内为具体的类型说明。
``` c
vector<int> ivec;   //表示int类型的vector
vector< vector<string> > file //vector中是string的vector。
```

## vector对象的初始化
- 默认初始化```c
vector<string> svec;    //默认初始化，不含任何元素
```
- 列表初始化
1. 元素的初值只能放在花括号内，不能放在圆括号内
2. 给定类内初始值时只能使用拷贝初始化或者列表初始化。
``` c
vector<string> c = {"hello", "world"};
```
- 值初始化
1. 只提供给vector对象容纳的元素数量而不用略去初始值。会创建一个值初始化的元素初值，并把它赋给容器中所有元素，这个初始值由`vector`对象中元素类型决定。
2. 如果`vector`对象中元素类型是内置类型，如`int`，初始值自动设置为0。
3. 如果是某种类类型，比如`string`，由类默认初始化。
4. 有些类要求必须显式的给出初始值。如果`vector`中对象不支持默认初始化，必须提供初始值。
5. 如果值提供了元素的数量而没有设定初值，只能使用直接初始化。
6. 初始化的真实含义依赖于传递初始值时用的是花括号还是圆括号。花括号进行列表初始化，圆括号提供的信息构造`vecotr`对象。如果使用花括号中的形式，但是提供的值不能用来列表初始化，考虑使用值构造`vector`对象。但是如果使用圆括号提供不能构造`vector`对象的值，不能用来进行列表初始化。也就是说花括号可以用来列表初始化，也可以用来构建`vector`对象，但是圆括号只能用来构建`vector`对象。


## vector添加元素
直接初始化的方式适用于三种情况：
1. 初始值已知，且数量较少
2. 初始值是另一个`vector`对象的副本
3. 所有元素的初始值都一样。

而当我们并不知道`vector`中有几个元素，元素的值也不知道的时候，就不适用直接初始化了，这个时候需要调用`vector`的成员函数`push_back`向其添加元素。

`vector`的最大优势是运行时能够高效的添加元素，它的容量能够在不够的时候，高效的增长，这样子在定义`vector`对象时设定大小没有太大的意义。只有一种情况，就是所有的元素值都一样的时候，指定`vector`的大小和初始值。否则更建议定义空的`vector`，然后向其中添加具体的而值。

## `vector`对象的操作
1. `vector`的大小
2. 向`vector`添加新的元素
3. 下标`[]`只能用于访问元素，不能用于添加元素。通过下标访问超出`vector`范围的元素的行为非常常见，这种错误叫做缓冲区溢出。

## 迭代器
C++定义了许多容器，包括`vector`，`list`，`map`等等，这些容器中，只有少部分支持`[]`运算符，其他的都不支持，为了访问这些容器，C++提供了迭代器访问这些容器，迭代器和指针类似，提供了对对象的间接访问。
注意`string`不是容器，但是它支持很多和容器类型的操作。

## 迭代器的使用
### 获得迭代器
拥有迭代器的类型（`string`和容器等）也拥有返回迭代器的成员函数，如`begin`和`end`，`begin`返回指向容器第一个元素或`string`第一个字符的迭代器，而`end`返回容器或者`string`的最后一个元素的下一个位置，是一个根本不存在的元素，通常把这个迭代器称为尾后迭代器。
如果容器（`string`)为空，`begin`和`end`返回的是同一个迭代器，都是尾后迭代器。

### 迭代器运算符
迭代器支持的运算有：
- `*iter`，表示返回迭代器iter所指元素的引用
- `iter->num`，解引用`iter`并且获得该元素中名为`num`的成员，等价于`(*iter).mem`，即箭头运算符把解引用和成员访问两个操作结合在了一起。
- `++iter`，让`iter`指向容器中的下一个元素
- `--iter`，让`iter`指向容器中的上一个元素
- `iter1==iter2`
- `iter1!=iter2`，判断两个迭代器是否相等，即是否指向同一个元素或者指向同一个容器的尾后迭代器。

尽量使用`!=`而不是使用`<`，因为所有的标准容器和`string`都定义了`==`和`!=`，而只有部分容器和`string`定义了`<`，所以`==`和`!=`是比较通用的。

### 迭代器的类型
迭代器有const和非const之分。他们通过如下方式定义：
``` c
vector<int>::iterator it;   //非const迭代器，可读和可写
string::iterator it2;

vector<int>::const_iterator it3;    //const迭代器，只是可读
vector<int>::const_iterator it4;
```

### `begin`和`end`
如果对象是常量，`begin`和`end`返回`const_iterator`，否则返回`iterator`。
而`cbgein`和`cend`无论对象是否是常量，都返回`const_iterator`。

### 解引用和成员访问操作相结合
``` c
(*it).empty()
it->empty()
```

### 迭代器失效
1. 范围for语句会使迭代器失效
2. 任何一种可能改变`vector`对象容量的操作，如`push_back`，都会使得`vector`对象的迭代器失效。

## 迭代器运算
所有的容器和`string`都实现了迭代器，迭代器的递增，递减运算，以及`==`和`!=`操作。
`vector`和`string`的迭代器提供了更多的运算符，这些运算被称为迭代器运算。如下所示：
- `iter+n`
- `iter-n`
- `iter+=n`
- `iter-=n`，
- `iter1-iter2`，
- `>, >=, <=, <`

前面四个比较直观，`iter1-iter2`就不行，他其实是两个迭代器之间的距离，`iter`和`iter2`必须是同一个容器中的迭代器。可以使用这些运算符进行二分搜索。
``` c
// text是给定的有序序列
auto begin = text.begin(), end = text.end();
auto mid = text.begin() + (end-begin)/2;
while(mid != end && *mid != sought)
{
    if(sought < mid)
    {
        end = mid;
    }
    else
    {
        begin = mid+1;
    }
    mid = begin + (end-begin)/2;
}

```



## 参考文献
1.《C++ Primer第五版》
