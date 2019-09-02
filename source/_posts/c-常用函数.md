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
导包
``` c++
#include<string>
```

### c类型字符串
#### 导包
``` c++
#include <cstring>
```

#### 完整字符串复制
``` c++
strcpy(des, src);
```

#### 部分字符串复制
``` c++
strncpy(des, src+n, len);
```
#### 结束符
``` c++
sub[len] = '\0';
```
#### new字符串数组
``` c++
char *str = new char[100];
```

#### delete字符串数组
``` c++
delete []str;
```

### vector
#### 向量长度
```
vector.size()
```

#### 访问元素

## 参考文献
