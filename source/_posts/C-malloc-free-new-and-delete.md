---
title: C/C++ malloc(alloc) free new and delete
date: 2019-11-05 10:37:48
tags:
 - C/C++
categories: C/C++
---

## `malloc`
### C标准`malloc`定义
`malloc`定义在头文件`<stdlib.h>`中
> Allocates size bytes of uninitialized storage.
> If allocation succeeds, returns a pointer to the lowest (first) byte in the allocated memory block that is suitably aligned for any object type with fundamental alignment.
If size is zero, the behavior is implementation defined (null pointer may be returned, or some non-null pointer may be returned that may not be used to access storage, but has to be passed to free).

> malloc is thread-safe: it behaves as though only accessing the memory locations visible through its argument, and not any static storage.
A previous call to free or realloc that deallocates a region of memory synchronizes-with a call to malloc that allocates the same or a part of the same region of memory. This synchronization occurs after any access to the memory by the deallocating function and before any access to the memory by malloc. There is a single total order of all allocation and deallocation functions operating on each particular region of memory. (since C11)

### `malloc`, `calloc`和`realloc`原型
``` c
#include <stdlib.h>

void *malloc(size_t size);
void *calloc(size_t nmemb, size_t size);
void *realloc(void *ptr, size_t size);
void *reallocarray(void *ptr, size_t nmemb, size_t size);
```

### `malloc`, `calloc`和`realloc`性质
0. `malloc`函数的实现中（C标准没有规定），在分配空间的时候，通常分配的空间要比申请的要大一些，这些额外的空间用来记录`malloc`这片空间的大小，在使用`free`的时候会用到。
1. `malloc`，分配指定字节的内存空间，初始值不定。
2. `calloc`，为指定长度的固定数量的对象分配空间，每一个bit都被初始化为0。
3. `realloc`，增加或者减少已经分配的内存空间的大小。当这个大小增加时，可能需要将之前分配的空间中的数据移到另一个足够大的区域以便于增加大小，新增加的区域内的值是不确定的。
4. 这三个函数返回的指针一定是对齐的，保证它可以用于任何对象。比如`double`的要求最严格，需要从8的倍数的地址单元开始，这三个函数返回的地址一定满足这个要求。
5. 它们的返回类型都是`void*`，需要使用强制类型转换。
6. `realloc`函数可以增加或者减少之前分配的内存空间的大小。比如分配了一个固定大小的数组，后来发小它不够用了，可以使用`realloc`对它进行扩充，如果原有的存储后有足够的大小进行扩充，则可以在原存储区的位置上向高地址进行扩充，无需移动原有数组，返回和传入相同的指针。如果原来的内存空间后没有足够的空间，就重新分配一个足够大的内存空间，再将原有数据的内容复制过去，然后释放原来的内存空间，返回新的指针。
7. `realloc`传入的参数是存储区的新长度。如果传入的`ptr`参数是`NULL`指针，那就退化成了`malloc`。


### 自己实现一个`malloc`
???


## `free`
### C标准`free`定义
`free`定义在头文件`<stdlib.h>`中
> Deallocates the space previously allocated by malloc(), calloc(), aligned_alloc, (since C11) or realloc().
If ptr is a null pointer, the function does nothing.
The behavior is undefined if the value of ptr does not equal a value returned earlier by malloc(), calloc(), realloc(), or aligned_alloc() (since C11).
The behavior is undefined if the memory area referred to by ptr has already been deallocated, that is, free() or realloc() has already been called with ptr as the argument and no calls to malloc(), calloc() or realloc() resulted in a pointer equal to ptr afterwards.
The behavior is undefined if after free() returns, an access is made through the pointer ptr (unless another allocation function happened to result in a pointer value equal to ptr)
free is thread-safe: it behaves as though only accessing the memory locations visible through its argument, and not any static storage.

> A call to free that deallocates a region of memory synchronizes-with a call to any subsequent allocation function that allocates the same or a part of the same region of memory. This synchronization occurs after any access to the memory by the deallocating function and before any access to the memory by the allocation function. There is a single total order of all allocation and deallocation functions operating on each particular region of memory. (since C11)

### `free`原型
```
#include <stdlib.h>

void free(void *ptr);
```

