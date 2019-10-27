---
title: c input and output
date: 2019-10-27 10:00:47
tags:
 - C
 - 标准输入/输出
 - 文件输入/输出
categories: C/C++
---

## 缓冲区
我们在使用输入输出函数的时候，不管是从文件还是控制台中读取数据，数据都会先存放在缓冲区里面，在需要使用的时候会在缓冲区里面提取。
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
12 23 34 45 56 67 78 89
控制台输出结果：
12 23
34 45
56 67

由于输入在控制台的数据已经到了缓冲区，所以除了第一次调用scanf()的时候控制台会弹出，另两次则不会弹出，缓冲区不为空，直接从缓冲区里面拿取数据。
如果不想这样做可以选择清空缓冲区，我们可以使用fclose()函数清空缓冲区并关闭流，这样我们就无法继续使用流了。因此我们可以选择使用fflush()函数，在不关闭流的情况下清空缓冲区。

``` C
int fflush ( FILE * stream );
```
如果给出的文件流是一个输出流,那么fflush()把输出到缓冲区的内容写入文件；
如果给出的文件流是输入类型的,结果未定义；
fflush(NULL)刷新所有的输出流；
fflush(stdin)刷新标准输入缓冲区，把输入缓冲区里的东西丢弃；
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
12 23 34 45 56 67 78 89
控制台输出
12 23
然后会等待控制台输入，这就是C语言的输入输出的大致运行模式

## 文件输入/输出
### 打开文件`fopen`和关闭文件`close`
#### 函数原型
``` C
//打开文件，打开成功返回一个FILE类型的指针，打开失败返回NULL
// filename是待打开文件的名称
// type是文件操作方法，包括'w', 'a', 'rb', 'wb', 'ab', 'r+', 'w+', 'a+', 'rb+', 'wb+', 'ab+'
FILE* fopen(char*filename, char* type);

//关闭FILE类型指针
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
		printf("Cannot open file \n"); 
		exit(0); 
	}
    fclose(fp)
}
```

### 从文件流中读写字符
#### 读字符
``` C
//读取字符
int fgetc(FILE* stream);

//getc和fget()功能功能一样，而getc是宏实现，进行了优化，getc可以被当做宏调用。而fgetc只能当做函数被调用。
int getc(File* stream);
```

#### 写字符
``` C
int fputc(int ch, FILE* stream);
int putc(int ch, FILE* stream);
```

### 从文件流中直接读取
#### 读取多个字节
``` C
size_t fread(void* ptr,size_t size,size_t nobj,FILE* stream);
```

#### 写入多个字节
``` C
size_t fwrite(const void* ptr,size_t size,size_t nobj,FILE* stream);
```

### 从文件流中读写字符串
#### 读字符串
``` C
char* fgets(char* str, int n, FILE* stream);

int fscanf(FILE* stream, char* format, variable-list);
```

#### 写字符串
``` C
int fputs(char* str, FILE* stream);

int fprintf(FILE* stream, char* format, variable-list);

```


## 标准输入/输出
### 读写字符
#### 读字符
``` C
scanf()加%c
//缓冲区是一个先进先出的队列，即取走数据的时候，遵循先输入的数据先取走的原则。scanf函数的格式说明符有几个就要取几次数据，只要碰到格式说明符就必须把数据取走，至于是不是要把取走的数据存放起来，就得看数据列表中的数据个数。没取完的数据继续留在缓冲区中。

getchar()
//将用户输入的字符输出到标准输出设备，也是从缓冲区读取。

getche()
//不读取缓冲区的字符，只要用户输入字符，getche()函数会立刻读取，而不需等待按回车键，并在屏幕上显示读入的字符。

getch()
//它与getche()的区别是，getch()不需将所输入的字符显示到屏幕上
```

#### 写字符
``` C
printf(); 
// 加%c
putchar()
```

### 读写字符串
#### 读字符串
``` C
// 1.gets
char *gets(char *s);
//Never use gets(). 别用gets()，因为get()没有指定可以使用的缓冲区大小，它会在遇到换行符或者EOF才能停止，把所有的字符都存在缓冲区中，然而因为不知道要读多少个字符，所以可能会发生溢出。fgets()指定了缓冲区大小，所以很安全。

// 2.scanf
scanf();
//scanf()加%s，但是%s会在遇到空白符（空格，tab）时自动结束，而gets和fgets()都是以换行或者EOF为结束。
//Avoid using scanf. 如果用的不仔细，会和gets发生一样的溢出。

// 3.fgets
fgets();
//建议使用fgets()。这个是文件读写。

// 4.getline
getline();

```

#### 写字符串
``` C
printf() 加%s

puts();
//以"\0"作为字符串的结束。
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
