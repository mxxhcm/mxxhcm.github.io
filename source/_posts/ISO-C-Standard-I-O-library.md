---
title: ISO C Standard I/O library
date: 2019-11-22 23:28:22
tags:
 - UNIX
 - IO
categories: UNIX
---

## 总结
1. `fgets`, `sprintf`, `snprintf`会在缓冲区数组的结尾加上一个null字节，但是使用的时候不会包含这个字节。
2. `fgets`和`getline`都会读入回车，并且将它存入数组。
3. 每个标准I/O流都有一个和它相关联的文件描述符，可以对一个流调用`fileno`获得它的文件描述符。`fileno`不是ISO C的部分，因为文件描述符不属于ISO C。
4. 标准I/O库的一个不足是效率不高。这和它复制的数据量有关。每当使用一次`fgets`和`fputs`时，通常需要复制两次数据，一次是在用户程序的行缓冲区和标准I/O缓冲区之间，一次是在内核和标准I/O缓冲区之间。
使用`fgets`需要用户指定`fgets`使用的缓冲区，或者使用`getline`，如果传入的指针指向NULL，`getline`会负责分配缓冲区大小。
标准I/O可以设置行缓冲和全缓冲，如果设置缓冲的话也需要一个缓冲区，通常是由系统指定的，也可以通过`setbuf`和`setvbuf`进行更改。而在`setbuf`中，如果`buf`是`NULL`的话，是关闭缓冲区，如果不为空的话，必须是`BUFSIZ`大小。在`setvbuf`中，通过`mode`指定缓冲区的类型，`buf`是`NULL`的话，库函数负责分配缓冲区。否则`buf`是多大就用多大的缓冲区。
`read`和`write`也需要设置缓冲区，这是系统调用级别的，大小可以任意指定，通常使用`sturct stat.st_blksize`的大小。


## 概述
### 特殊符号的ASCII
'\n'是10。
EOF是-1。

### 标准I/O和文件I/O
文件I/O是围绕文件描述符进行的，使用`open`打开一个文件时，返回一个文件描述符，然后使用文件描述符进行后续I/O操作。文件I/O是UNIX相关的实现，其他系统可能有不同的实现，是不跨平台的。
标准I/O是围绕stream进行I/O操作的。当标准I/O库打开或者创建一个文件时，一个流已经和文件相关联。标准I/O库处理很多细节，比如缓冲区分配，使用优化的长度块执行I/O等，使用户不用担心选择多大的block进行I/O会更快。标准I/O库是ISO C标准定义的，不仅仅UNIX系统有实现，凡是支持ISO C标准的操作系统都应该实现，是支持跨平台的。标准I/O在UNIX上需要使用文件I/O实现，在windows等其他系统上就需要其他的实现。

### stream和`FILE`对象
> 12.1 Streams
> For historical reasons, the type of the C data structure that represents a stream is called FILE rather than “stream”. Since most of the library functions deal with objects of type FILE *, sometimes the term file pointer is also used to mean “stream”. This leads to unfortunate confusion over terminology in many books on C.

标准I/O的操作是围绕stream进行的，当打开一个stream时，它返回一个指向FILE类型的指针（通常叫做文件指针）。FILE是一个结构体，包含了标准I/O管理这个stream需要的所有信息，包含用于实际I/O的文件描述符，指向这个流缓冲区的指针，缓冲区的长度，当前缓冲区中的字符等。为了引用一个stream，需要将FILE指针作为参数传递给每个标准I/O函数。

### stream的定向
对于ASCII字符集，一个字符用一个字节表示。对于国际字符集，一个字符用多个字节表示。标准I/O FILE stream可以用于单字节也可以用于多字节字符集。stream的orientation决定了读写的字符是单字节还是多字节，最开始创建stream时，它的orientation没有被确定，使用什么字符的I/O就会将stream的orientation定义为什么。
有两个函数可以改变stream的orientation，它们是`freopen`和`fwide`，原型如下：

### `freopen`和`fwide`原型
``` c
#include <stdio.h>
#include <wchar.h>

int fwide(FILE *stream, int mode);
FILE *freopen(const char *pathname, const char *mode, FILE *stream);
```

