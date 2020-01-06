---
title: C and C++ lvalue and rvalue
date: 2019-11-14 17:22:02
tags:
 - C/C++
categories: C/C++
---

## 对象(object)和值(value)
### C11标准
C11标准的定义如下([14]3.15, 3.19)：
> 3.15 object
region of data storage in the execution environment, the contents of which can represent values
NOTE When referenced, an object may be interpreted as having a particular type; see 6.3.2.1.
> 3.19 value
precise meaning of the contents of an object when interpreted as having a specific type

object是一块内存空间，它的内容可以表示值。当被使用时，一个对象可以解释为一种特定的类型。value是以具体类型解析object中的内容。

### C++11标准
C++11标准的定义如下([17]1.8)：
> An object is a region of storage. [ Note: A function is not an object, regardless of whether or not it occupies storage in the way that objects do. — end note ] 
> An object can have a name. 
> An object has a storage duration which influences its lifetime. 
> An object has a type. 
> The term object type refers to the type with which the object is created.

C++ 中的obejct也是一个内存空间。object可以有名字，可以有类型（内置类型还是复合类型都行），有一个duration。也就是C++ Primer第五版中说的对象是具有某种数据类型的内存空间，可以有名，可以没有名字。

## C语言中的lvalue和rvalue
### 左值和右值的定义
C中早期的定义：
左值是一个表达式，可以出现在赋值操作的左边或者右边，而右值只能出现在左边。
C11中左值的定义：
> An lvalue is an expression (with an object type other than void) that potentially designates an object;64) if an lvalue does not designate an object when it is evaluated, the behavior is undefined. When an object is said to have a particular type, the type is specified by the lvalue used to designate the object. A modifiable lvalue is an lvalue that does not have array type, does not have an incomplete type, does not have a constqualified type, and if it is a structure or union, does not have any member (including, recursively, any member or element of all contained aggregates or unions) with a constqualified type.

### 可修改左值和不可修改左值
1. 左值指向的位置的内容，如果可以被修改，那么这个左值是一个可修改左值，否则就是不可修改左值。
2. 如果一个表达式指向内存中的一个位置，并且它的类型是算术类型，`struct`，`union`或者pointer，那么它就是一个可修改左值。
3. 可修改左值不能含有数组类型，不完整的类型，`const`修饰的类型，它们都是不可修改左值。如果`struct`或者`union`要是可修改左值，那么它们不能有`const`成员。
4. 为什么可修改左值不能是`array`，我们不能对数组赋值，但是可以通过下标操作对数组进行赋值，所以数组名字不能是一个可修改的左值。或者说数组名字其实是数组首元素的地址。这里的数组赋值说的是：```c
int a[] = {1, 2, 3};
int b[] = {4, 5, 6};
b = a;  //想要做的操作是把数组a赋值给数组b，这是错误的。
```
5. 为什么没有说函数不能是可修改左值，函数名指定的是一个函数不是一个对象，所以函数名不能是左值，自然也不能是一个可修改左值。
6. 数组和指针都是左值，但是数组是不可修改左值，而指针是可修改左值（非const）。数组作为作为右值表示的是数组首元素的地址，数组作为左值，表示的是数组类型，是不能修改的左值。而指针变量可以作为左值，因为我们可以取得它的地址，指针变量作为右值是指针变量存储的值，即它指向变量的地址。

### 左值类型
C语言中的左值有以几种：
1. 任意类型变量的名字
2. 下标运算符`[]`
3. 指针的成员访问操作`->`和`.`
4. 单目运算符解引用`*`的表达式，不能指向一个数组
5. 指针的解引用操作，不能是一个函数指针
6. 数组，`const`对象，是一个不可修改左值，比如`const int a = 0;`，`a`是一个不可修改左值
7. 字符串字面值常量是一个不可修改左值[12]，因为C中没有字符串类型，字符串常量都是以字符数组类型存储的，而在C中，除了左值以外没有任何方式可以让数组存在于表达式中。

### 左值和右值的转换
#### 左值到右值的隐式类型转换[13]
1. C语言存在左值到右值的默认类型转换，当运算符需要右值操作对象时，而给出的是左值操作对象时，编译器会默认将左值转换成右值。
2. 数组到指针的转换。在需要右值操作对象的时候，编译器换将数组名转换为其首元素的地址，类型为指向元素的指针。
3. 函数到指针的转换。

一般情况下，对象之间的运算，对象是以右值的形式参与的。比如`+`运算符需要两个右值运算数：```c
int a = 1;
int b = 2;
int c = a + b;
```
在上面的例子中,`a`和`b`都是左值，在`int c = a+b;`中，它们经历了隐式的类型转换，将左值转换为了右值。
除了数组，函数，不完整的类型，所有的左值都可以转换为右值，但是右值不能转换为左值。

#### 右值产生左值
``` c
int a[] = {1, 2, 3};
int *p = &a[0]; //a和&a[0]都是数组首元素的值
*(p+1) = -1;    //p+1是右值，但是*(p+1)是左值
```

#### 左值产生右值
``` c
int var = 10;
int *p = &var;  //var是左值，但是&var是右值。
```
单目运算符`&`需要一个左值作为它的运算对象，当且仅当`n`是一个变量时，`&n`是一个有效的表达式，`&12`是错误的。

## C++中表达式的value category
C++ 中，一个表达式有两个基本属性，基本类型和值类别。在C++ 中有五种value category，它们的关系如下：
![value_category](value_category.png)
每一个表达式属于三种基本value中的一个：lvalue, xvalue和prvalue，表达式的这种属性叫做value category。


### `lvalue`
> An lvalue (so called, historically, because lvalues could appear on the left-hand side of an assignment expression) designates a function or an object. [Example: If E is an expression of pointer type, then *E is an lvalue expression referring to the object or function to which E points. As another example,the result of calling a function whose return type is an lvalue reference is an lvalue. — end example ]

