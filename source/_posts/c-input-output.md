---
title: c input and output
date: 2019-10-27 10:00:47
tags:
 - C
 - 标准输入/输出
 - 文件输入/输出
categories: C/C++
---

## ASCII
'\n'是10。
EOF是-1。

## 缓冲区
我们在使用输入输出函数的时候，不管是从文件还是控制台中读取数据，数据都会先存放在缓冲区里面，在需要使用的时候会在缓冲区里面提取，缓冲区是一个队列，遵循先进先出的规则。

例如：
``` C
#include<stdio.h>
int main() {
    int a, b;
    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);

    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);

    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);
    return 0;
}
```
在控制台输入：
``` txt
12 23 34 45 56 67 78 89
```
控制台输出结果：
``` txt
12 23
34 45
56 67
```

由于输入在控制台的数据已经到了缓冲区，所以除了第一次调用scanf()的时候控制台会弹出（缓冲区为空），另两次则不会弹出（缓冲区不为空），直接从缓冲区里面拿取数据。
如果不想这样做可以选择清空缓冲区，可以使用fclose()函数清空缓冲区并关闭流，但是这样我们就无法继续使用流了。我们还可以选择使用fflush()函数，在不关闭流的情况下清空缓冲区。

``` C
int fflush ( FILE * stream );
```
如果给出的文件流是一个输出流,那么fflush()把输出到缓冲区的内容写入文件；如果给出的文件流是输入类型的,结果未定义；
fflush(NULL)刷新所有的输出流；
fflush(stdin)刷新标准输入缓冲区，把输入缓冲区里的东西丢弃；但是fflush[在linux上并不work](https://stackoverflow.com/questions/17318886/fflush-is-not-working-in-linux)。
fflush(stdout)刷新标准输出缓冲区，把输出缓冲区里的东西打印到标准输出设备上。
例如
``` c
#include<stdio.h>
int main() {
    int a, b;
    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);
    fflush(stdin);

    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);
    fflush(stdin);

    scanf("%d %d", &a, &b);
    printf("%d %d\n", a, b);
    return 0;
}
```
在控制台输入：
```
12 23 34 45 56 67 78 89
```
控制台输出
```
12 23
```
然后会等待控制台输入。（我在linux上并没有实验成功。）

## 文件输入/输出
```
//字符
fgetc() and fputc()
getc() and putc()

//n个项
fread() and fwrite()

//字符串
getline() 
fgets() and fputs()
fscanf() and fprintf()
```
 
### 打开文件`fopen`和关闭文件`close`
#### 函数原型
``` C
/* 
 * 打开文件，打开成功返回一个FILE类型的指针，打开失败返回NULL
 * filename：是待打开文件的名称
 * type：是文件操作方法，包括'w', 'a', 'rb', 'wb', 'ab', 'r+', 'w+', 'a+', 'rb+', 'wb+', 'ab+'
 */
FILE* fopen(char*filename, char* type);

/*
 * 关闭FILE指针
 * FILE* stream: 文件流
 */
int fclose(FILE* stream);
```

#### 示例
``` c
#include <stdio.h> 
#include <stdlib.h>

int main() 
{ 
    char filename[20] = "data.txt";
    FILE *fp = fopen(filename, "r"); 
    if (fp == NULL) 
    { 
        printf("Cannot open file.\n"); 
        exit(0); 
    }
    else
    {
        printf("Open file %s success.\n", filename);
    }
    fclose(fp);

    return 0;
}
```

### 从文件流中读写字符
#### 读字符
``` C
/*
 *
 * 从文件流中读取单个字符，会读取'\n'字符
 * FILE* stream: 文件流
 *
 */
int fgetc(FILE* stream);

/*
 * getc()和fgetc()功能功能一样，只不过getc()是宏实现，进行了优化，getc()可以被当做宏调用。
 * 而fgetc()只能当做函数被调用。
 * getc()也能读取'\n'字符。
 * FILE* stream: 文件流
 *
 */
int getc(File* stream);
```

#### 写字符
``` C
/*
 * 将一个字符写入文件流中
 * int ch: 待写入字符
 * FILE * stream：输出文件流
 */
int fputc(int ch, FILE* stream);


/*
 *
 * 将一个字符写入文件流中
 * 和fputc()的区别是putc()是宏实现，进行了优化
 * int ch: 待写入字符
 * FILE * stream：输出文件流
 *
 */
int putc(int ch, FILE* stream);
```

### 从文件流中直接读取
#### 读取多个字节
``` C
/*
 * 从流指针指定的文件中读取nobj个数据项，每个数据项有size个字节，存入ptr指向的缓冲区。读取的数据项不一定等于nobj，可能读完了或者出错了。
 * ptr:缓冲区指针
 * size_t size: 每个数据项字节
 * size_t nobj: 读取多少个数据
 * FILE* stream: 流指针
 * 
 */
size_t fread(void* ptr, size_t size,size_t nobj,FILE* stream);
```

#### 写入多个字节
``` C
/*
 *
 * 和fread一样，不过这个是写入
 *
 */
size_t fwrite(const void* ptr,size_t size,size_t nobj,FILE* stream);
```

### 从文件流中读写字符串
#### 读字符串
``` C
/*
 * 从文件中读取一行
 *
 *
 */
ssize_t getline(char **lineptr, size_t *n, FILE *stream);

/*
 *
 * fgets从流文件中n个字符到指定数组中，遇到 "\n"就停止。
 * char* str: 字符数组或者字符串指针，字符串指针一定要配内存空间，否则会出问题
 * int n: 从流指针开始读取n个字符。
 * 返回值：
 *      n <= 0或者读入错误，或者EOF，返回NULL
 *      n = 1返回空串 
 *      读入成功返回缓冲区地址
 *
 */
char* fgets(char* str, int n, FILE* stream);

/*
 *
 *
 *
 */
int fscanf(FILE* stream, char* format, variable-list);
```

#### 写字符串
``` C
/*
 *
 * fputs将字符串输出到指定的流指针中。
 *
 */
int fputs(char* str, FILE* stream);

/*
 *
 *
 *
 */
int fprintf(FILE* stream, char* format, variable-list);
```

## 标准输入/输出
### 读写字符
#### 读字符
``` C
/*
 *
 * scanf函数的格式说明符有几个就要取几次数据，
 * 只要碰到格式说明符就必须把数据取走，没取完
 * 的数据继续留在缓冲区中。
 * scanf()加%c
 *
 */
scanf()

/*
 *
 *
 * 将用户输入的字符输出到标准输出设备，也是按下回车后从缓冲区读取。
 *
 */
getchar()

/*
 *
 * 不读取缓冲区的字符，只要用户输入字符，getche()函数会立刻读取，而不需等待按回车键，并在屏幕上显示读入的字符。
 *
 */
getche()

/*
 *
 * 它与getche()的区别是，getch()不需将所输入的字符显示到屏幕上
 *
 */
getch()
```

#### 写字符
``` C
/*
 *
 * 加%c
 *
 */
printf(); 

/*
 *
 *
 *
 */
putchar()
```

### 读写字符串
#### 读字符串
``` C
/*
 *
 *
 * gets
 * /Never use gets(). 别用gets()，因为get()没有指定可以使用的缓冲区大小，它会在遇到换行符或者EOF才能停止，把所有的字符都存在缓冲区中，然而因为不知道要读多少个字符，所以可能会发生溢出。
 * fgets()指定了缓冲区大小，所以很安全。
 *
 */
char *gets(char *s);

/*
 * scanf
 * scanf()加%s，但是%s会在遇到空白符（空格，tab）时自动结束，而gets和fgets()都是以换行或者EOF为结束。
 * Avoid using scanf. 如果用的不仔细，会和gets发生一样的溢出。
 * 
 *
 */
scanf();

/*
 * fgets
 * 在文件字符串读中介绍过fgets()，建议使用fgets()。这个是文件读写。
 *
 *
 */
char* fgets(char* str, int n, FILE* stream);

/*
 * getline 
 * lineptr需要的参数是指针的指针，如果只传指针的话，如果lineptr是NULL的话，就会出错。
 * 其中*lineptr指向一个动态分配的内存区域。*n是所分配内存的长度。如果*lineptr是NULL的话，getline函数会自动进行动态内存的分配（忽略*n的大小），所以使用这个函数非常注意的就使用要注意自己进行内存的释放。
 * 如果*lineptr分配了内存，但在使用过程中发现所分配的内存不足的话，getline函数会调用realloc函数来重新进行内存的分配，同时更新*lineptr和*n。
注意*lineptr指向的是一个动态分配的内存，由malloc，calloc或realloc分配的，不能是静态分配的数组。
 * 读入换行符。
 *
 *
 */
ssize_t getline(char **lineptr, size_t *n, FILE *stream);
```

#### 写字符串
``` C
/*
 *
 *
 *
 */
printf() 加%s

/*
 *
 * 以"\0"作为字符串的结束。
 *
 *
 */
puts();
```

### 其他
可以定义文件指针，指向`stdin`和`stdout`，文件的读写函数就也可以使用到标准输入输出了。
如下示例：
``` C
#include <stdio.h>

void main()
{
    int ch;

    FILE *pfin = stdin; //定义一个文件指针，并指向标准输入设备(键盘)
    FILE *pfout = stdout; //定义一个文件指针，并指向标准输出设备(屏幕)
    
    printf("Enter a string: ");
    
    ch = getc(pfin); //使用getc()函数获取缓冲区中的第一个字符

    putc(ch, pfout); //使用putc()函数输出该字符
    putc('\n', pfout); //使用putc()函数输出换行字符
}
```

## 文件输入示例
### 从文件中一行一行的读取数字
有data.txt如下所示：
```txt
13
38
28
```
读取程序如下：
```
#include <stdio.h> 
#include <stdlib.h>

#define N 510

int main() 
{ 
    char filename[20] = "data.txt";
    FILE *fp = fopen(filename, "r"); 
    if (fp == NULL) 
    { 
        printf("Cannot open file \n"); 
        exit(0); 
    } 

    char chunk[128];
    int array[N] = {0};
    int count = 0;
    while (fgets(chunk, sizeof(chunk), fp)!=NULL) 
    { 
        array[count] = atoi(chunk);
        count += 1;
    } 
    fclose(fp); 
    return 0; 
}
```

## 参考文献
1.https://blog.csdn.net/strongwangjiawei/article/details/7786085
2.https://solarianprogrammer.com/2019/04/03/c-programming-read-file-lines-fgets-getline-implement-portable-getline/
3.https://stackoverflow.com/questions/20378430/reading-numbers-from-a-text-file-into-an-array-in-c
4.http://manpages.ubuntu.com/manpages/bionic/man3/fgetc.3.html
5.https://www.cnblogs.com/JCSU/articles/1306308.html
6.https://www.cnblogs.com/JCSU/articles/1306308.html
7.https://blog.csdn.net/baidu_27435045/article/details/53313699
8.https://stackoverflow.com/questions/17318886/fflush-is-not-working-in-linux
