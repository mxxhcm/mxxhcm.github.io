---
title: c++常用函数
date: 2019-08-31 11:31:32
tags:
categories:
---

## 命令空间
``` c++
using std::cin;
using std::cout;
using std::endl;
```

## 常用库函数
``` c++
#include < vector>
```

## 获得数组长度
``` c++
int a[] = {1, 2, 3, 4};
l = sizeof(a)/sizeof(int);
```
## printf输出格式
%d  有符号十进制整数(int)
%ld, %Ld    长整形数据(long)
%i  有符号十进制数，和%d一样
%u  无符号十进制整数(unsigned int)  
%lu, %Lu    无符号十进制长整形数据(unsigned long)
%f  单精度浮点数(float)
%c  单字符(char)
%o  无符号八进制整数
%x(%X) 十六进制无符号整数


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

### c类型字符串
1. 导包
``` c++
#include <cstring>
```

2. 完整字符串复制
``` c++
strcpy(des, src);
```

3. 部分字符串复制
``` c++
strncpy(des, src+n, len);
```

4. 结束符
``` c++
sub[len] = '\0';
```
5. new字符串数组
``` c++
char *str = new char[100];
```

6. delete字符串数组
``` c++
delete []str;
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