左值指定了一个函数或者对象（变量）。它存放在内存中的某个位置，并且允许使用取值地址符`&`获取这块内存的地址。如果`E`是指针类型的表达式，那么`*E`是`E`指向的函数或者对象的左值表达式。左值分为可修改左值和不可修改左值，像常量，数组名，等属于不可修改左值，而其它的左值都是可修改左值。如果一个表达式不是左值，那么它就被定义为右值。
怎么样判断左值，满足以下两点中任何一点就是一个左值：
1. 是否有名字
2. 是否能够取到它的地址

C++ Primer中给出的一个方法：当一个对象被用作右值的时候，用的是对象的值。当一个对象被用作左值的时候，用的是对象在内存中的位置。
示例
```c
char ch = 'a';
char *cp = &ch; //ch可以当做左值，也可以当做右值
&ch = 3;  //错误，因为&ch我们只能取得它的值，并不能获取它在内存中的地址，即它只是一个右值，不能当做左值。
```
**对象（变量）和指针变量中存放的内容（即地址）的区别，对象可以直接进行赋值。指针变量中存放的是一个地址，地址本身就是一个数字，是一个右值，不能对其进行赋值，对这个地址进行解引用，得到指针指向对象的左值表达式。**

### `xvalue`
> An xvalue (an “eXpiring” value) also refers to an object, usually near the end of its lifetime (so that its resources may be moved, for example). An xvalue is the result of certain kinds of expressions involving rvalue references (8.3.2). [Example: The result of calling a function whose return type is an rvalue reference is an xvalue. — end example ]

`xvalue`也指向一个对象，通常在对象声明周期的最后。一个`xvalue`是和右值引用相关的特定表达式的结果。

### `prvalue`
> A prvalue (“pure” rvalue) is an rvalue that is not an xvalue. 

prvalue`是不是xvalue的rvalue。下列表达式是`prvalue`表达式：
- 一个字面值常量（除了字符串常量），比如`42`, `true`或者`nullptr`
- 返回值类型是非引用类型的函数调用或者重载的operator表达式的结果。
- 内置的后置自增自减运算符, `a++`, `a--`。
- 内置的算术表达式，`a+b`, `a&b`, `a<<b`。
- `a>b`, `a==b`, `a>=b`，内置的比较表达式
- `&a`,内置的取地址符
- `a.m`表达式的成员对象，其中`m`是一个member enumerator或者一个非静态的成员函数，或者[a是一个rvalue，m是一个非引用类型的非静态数据成员，until C++ 11]。

### glvalue
> A glvalue (“generalized” lvalue) is an lvalue or an xvalue.

一个glvalue是一个lvalue或者一个xvalue。下面的表达式是xvalue表达式：
- 返回值类型是引用类型的函数调用或者重载的operator表达式的结果。
- `a[n]`，内置的下标表达式，a是一个array rvalue。
- `a.m`，对象成员的表达式，其中a是一个rvalue，m是一个非引用类型的非静态数据成员。


### rvalue
> An rvalue (so called, historically, because rvalues could appear on the right-hand side of an assignment expression) is an xvalue, a temporary object (12.2) or subobject thereof, or a value that is not associated with an object.

一个右值可以是一个xvalue，一个临时对象，或者一个没有和对象关联的值。

## C++和C中lvalue的区别
《C++ Primer》中说C++ 和C中的左值和右值不一样，我怎么觉得都一样呢。（好吧，自己还是道行太浅了）。
举例来说：
1. 比如说`++i`和`--i`操作，在C中，它是一个右值，而在C++ 中，它是左值，而`i++`和`i--`在C和C++ 中都是右值[9]。
2. 在C语言中，三目运算符（?:）的结果一定是右值，而在C++中，如果:旁边的两个操作数是左值，那么结果也是左值[10]。


### C++中的左值运算
1. 赋值运算需要一个非常量左值作为左侧运算对象，得到的结果也仍然是一个左值。
2. 取地址符用作一个左值运算对象，返回一个指向该运算对象的地址，这个值是一个右值。
3. 内置解引用运算符，下标运算符，迭代器解引用运算符，`string`和`vector`的求值结果都是左值。
4. 内置类型和迭代器的递增递运算符作用于左值运算对象，其前置版本所得的结果也是左值，即`++iter`, `++i`等。


## 参考文献
1.《C和指针》
2.《C++ Primer》
3.https://www.geeksforgeeks.org/lvalue-and-rvalue-in-c-language/
4.https://segmentfault.com/a/1190000003793498
5.https://stackoverflow.com/questions/45656162/why-cant-a-modifiable-lvalue-have-an-array-type
6.https://www.quora.com/What-is-lvalue-and-rvalue-in-C
7.https://www.internalpointers.com/post/understanding-meaning-lvalues-and-rvalues-c
8.https://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c
9.https://www.zhihu.com/question/29936562/answer/46129706
10.https://www.zhihu.com/question/313519801/answer/642403872
11.https://www.zhihu.com/question/36052573/answer/65743965
12.https://stackoverflow.com/questions/10004511/why-are-string-literals-l-value-while-all-other-literals-are-r-value
13.https://www.zhihu.com/question/25814721/answer/31648501
14.https://stackoverflow.com/questions/3601602/what-are-rvalues-lvalues-xvalues-glvalues-and-prvalues
15.http://www.open-std.org/jtc1/sc22/WG14/www/docs/n1570.pdf
16.https://stackoverflow.com/questions/54621080/lvalues-in-the-iso-c11-standard
17.http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3242.pdf
18.https://en.cppreference.com/w/cpp/language/value_category