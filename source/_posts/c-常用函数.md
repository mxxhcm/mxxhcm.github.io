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
1. 向量长度
```
vector.size()
```

2. 访问元素

## 参考文献