### `freopen`和`fwide`性质
1. `fwide`用于设置stream的orientation。如果`mode`为负，是单字节定向的。如果`mode`为正，是多字节定向的。如果`mode`为0，`fwide`确定当前stream的oritentation并返回。
2. `fwide`不能改变已经定向的stream的orientation。
3. `fwide`没有出错返回


### 标准输入，标准输出和标准错误
通常对一个进程预定义了三个stream，它们可以自动的被进程使用。它们是标准输入，标准输出和标准错误，这些stream引用的文件和文件描述符`STDIN_FILENO`,`STDOUT_FILENO`和`STDERR_FILENO`所引用的文件一样。
这三个stream定义在头文件`<stdio.h>`中，通过预定义文件指针`stdin`, `stdou`和`stderr`使用。


## 三种缓冲类型
标准I/O库提供缓冲的目的是尽可能减少`read`和`write`的调用次数，标准I/O库对每个流自动的进行缓冲管理，使得应用程序不用考虑缓冲区的管理。
标准I/O提供了三种类型的缓冲：

### 全缓冲
填满标准I/O的缓冲区之后，进行实际的I/O操作。对于存储在磁盘上的文件通常是由标准I/O实施全缓冲的。在一个流上第一次执行I/O操作时，相关的标准I/O函数调用`malloc`获得需要的缓冲区。
标准I/O库使用flush将输出流缓冲区的内容写到和输出流相关联的文件，缓冲区可以使用标准I/O例程自动的flush，比如当缓冲区填满时，或者缓冲区不满时可以手动调用`fflush`函数进行flush。``` c
int flussh(FILE *fp);
```
任何时候，都可以手动强制冲洗一个流，当`fp`是`NULL`时，冲洗所有的输出流。

### 行缓冲
在行缓冲中，当输入和输出遇到换行符时，标准I/O库执行I/O操作。这允许我们一次输出一个字符，使用标准I/O的`fputc`，但是只有在写了一行之后才能进行实际I/O操作。当涉及一个终端时，通常使用行缓冲。
对于行缓冲来说，有两个限制：
1. 因为标准I/O库用来存放每一行的缓冲区的长度是固定的，所以，只要缓冲区满了，没有遇到换行符，也进行I/O操作。
2. 任何时候只要通过标准I/O库要求从一个不带缓冲的流或者一个行缓冲的流中得到输入数据，那么就会flush所有行缓冲输出流。从行缓冲的流中得到输入数据的一个例子就是从终端按下回车，刚才输入的数据就会立刻从输出流中输出。

### 不带缓冲
标准I/O库不对字符进行缓冲存储。如果将字符传入不带缓冲的输出流中，字符会立即输出到输出流关联的文件或者设备。

### ISO C缓冲标准和UNIX具体实现
ISO C要求：
1. 当且仅当标准输入和标准输出不指向交互设备时，它们才是全缓冲的。
2. 标准错误不会是全缓冲的。

UNIX具体实现：
1. 标准错误不带缓冲
2. 指向终端设备的流，都是行缓冲的，否则是全缓冲的。

### 修改默认缓冲
可以通过`setbuf`和`setvbuf`更改流的缓冲类型。

### `setbuf`和`setvfuf`原型
``` c
#include <stdio.h>

void setbuf(FILE *stream, char *buf);
int setvbuf(FILE *stream, char *buf, int mode, size_t size);
```

### `setbuf`和`setvfuf`性质
1. 这些函数需要在流被打开后调用，因为他们需要文件指针作为参数，而且应该在对流执行任何操作之前调用。
2. 可以使用`setbuf`函数打开和关闭缓冲机制。将`buf`设置为`NULL`，就是关闭缓冲。如果`buf`不为`NULL`，它必须指向一个长度为`BUFSIZ`的缓冲区，通常在这之后就是全缓冲的，如果和终端设备关联，可能会是行缓冲的。
3. `setvbuf`可以通过`mode`指定缓冲的类型，`_IOFBF`是全缓冲，`_IOLBF`是行缓冲，`_IONBF`是不缓冲。指定不缓冲，忽略`buf`和`size`参数。如果指定全缓冲或者行缓冲，`buf`和`size`可以通过`buf`和`size`指定缓冲区的位置和大小。如果指定带缓冲，而`buf`是`NULL`，系统会自动分配`BUFSIZE`大小的缓冲区。
4. 一般而言，应该由操作系统选择缓冲区的长度，并且自动分配缓冲区，这种情况下，关闭流，标准I/O库会自动释放缓冲区。

