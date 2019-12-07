---
title: C/C++ compound type pointer
date: 2019-11-13 14:11:26
tags:
 - 指针
 - 复合类型
 - C/C++
categories: C/C++
---

## 指针
1. 变量的值存储在计算机的内存中，每个变量都占据一个特定的位置，每一个内存位置都由地址唯一确定并引用。指针可以看成是地址的另一个名字[2](3.1.3)。
2. 指针变量也是一个变量，其中存放的是另一个变量的地址，因为指针是一个变量，所以指针变量本身也存放在内存中的某个位置[2](3.1.3)。允许对指针赋值和拷贝，在指针的生命周期内可以先后指向几个不同的对象。
3. 指针无须在定义时赋值，和其它内置类型一样，在块作用域内定义的指针如果没有被初始化，也将拥有一个不确定的值。

## 指针声明和定义
指针也是一个复合类型，需要按照复合类型的声明和定义进行声明。

### 指针定义
定义指针类型的方法将声明符写成`*d`的形式，其中`d`是变量名。如果在一条语句中定义了几个指针变量，每个变量前面都必须有符号`*`。即`*`是修饰声明符的，而不是修饰`int`的。如下所示：
``` c
int *p; // 定义一个int*的指针p，p进行了默认初始化
int *a, b, *c;  //定义了两个int*类型的指针a,c，一个int类型变量b

char *message = "Hello world!"; //定义一个char*变量，指向数据段的字符串常量区中的字符串"Hello world!"的首字符的地址。
//下面两行代码和上面一行代码的作用是相同的，message是一个`char *`指针，指向字符串常量"Hello world!"的首字符的地址。
char *message = NULL;
message = "Hello world!"; 
```

### 取地址符
指针存放某个对象的地址，要想获取改地址，需要使用取地址符`&`。除了两种特殊情况外，所有指针类型都要和它所指的对象严格匹配。
两种特殊情况：
1. 一个指向常量的指针可以指向个非常量对象。
2. ...

### 指针的值，就是地址，有四种可能取值
1. 指向一个对象
2. 指向紧邻对象所占空间的下一个位置
3. 空指针，没有指向任何对象
4. 无效指针，除了以上三种情况的任何值。

使用无效指针和使用未初始化变量是同类错误，编译器都不负责进行检查。

### 解引用指针（间接访问）操作符
C11中解引用操作符`*`的定义：
> The unary * operator denotes indirection. If the operand points to a function, the result is a function designator; if it points to an object, the result is an lvalue designating the object. If the operand has type ‘‘pointer to type’’, the result has type ‘‘type’’. If an invalid value has been assigned to the pointer, the behavior of the unary * operator is undefined.102)

C++11中解引用操作符`*`的定义：
> The unary * operator performs indirection: the expression to which it is applied shall be a pointer to an object type, or a pointer to a function type and the result is an lvalue referring to the object or function to which the expression points. If the type of the expression is “pointer to T,” the type of the result is “T.” [ Note: a pointer to an incomplete type (other than cv void) can be dereferenced. The lvalue thus obtained can be used in limited ways (to initialize a reference, for example); this lvalue must not be converted to a prvalue, see 4.1. — end note ]

解引用操作符`*`应用于指向对象或者指向函数的指针表达式，得到指针指向对象的左值表达式，给解引用的结果赋值其实就是给指针所指的对象赋值([1]2.3.2)。如果不解引用，指针变量中存放的内容就只是地址。
**对象（变量）和指针变量中存放的内容（即地址）的区别，对象可以直接进行赋值。指针变量中存放的是一个地址，地址本身就是一个数字，是一个右值，不能对其进行赋值，对这个地址进行解引用，得到指针指向的对象。**
定义一个指针`p`：``` c
int a=3; 
int *p = &a;
```
`p`是`int*`类型，存放的是变量`a`的地址，`*`是间接访问，`*p`对指针进行解引用得到指针指向对象的左值表达式，其实就是`a`，`&`表示取变量`a`的地址。

### 未初始化和非法的指针
看一个错误的代码片段：```c
int *a; //定义一个指针*p
*a = 12;    //把12存储在a指向的内存中，错误
```
这个代码中犯了一个很严重的错误，我们在声明了变量`a`，但是没有对它进行显示初始化，所以编译器会对a进行默认初始化，默认初始化并不会为变量分配内存。如果程序执行这个赋值操作，假如a是一个非法地址，程序会出错，终止程序，在UNIX系统上，这个错误称为"segmentation violation"或者"memory fault"，它告诉我们程序正在访问一个非法的地址。如果a是一个合法的地址，这就会错误的修改a指向的内存中的值，造成一些难以预料到的错误。

