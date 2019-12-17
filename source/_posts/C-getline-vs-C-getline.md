---
title: C getline vs C++ getline
date: 2019-11-13 13:11:10
tags:
 - C/C++
 - string
 - C风格字符串
 - 数组
categories: C/C++
---

## getline in C
### 原型
使用`man getline`可以看到UNIX提供的库函数`getline`的原型。
``` c
#include <stdio.h>

ssize_t getline(char **lineptr, size_t *n, FILE *stream);
ssize_t getdelim(char **lineptr, size_t *n, int delim, FILE *stream);
```

### 性质
1. `getline()`从stream中读入一整行，
2. 如果`*lineptr`设置为NULL并且`*n=0`，`getline()`会分配一个buffer存储读入的line。这个buffer应该被用户程序释放，即使`geline()`失败了。
3. 如果`*lineptr`包含一个指针，大小是`*n`字节。当buffer不能存下读入的line时，`getline()`会使用`realloc(3)`对buffer进行resize，更新`*lineptr`和`*n`。
4. 只要成功调用，`*lineptr`和`*n`分别表示的是buffer的地址和分配的内存大小。
5. `getdelim()`和`getline()`一样，只不过可以指定一个delimiter而不是使用newline作为delimiter。任何delimiter都会存进`*lineptr`中。
6. 为什么`getline`需要的是`char**`而不是`char*`，因为`getline`在lineptr指向的空间不足时，重新分配内存，如果使用的是`char*`的话，当`getline`重新分配内存后，我们就失去了对`line`的访问，而使用一个`char**`类型的字符串，使用一个`char**`类型，即指针的指针记录每次分配的`char*`。[3]。

## getline in C++
### cin.getline
`cin.getline()`是操作C strings，即字符数组的。不会将`'\n'`读入。

### std::getline
1. `std::getline()`是操作C++ strings的，即`std::string`。
2. `std::getline()`从input stream中读一个string，遇到delimiter就停止，默认的delimiter是`'\n'`。即使输入开始就是delimiter也会停止。
3. `std::getline()`会把delimiter也读进来，然后把读到的内容存到`string`对象中去，存入的内容不包含delimiter。

## 代码示例
### getline
``` c
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    FILE *stream = stdin;
    char *line = NULL;
    size_t len = 0;
    ssize_t nread;

    while ((nread = getline(&line, &len, stream)) != -1) {
        printf("Retrieved line of length %zu:\n", nread);
        fwrite(line, nread, 1, stdout);
    }

    free(line);
    exit(EXIT_SUCCESS);
}
```

### getdelim
```c
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    FILE *stream = stdin;
    char *line = NULL;
    size_t len = 0;
    ssize_t nread;
    int delim = ',';

    while ((nread = getdelim(&line, &len, delim, stream)) != -1) {
        printf("Retrieved line of length %zu:\n", nread);
        fwrite(line, nread, 1, stdout);
    }

    free(line);
    exit(EXIT_SUCCESS);
}
```

## 参考文献
1.https://stackoverflow.com/questions/4872361/why-are-there-two-different-getline-functions-if-indeed-there-are
2.https://www.reddit.com/r/learnprogramming/comments/4fx64h/is_there_a_difference_between_cingetline_and/
3.https://stackoverflow.com/questions/5744393/why-is-the-first-argument-of-getline-a-pointer-to-pointer-char-instead-of-c/36098042