## 打开一个stream
可以使用`fopen`, `freopen`和`fdopen`函数打开一个standard I/O stream。它们的原型如下：

### `fopen`, `freopen`和`fdopen`原型
```c
#include <stdio.h>

FILE *fopen(const char *pathname, const char *mode);
FILE *fdopen(int fd, const char *mode);
FILE *freopen(const char *pathname, const char *mode, FILE *stream);
```

### `fopen`, `freopen`和`fdopen`性质
1. `fopen`打开路径名为`pathname`的一个文件
2. `fdopen`使用一个已有的文件描述符，并将一个标准I/O stream和该文件描述符结合。**这个函数通常用于由创建管道和网络通信通道函数返回的文件描述符，因为这些特殊文件不能使用标准I/O函数`fopen`打开，所以需要使用设备专用函数获得一个文件描述符，然后使用`fdopen`将文件描述符和一个I/O stream结合。**
3. `freopen`函数在一个指定的stream打开一个指定的文件，如果这个stream已经打开，先关闭这个stream；如果这个stream已经进行了定向，使用`freopen`清楚该定向。**这个函数一般用于将一个指定的文件打开为一个预定义的stream：stdin, stdout和stderr。**
4. `fopen`和`freopen`是ISO C的部分，因为ISO C不包含文件描述符，所以只有POSIX.1有`fdopen`。
5. `mode`有15种取值：`r`, `w`, `a`, `rb`, `wb`, `ab`,`r+`,`r+b`, `rb+`,`w+`,`w+b`,`wb+`, `a+`, `a+b`, `ab+`。对于标准I/O来说，使用`b`可以区分二进制和文本文件。但是对于UNIX来说，二进制和文本文件没有区别，有没有`b`无所谓。
6. 当用追加写时，如果有多个进程用追加写方式打开同一个文件，每个进程的数据都会正确的写入文件中。
7. `fdopen`不会截断也不会创建文件。对于`fdopen`来说，因为需要文件描述符，所以文件必须是打开的，当`mode`是`w`,`wb`时，并不会截断文件，`a`和`ab`也不能用于创建文件，因为文件描述符必须引用一个存在的文件。而如果使用`a+`, `ab+`, `w+`, `wb+`等，这个时候文件已经存在了，不会创建，也不会截断，需要写或者追加就行了，就不会有前半句说的问题了。
8. 使用`a`和`w`相关的`mode`创建文件时，没有办法指定文件的权限位。而POSIX.1要求使用如下的权限创建文件：
`S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IOTH|S_IWOTH`
可以在使用`fopen`等函数之前，使用`umask`指定文件的权限位。
9. 如果流引用终端设备，是行缓冲的，否则是全缓冲的。

### `fclose`函数和性质
``` c
#include <stdio.h>

int fclose(FILE *fp);
```
在文件被关闭之前，flush输出数据。缓冲区中的输入数据被丢弃。如果标准I/O库为这个stream自动分配了缓冲区，释放该缓冲区。
当一个进程正常终止时，所有带未写缓冲数据的标准I/O都被flush，所有打开的标准I/O都被关闭。

## 读写stream
对于一个打开的stream，可以使用3种不同的类型的非格式化I/O以及格式化I/O，对其进行读写操作。
3种非格式化I/O包括：
1. 单字符的I/O。如果流是带缓冲的，标准I/O会负责处理缓冲。
2. 单行的I/O。**这里需要注意一下，单行I/O指定的buffer和标准I/O的buffer不一样，而标准I/O的buffer和`read`等的buffer又不一样。**
3. 直接I/O（direct I/O）。

