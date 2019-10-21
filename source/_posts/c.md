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

## 生成随机数
``` c
#include <time.h>
#include <stdlib.h>

srand(time(NULL));   // Initialization, should only be called once.
int r = rand();      // Returns a pseudo-random integer between 0 and RAND_MAX.
```

## 计算函数运行时间
### 计算实际运行时间
``` c
#include<cstdlib>
#include<cstdio>
#include<ctime>
#include<sys/time.h>
#include<unistd.h>


//struct timeval
//{
//     long tv_sec;//秒域
//     long tv_usec;//微秒域
//}
//int getimeofday(struct timeval* tv,NULL);

int main()
{

    struct timeval  tv1, tv2;
    gettimeofday(&tv1, NULL);
    sleep(2);
    gettimeofday(&tv2, NULL);

    printf ("Total time = %f seconds\n",
         (double) (tv2.tv_usec - tv1.tv_usec) / 1000000 +
         (double) (tv2.tv_sec - tv1.tv_sec));
    return 0;
}
```

### 计算cpu运行时间
clock()返回的是OS花了多少时间运行当前的进程。
``` c
#include<ctime>

int main()
{

    clock_t start, end;
    start = clock();
    time.sleep(10);
    end = clock()
    print("%f", (double)(end-start)/CLOCK_PER_SEC);
}
```
## 参考文献
1.https://stackoverflow.com/questions/822323/how-to-generate-a-random-int-in-c
2.https://blog.csdn.net/zhangwei_zone/article/details/11219757
3.https://stackoverflow.com/questions/5248915/execution-time-of-c-program
