---
title: C++ STL
date: 2019-08-31 11:31:32
tags:
 - C/C++
 - STL
categories: C/C++ 
mathjax: true
---

## STL
### 顺序容器
#### vector
- 内部数据结构为数组，可以自动增长
- 在后端插入和删除，push_back()和pop_back()，时间复杂度为$O(1)$
- 在中间和前段插入和删除，insert()和erase()，时间和空间复杂度是$O(n)$
- 分配连续内存，
- 支持随机数组存取，查找的时间复杂度$O(1)$
- 支持[]访问
- 头文件vector

#### list
- 内部数据结构为双向环状链表
- 任意位置插入和删除的时间复杂度是$O(1)$
- 链式存储，非连续内存
- 不支持随机存取，查找的时间复杂度是$O(n)$
- 不支持[]访问
- 头文件list

#### deque
- vector和deque的结合，使用若干个内存片段进行链接。兼有vector和list的好处。
- 内部数据结构为数组
- 头文件deque

### 关联容器
#### map, multimap
封装了二叉树
#### set, multiset
封装了二叉树

## 命令空间
``` c++
using std::cin;
using std::cout;
using std::endl;
using std;:vector;
using std::string;
```

## 字符串
### string
1. 导包
``` c++
#include<string>
```
2. string转字符串数组
``` c++
string.c_str()
```


## vector
1. 初始化
``` c++
//1.初始化一个size为0的vecotr
vector<int> a; 

//2.初始化size为10，默认值为0的vector
vector<int> b(10);  

//3.初始化size为10，默认值为1的vector
vector<int> c(10, 1);   

//4.使用vector初始化vector
vector<int> d(a);   

//5.使用数组初始化vector
int e = {1, 2, 3};
vector<int> f(e, e+3);
```

2. vector的读取方式
``` c++
//[]和迭代器,at
int a = { 1, 2, 3, 4};
vector <int> b(a, a+ 4);
//1.[]
printf("%d\n", a[3]);
//2.迭代器
for(auto it = b.begin(); it != b.end(); it++)
{
    printf("%d\n", *it);
}
//3.at
printf("%d\n", a.at(3));
```

3. 向量长度
```
vector.size()
```

## vector的vector
1. 声明
``` c++
#include <vector>
using std::vector;

int len = 100;
std::vector< std::vector<int> > a(len);
```

2. 访问
``` c++
#include <vector>
using std::vector;

int len = 0;
std::vector< std::vector<int> > a(len);
std::vector<int> temp;

std::vector< std::vector<int> >::iterator it ;
for(int i = 0; i < len; i++)
{
    for(int j =0; j < len; j++)
    {
        temp.push_back(j);
    }
    a.push_back(temp);
    temp.clear();
}
```

## 参考文献
1.https://blog.csdn.net/xiexievv/article/details/6831194