## `ferror`和`feof`, `clearerr`函数和属性
不管是出错还是到达文件结束，`getc`,`fgetc`和`ungetc`等许多函数都返回同样的值`EOF`，`EOF`是-1，可以使用`ferro`和`feof`判断到底是出错还是到达文件尾端。大多数实现中是为每个流在`FILE`对象中维护了出错标志和文件结束标志，可以使用`clearerr`清除相应的标志。函数的原型如下：``` c
#include <stdio.h>

void clearerr(FILE *stream);
int feof(FILE *stream);
int ferror(FILE *stream);
```

## 单字符I/O
`getc`, `fgetc`和`getchar`函数可以用于一个读一个字符。它们的原型如下：

### `getc`, `fgetc`和`getchar`, `ungetc`原型
``` c
#include <stdio.h>

int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar(void);

int ungetc(int c, FILE *stream);
```

### `getc`, `fgetc`和`getchar`, `ungetc`性质
`getc`和`fgetc`功能一样，只不过`getc`可以被实现为宏，而`fgetc`不能被实现为宏。所以：
1. `getc`的参数不应该是具有副作用的表达式，因为它可能会被计算多次。
2. `fgetc`一定是函数，所以可以得到它的地址。可以当做参数传递给其他函数。
3. `fgetc`的调用时间通常要比`getc`长，因为调用函数的时间通常比调用宏的时间长。

### `ungetchar`函数和属性
1. 从流中读取的数据可以送回流中。
2. ISO C规定可以支持任何次数的回送，但是一次只能送一个字符。
3. 回送的字符可以不是上次读到的字符。
4. 回送的字符不能是`EOF`，但是读到文件尾端时，还可以回送一个字符，因为一次成功的`ungetc`调用会清除`EOF`标志。
5. 用`ungetc`只能将字符写入到标准I/O库的流缓冲区中，并没有将它们写到底层设备或者文件中。

函数的原型如下：``` c
#include <stdio.h>

int ungetc(int c, FILE* fp);
```

### 输出函数`putc`, `fputc`和`putchar`
它们的原型如下：``` c
#include <stdio.h>

int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);     //相当于putc(c, stdout);
```


## 单行I/O
`fgets`和`gets`,`getline`提供了单行输入的功能，单行I/O需要指定一个缓冲区，这个缓冲区是用户自己定义的，是应用程序级别的，它和标准I/O的buffer不一样，我们可以通过`setbuf`和`setvbuf`设置标准I/O的buffer，这是标准I/O即库函数层级的，而`read`和`write`等使用的buffer又是一类buffer，这是系统调用层级的，我们也可以自己指定。
它们的原型如下：
### `fgets`和`gets`,`getline`原型
``` c
#include <stdio.h>

char *fgets(char *s, int size, FILE *stream);
char *gets(char *s);
ssize_t getline(char **lineptr, size_t *n, FILE *stream);
ssize_t getdelim(char **lineptr, size_t *n, int delim, FILE *stream);
```

### `fgets`和`gets`,`getline`性质
1. `gets`从标准输入读，而`fgets`从指定的流中读
2. `gets`不会读入`'\n'`，而`fgets`, `getline`都会读入`'\n'`;
3. `fgets`需要指定缓冲的长度，遇到`"\n"`停止，但是不能超过`n-1`个字符，读入的字符送入缓冲区。缓冲区以`NULL`字节结尾，如果这一行包含最后一个换行符超过了`n-1`个字符，`fgets`只返回一个不完整的行，但是这一行还是以`NULL`结束，下一次调用继续从该行读。
4. `gets`不推荐使用，因为没有指定缓冲区的长度，可能会造成缓冲区溢出，很危险。

### `fputs`和`puts`原型
``` c
#include <stdio.h>

int fputs(const char *s, FILE *stream);
int puts(const char *s);
```