### 空指针
有以下几种方法声明空指针：``` c
// 方法1.
int *p1 = nullptr;
// 方法2.
int *p2 = 0;
// 方法3.
int *p3 = NULL;//NULL定义在cstdlib中
```
最好使用`nullptr`或者`0`，而避免使用`NULL`。

### `void*`指针
`void *`指针可以存放任意类型的地址，但是我们并不知道它存放的是什么类型的对象。

### 指针的指针
因为指针也是一个变量，所以它自然也就有地址，也就存在指向指针的指针。``` c
int a, *b, **c;
b = &a;
c = &b;   //紧挨着c左边的那个*表示c是一个指针，然后再往左边的那个`*`是和`int`在一起的，表示指针`c`指向的变量的类型是`int *`类型的。
// a,*b, **c表示同样的东西，都是变量a的值。
// *c和b和&a表示同样的东西，都是变量a的地址。
// a表示int类型的变量
// b是a的地址，*b表示对指针p解引用，*b就是a。
// c是b的地址，*c表示对指针c解引用，*c就是b，也就是a的地址，*(*c)也就是*b，也是a。
```

## 指针的特点
1. 存放的是对象的地址，要想获取变量的地址，需要使用取地址符`&`，访问指针中地址指向的变量，使用解引用符号`*`，即：```c
int val = 32;
int *p = & val; //指针p存放的是变量val的地址
int b = *p; //b被初始化为32,p存放的是val的地址，*p获得该地址指向的变量val。
*p = 3; //将val赋值为3
```
2. 赋值永远改变的是`=`左侧的对象，可以用来判断到底是改变了指针的值还是改变了指针所指的对象的值。
3. 除了两种特殊情况外，所有指针的类型都必须和它指向的对象严格匹配。

## 指针和引用的不同点
1. 指针本身就是一个对象，允许赋值和拷贝，在生命周期内可以指向几个不同的对象；而引用本身并非一个对象，一定定义了引用，它就和一个对象终生绑定在了一起。
2. 指针定义时无须赋值；而引用必须在定义时赋值。

## 指针运算
指针加上一个整数的结果是另一个指针。注意，这里假设每种类型都是连续存储的。假设字符型占一个字节，`float`占四个字节，`double`占八个字节。
如果将指针加上1：
对于一个字符型，新的指针指向内存中的下一个字符，指针的值实际上增加了1。
对于一个`float`型，它指向内存中的下一个`float`，指针的值实际上增加了4。
对于一个`double`型，它指向内存中的下一个`double`，指针的值实际上增加了8。
也就是说，对于一个给定类型的指针，将它加一，得到的新指针指向下一个同类型的变量，这也是声明指针类型的作用。

### 指针的大小
任何类型的指针本身所占的大小都是相等的，取决于计算机的地址大小，如果是`32`位的地址，指针的大小就是`4`个字节，如果是`64`位的地址，指针的大小就是`8`个字节。!!!!这是错误的。。
> The size of a pointer depends on many factors - including the CPU architecture, compiler, Operating System etc.
Usually the size is equal to the word size of the underlying processor architecture, and the size of total addressable memory (including virtual memory).
So, for a 32bit computer, the pointer size can be 4 bytes; 64bit computers can have 8 bytes. Or, a 64bit computer running a 32bit OS will have 4 bytes. Still, under a specific architecture, all types of pointers (void*, int*, char*, long* etc) will have same size (except function pointers).
That's, pointers in C (or C++) doesn't have a fixed size.


### 算术运算
C的算术运算只有两种形式。第一种是指针加减一个整数，第二种是两个指针相减。
#### 指针加减一个整数
指针加减一个整数的运算形式只能用于指向数组中某个元素的指针，将它加减一个整数得到的表达式也还是一个指针，它仍指向数组中某个元素。如果对指针进行加法或者减法运算之后，指针所指的位置是在数组第一个元素前面或者在最后一个元素后面，它的效果是未定义的。这种操作编译器不会进行检查，需要程序员自己进行检查。
这种形式也适用于`malloc`函数动态分配的内存。

#### 指针减指针
两个指针相减的结果类型是`ptrdiff_t`，是一种有符号整数类型。运算结果是两个指针在内存中的距离，以数组元素的长度为单位，而不是以字节为单位。两个指针必须指向同一个数组，结果可正可负。如果两个指针指向不同的数组，这个距离就没有意思。因为我们不知道两个数组分别存在哪个位置。

### 关系运算
`>`, `>=`, `<`, `<=`
关系运算也需要指针指向同一个数组中的元素。为了和C++的迭代器兼容，最好使用`==`或者`!=`，因为迭代器不支持关系运算，而指针和迭代器都支持`==`和`!=`运算。