### `free`属性
从标准的定义可以看出来，以下都是未定义的行为：
1. `free`的对象不是`alloc`函数族的返回值；比如```c
int *pi = (int*)malloc(10 * sizeof(int));
pi++;
//下面就是错误的，因为这个`pi`不是`alloc`函数族的返回值。
free(pi);
```
2. `free`一个已经被释放过了的块；
3. 访问一个`free`已经释放了的块。
等等。为什么？？？因为标准并没有定义`malloc`应该怎么实现，有的内存分配器，`malloc`实际申请的内存要比传入的参数大，里面存放了额外的数据记录这块内存有多大，一般就是存在指针左边。`free`的时候，就会读取那个内存块中存放的信息，进行`free`，所以上面的那些都是未定义的行为。

其他属性
1. `free`可以释放`ptr`指向的内存空间，释放的空间通常送入可用内存池，之后可以通过这三个函数重新分配。
2. `malloc`和`free`底层通常使用`sbrk`系统调用实现，这个系统调用扩充或者减小进程的堆，虽然`sbrk`可以扩充或者缩小进程的堆，但是一般`malloc`和`free`的实现不会减少进程的内存空间，释放的内存空间保存在`malloc`池中，而不是交给内核。
3. 大多数实现分配的空间要比请求的空间大一些，因为需要存储一些管理信息，如block的大小，指向下一个block的指针等等。因此，如果对超过一个分配区域的内存进行读写的话，会造成很严重的错误。
4. `free`一个已经释放了的块，`free`的不是`alloc`函数的返回值，没有进行`free`等等，都是未定义的结果。为什么？？？

## `new`
对于自定义类型而言，`new`操作符首先调用operator new()函数申请内存，其内部调用的是malloc函数，返回一个`void*`类型的指针；`new`还会负责把它转换成自定义对象的指针；然后调用类的构造函数初始化对象；最后返回自定义对象的指针。`malloc`只负责内存的分配而不会调用类的构造函数数。举个例子：``` c++
Complex *pc = new Complex(1, 2);
//等价于
// 1.内部调用malloc
void *mem = operator new(sizeof(Complex));
// 2.将void *类型指针转换为Complex*类型的
pc = static_cast<Complex*> (mem);
// 3.调用Complex的构造函数
pc->Complex::Complex(1, 2);
```

1. 默认初始化。`new`后面加类型，没有小括号，也没有花括号。
默认情况下，new分配的对象，不管是单个分配的还是数组中的，都是默认初始化的。这意味着内置类型或组合类型的对象的值是无定义的，而类类型对象将用默认构造函数进行初始化。
2. 值初始化。类型名字后加()即可，对于内置类型的变量，初始化为0，对于类类型的变量，调用默认构造函数。
3. 直接初始化。使用初始化列表加对象值，或者小括号加对象值。

对于自定义类型而言，只要一调用new，无论后面有没有加()，那么编译器不仅仅给它分配内存，还调用它的默认构造函数初始化。

## `delete`
1. `delete`是和`new`配对的操作。调用`delete`操作，编译器实际上会将它转换为两步操作，第一步是调用析构函数；第二部调用`opereator delete()`，其内部调用了`free`。
2. 数组的`new`一定要和数组的`delete`配对，否则就会出现内存泄露。第一条中说了`delete`分为两步，第一步其实不会造成内存泄露；而是第二步中，析构函数被调用的次数，`delete p`只会调用一次析构函数，而`delete[] p`会调用多次析构函数，调用次数和`new`时申请的数组大小一样。

## `malloc` vs `new`
1. `malloc`是C语言中的函数，而`new`是C++的操作符；
2. `malloc`返回的是`void*`类型的指针，需要我们手动进行强制类型转换转换成我们需要的类型，而`new`返回的是对象类型的指针，类型和对象严格匹配，`new`是类型安全型操作符。
3. 在分配内存失败时，`malloc`会返回NULL，而`new`会throw on failure。
4. `malloc`需要指定申请的内存占多少个字节，而`new`不需要指定申请内存块的大小，编译器会根据类型计算需要的内存大小；
5. `malloc`和`new`都是申请heap上的内存；
6. `new`操作符调用的operator new()函数可以重载（操作符`new`不能重载），而`malloc`不能重载。
7. `malloc`和`free`，`new`和`delete`必须配套使用。

几个问题：
1. 什么时候`malloc`和`new`会申请内存失败。
2. `new`操作符的两个步骤，一个是申请内存，一个是调用构造函数，`new`的申请内存和`malloc`的申请内存有什么区别。


## 参考文献
1.《C++ Primer第五版》
2.https://stackoverflow.com/questions/184537/in-what-cases-do-i-use-malloc-and-or-new
3.https://zhuanlan.zhihu.com/p/47089696?utm_source=wechat_session&utm_medium=social&utm_oi=687606928481730560