### `fputs`和`puts`性质
1. `fputs`将一个以`NULL`字节结束的字符串写到指定的流中，尾端的`NULL`不输出。这并不是每次输出一行，只有`NULL`前面的字节中包含`'\n'`时，才会输出一行。
2. `puts`不会输出`NULL`字节，但是会自动将字符串后添加一个换行符。
3. `puts`并不像`gets`那样不安全，但是因为自动加了换行符很难受。所以尽量使用`fgets`和`fputs`。


## 二进制直接I/O
除了可以以字符和行为单位进行读取，还可以使用二进制stream进行I/O。

### `fread`和`fwrite`原型
``` c
#include <stdio.h>

size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

### `fread`和`fwrite`性质
1. `fread`和`fwrite`通常用来读写一个二进制数组或者一个结构体。`ptr`是要读写的首地址，`size`是每个对象的大小，`nmemb`是要写的对象的个数。
2. `fread`和`fwrite`返回读写的对象数，读出错或者到达文件结尾，返回的数可以少于`nmemb`。可以调用`ferror`或者`feof`判断是结束还是出错。如果写返回的数值小于`nmemb`，那么就是出错。
3. `fread`只能用于读在同一个系统上的数据，因为不同的系统上二进制文件的格式可能不同。
4. `fread`和`read`的区别，`read`是系统调用，而`fread`是ISO C的函数。`read`的buf大小是字节，而`fread`的size是每个对象的大小，`nmemb`是对象的个数。

## 格式化I/O
除了三种非格式化的I/O，还有标准化I/O函数。标准化I/O函数需要指定格式说明。

### 格式说明
格式说明控制其余参数如何编写，以后如何限制。每个参数按照转换说明编写，转换说明以%号开始。除了转换说明外，格式化字符串中的其他字符都按照原样输出。
一个转换说明由四个可选部分构成：
`%[flags][fldwidth][precision][lenmodifier] convtype`
#### flags
- `'`，将整数按千位分组字符
- '-'，左对齐
- `+`，显示带符号数的正负号
- ` `，如果第一个字符不是正负号，在前面加上一个空格
- `#`，指定另一种形式，比如0x指定十六进制
- `0`，添加前导0而不是空格进行填充

#### fldwitdth
最小宽度，多余字符用空格填充

#### precision
整形转换后最少输出数字位数
浮点数转换后小数点后的最少位数。
字符串转换后最大字节数

精度使用一个`.`，然后跟着一个可选的非负十进制整数或者`x`。

#### lenmodifier
`l`, `ll` , `L`分别表示`long`, `long long`以及`long double`。

#### convtype
- `d`, `i`，有符号十进制
- `o`，无符号八进制
- `u`，无符号十进制
- `x`, `X`，无符号十六禁止
- `f`, `F`，双精度浮点数
- `e`, `E`，指数形式双精度浮点苏
- `g`, `G`
- `a`, `A`，十六进制指数形式双精度浮点数
- `c`，字符
- `s`，字符串
- `p`，指向void的指针
- `n`，
- `%`，一个`%`字符
- `C`，宽字符，等于`lc`
- `S`，宽字符串，等于`ls`

常见的格式化输出函数原型如下：
### `printf`,`frpintf`, `snprintf`,`dprintf`和`fpritnf`原型
``` c
#include <stdio.h>

int printf(const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
int dprintf(int fd, const char *format, ...);
int sprintf(char *str, const char *format, ...);
int snprintf(char *str, size_t size, const char *format, ...);
```

### `printf`,`frpintf`, `snprintf`,`dprintf`和`fpritnf`性质
1. `printf`将格式化数据输出到标准输出
2. `fprintf`将格式化数据写到指定的流。
3. `dprintf`将格式化数据写到指定的文件描述符。
4. `sprintf`将格式化数据送入数组`buf`中，`sprintf`在数组的尾端自动加一个null字节，但是该字符不包含在返回值中。
5. `sprintf`可能会造成`buf`指向的缓冲区溢出，调用者有责任保证该缓冲区足够大。
6. `snprintf`是为了解决缓冲区溢出的问题而引入的，它需要显式指定缓冲区的长度，超过这个长度的话，输入数据都会被丢弃，同样`ssprintf`在数组的尾端自动加一个null字节，但是该字符不包含在返回值中。