## C中的指针表达式和左值右值[2]
### 指针自增自减操作
``` c
char ch = 'a';
char *cp = &ch;
```
1. 前置自增操作`++cp`，将`cp`的值加一，该操作先将`cp`的值加一，指向`ch`后面的一个位置，然后返回`cp`的一个拷贝。表达式`++cp`和`cp`加一后的对象一样。
2. 后置自增操作`cp++`，将`cp`的值加一，该操作先返回对象`cp`的一个拷贝，然后将`cp`的值加一，指向`ch`后面的一个位置。表达式`cp++`和`cp`加一前的对象一样。
3. 解引用前置自增操作`*++cp`，这个式子其实是对表达式`++cp`的解引用操作，也就是对`cp`加一后的拷贝的解引用操作，而不是对`cp`的操作。
4. 解引用前置自增操作`*cp++`，这个式子其实是对表达式`cp++`的解引用操作，是对`cp`加一前的拷贝的解引用操作，而不是对`cp`的操作。

### 指针表达式和左值右值
关于左值和右值的介绍，可以查看[C C++ lvalue and rvalue](http://localhost:4000/2019/11/14/C-CPP-lvalue-and-rvalue/)。
给出下列代码``` c
char ch[] = "abc";
char *cp = ch;

//&cp = 4; //这个是错的，&cp是一个地址，但是它本身只是一个数，它的本质和`10=4;`没有区别，使用解引用符号访问这个地址上的对象。
// 10 = 4; 错误，10既不是指针，也不是变量
//*10 = 4; 错误，10既不是指针，也不是变量
*(int*)10 = 4; //10是一个int，首先把它转化成一个指针，表示一个地址，然后使用解引用进行赋值
```
1. `ch`，作为左值时，表示的是`ch`在内存中的位置；`ch`作为右值时，值是`'a'`，
2. `&ch`，不能当左值，因为它没有存放在内存中；当右值时，值是变量`ch`的地址
3. `cp`，作为左值，是一个指针变量；作为右值，值是变量`ch`的地址
4. `&cp`，无法作为左值；作为右值，值是指针变量`cp`的地址
5. `*cp`，作为左值，和`ch`等价；作为右值，值是`'a'`
6. `*cp+1`，无法作为左值；作为右值，值是`'a'+1`。
7. `*(cp+1)`，作为左值，是一个指针，指向`ch`后面的一个内存单位；作为右值，是`ch`后面一个内存单位的值。
8. `++cp`，无法作为左值，右值和`*(cp+1)`一样。
9. `cp++`，无法作为左值，右值和`*(cp+1)`一样。
10. `*++cp`，作为左值，是一个指针，指向`ch`后面的一个位置；作为右值，是`ch`后面那个位置的值。
11. `*cp++`，作为左值，是一个指针，指向`ch`；作为右值，`'a'`。
12. `++*cp`，无法作为左值；作为右值，是`'b'`；
13. `(*cp)++`，无法作为左值；作为右值，是`'a'`。
14. `++*++cp`，无法作为左值；作为右值，是`'c'`。
15. `++*cp++`，无法作为左值；作为右值，是`'c'`

## 指针和数组
1. 指针和数组的联系很紧密，在很多用到数组名字的时候，编译器都会自动的将它转换成一个指向数组首元素的指针。数组名字是个常量指针。
2. 使用取地址符获取某个对象的指针，对数组元素使用取地址符就能得到指向该元素的指针。

关于更多指针和数组之间的内容，点击查看[数组](https://mxxhcm.github.io/2019/11/13/C-compound-type-array/)的介绍。

## 函数指针
函数指针指向的是函数而不是对象。和其他指针一样，函数指针指向某种特定类型。函数的类型由它的返回值和形参共同决定，和函数名无关。例如：``` c++
bool lengthCompare(const string &, const string &);
```
这个函数的类型是`bool(const string &, const string &)`，要想声明一个指向该函数的指针，只需要使用指针代替函数名字即可：``` c++
bool (*pf)(const string &, const string &);
```
从声明符中的变量名字开始，`pf`前面有个`*`，所以`pf`是个指针，右侧是形参列表，左侧是函数的返回值类型。因此，`pf`是一个指向函数的指针，函数的参数是两个`const string`的引用，返回值是`bool`类型，指针类型是`boo(*)(const string &, const string &)`

函数指针可以作为形参，也可以作为返回值，还可以使用`decltype`和`typedef`简化函数指针。需要注意的是，`decltype`不会把函数转换成指针。


## 参考文献
1.《C++ Primer第五版》
2.《C和指针》
3.https://stackoverflow.com/questions/54621080/lvalues-in-the-iso-c11-standard
4.http://www.open-std.org/jtc1/sc22/WG14/www/docs/n1570.pdf
5.https://www.quora.com/What-is-the-size-of-a-pointer-in-C`
