---
title: c
date: 2019-09-17 17:12:35
tags:
 - C
categories: C/C++
mathjax: true
---

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

## c类型字符串数组
1. 导包
``` c++
#include <string.h>
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