### `scanf`, `fscnaf`, `sscanf`原型
``` c
#include <stdio.h>

int scanf(const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char *format, ...);
```

### `scanf`, `fscnaf`, `sscanf`性质
1. `scanf`用于分析输入字符串，并将字符序列转换成指定类型的变量。格式后的各个参数给出了变量的地址，用转换结果对这些变量赋值。
2. 格式说明控制如何转换参数，以便于对他们赋值，除了转换说明和空白字符外，格式字符串中的其他字符必须和输入匹配，如果有一个不匹配，就停止处理。

## 标准I/O效率
`fgets`, `fgetc`, `getc`, `read`这几个函数，哪个效率更高？
当他们同时读取一个300万行的98.5M的程序时，`read`效果最好。它们的系统CPU时间基本一样，但是用户CPU时间查了很多，以及等待I/O的时间也差了很多。为什么呢？
1. 系统CPU时间相同，因为它们对内核提出的读写请求数基本相同。
2. CPU时间差太多是因为，`getc`和`fgetc`需要进行上亿次的循环（上亿个字符），而`fgets`需要进行百万次的循环，而`read`只需要几万次（缓冲区大小设置为4096时）。
3. `fgetc`和`read`缓冲区大小设置为1时，`read`要慢很多，因为`read`调用了两亿次系统调用，而`fget`调用了两亿次函数调用。系统调用的时间和各项开销要比函数调用大得多。


## 定位stream
有三种方法对I/O stream进行定位，分别是`ftell`和`fseek`，`ftello`和`fseeko`，`fgetpos`和`fsetpos`。它们的原型如下：

### `ftell`, `feek`, `ftello`, `fseeko`和`fgetpos`, `fsetpos`原型
``` c
#include <stdio.h>

int fseek(FILE *stream, long offset, int whence);
long ftell(FILE *stream);
void rewind(FILE *stream);

int fgetpos(FILE *stream, fpos_t *pos);
int fsetpos(FILE *stream, const fpos_t *pos);

int fseeko(FILE *stream, off_t offset, int whence);
off_t ftello(FILE *stream);
```

### `ftell`, `feek`, `ftello`, `fseeko`和`fgetpos`, `fsetpos`性质
1. `ftell`和`fseek`假设文件的位置可以存放在一个长整形中，而`ftello`和`fseeko`使用`off_t`代替了长整形。除此之外，它们完全相同。
2. `fgetpos`和`fsetpos`是ISO C的标准，其他是SUS，所以跨平台时，使用`fgetpos`和`fsetpos`。
3. 对于二进制文件，`whence`可以使用`SEEK_SET`, `SEEK_CUR`，这是跨平台的，而`SEEK_END`不是平台的。
4. 对于文本文件，`whence`必须要用`SEEK_SET`且`offset`只能是0或者`ftell`返回的值。


## 实现细节
所有的standard I/O库都要使用到文件的I/O。每个I/O stream都有一个和其相关的文件描述符，可以使用`fileno`函数获得stream的文件描述符。


## 临时文件
ISO C提供了两个函数`tmpnam`和`tmpfile`帮助创建临时文件。它们的原型如下：

### `tmpnam`和`tmpfile`原型
``` c
#include <stdio.h>

char *tmpnam(char *s);
FILE *tmpfile(void);

char *mkdtemp(char *template);
int mkstemp(char *template);
```

### `tmpnam`和`tmpfile`性质
1. `tmpnam`产生一个与现有文件名不同的一个有效路径名字符串。避免使用`tmpnam`。
2. `tmpfile`创建一个临时二进制文件(wb+)，在关闭文件或者程序结束时自动删除这个文件。注意UNIX对于二进制文件不做特殊区分。
3. `tmpfile`函数经常使用的标准UNIX技术是先使用`tmpnam`产生一个唯一的路径名，然后使用它创建一个文件，并且立刻`unlink`它。注意，对一个文件`unlink`之后，如果链接计数等于0，并不立即删除它，因为可能有进程在使用这个文件，关闭文件时才删除文件。
4. `mkdtemp`和`mkstemp`是XSI的扩展部分。
5. `mkstemp`和`mkdtemp`都需要传入一个字符串，它的后六位设置为`XXXXXX`，函数通过将这些占位符替换成不同的字符构建一个唯一的路径名。如果只指定了名字，就创建在当前目录下。
6. `mkdtemp`创建的目录的权限是`S_IRUSR`,`S_IWUSR`, `S_IXUSR`。`mkstemp`创建的文件的权限是`S_IRUSR`,`S_IWUSR`，可以使用`umask`进行修改。
7. `mkstemp`创建的文件不会被自动删除。

## 内存stream
Standard I/O把数据缓存在内存中，因此字符和单行的I/O更有效一些，我们也可以使用`setbuf`和`setvbuf`让标准I/O库使用自己指定的缓冲区。
在SUS4之后添加了对memory streams的支持，这些standard I/O streams没有底层文件支持，但是仍然可以使用FILE指针访问，所有的I/O都是通过在缓冲区和主存中来回交换字节实现的。这些流虽然看起来像文件流，但是某些特征更像字符串操作。

有三个函数可以用于内存流的创建，它们分别是`fmemopen`，`open_memstream`和`open_wmemstream`。
### `fmemopen`函数和属性
``` c
#include <stdio.h>

FILE *fmemopen(void *buf, size_t size, const char *mode);
```
1. `fmemopen`函数open memory as stream
2. `fmemopen`函数允许调用者提供缓冲区用于memory stream，`size`指定了缓冲区大小的字节数。如果`buf`为空，`fmemopen`会分配`size`字节数的缓冲区，流关闭时缓冲区会被释放。
3. `type`和`fopen`的取值一样，总共有15种取值，

### `open_memstream`和`open_wmemstream`函数和属性
``` c
#include <stdio.h>

FILE *open_memstream(char **ptr, size_t *sizeloc);

#include <wchar.h>

FILE *open_wmemstream(wchar_t **ptr, size_t *sizeloc);
```
它们一个面向字节，一个面向宽字节。它们和`fmemopen`之间的区别：
1. 创建的流只能打开；
2. 不能指定自己的缓冲区，但是可以访问缓冲区地址和大小。
3. 关闭流后需要自己释放缓冲区
4. 对流添加字节会增加缓冲区大小。
5. 缓冲区地址和长度只有在调用`fclose`或者`fflush`后才有效。这些值只有在下一次流写入或者调用`fclose`前。

## 标准I/O的替代软件
标准I/O库的一个不足是效率不高。这和它复制的数据量有关。每当使用一次`fgets`和`fputs`时，通常需要复制两次数据，一次是在用户程序的行缓冲区和标准I/O缓冲区之间，一次是在内核和标准I/O缓冲区之间。
OK，这章我就认识到了这一个很重要的知识点。。
使用`fgets`需要用户指定`fgets`使用的缓冲区，或者使用`getline`，如果传入的指针指向NULL，`getline`会负责分配缓冲区大小。
标准I/O可以设置行缓冲和全缓冲，如果设置缓冲的话也需要一个缓冲区，通常是由系统指定的，也可以通过`setbuf`和`setvbuf`进行更改。而在`setbuf`中，如果`buf`是`NULL`的话，是关闭缓冲区，如果不为空的话，必须是`BUFSIZ`大小。在`setvbuf`中，通过`mode`指定缓冲区的类型，`buf`是`NULL`的话，库函数负责分配缓冲区。否则`buf`是多大就用多大的缓冲区。
`read`和`write`也需要设置缓冲区，这是系统调用级别的，大小可以任意指定，通常使用`sturct stat.st_blksize`的大小。
``` c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);

# include <stdio.h>
void setbuf(FILE *stream, char *buf);
void setbuffer(FILE *stream, char *buf, size_t size);

char *fgets(char *s, int size, FILE *stream);
```

## 参考文献
1.《APUE》第三版
2.https://stackoverflow.com/questions/20937616/what-is-the-difference-between-a-stream-and-a-file
