---
title: assamble MIPS
date: 2018-10-24 20:42:37
tags:
- 汇编语言
- MIPS
categories: 汇编
---

## 技巧
怎么写汇编程序。
写一个函数时，在被调用函数内，首先说明参数，表明每个寄存器存放的都是什么。比如a0-a3,都存放的是什么。在调用功能函数内，使用a0-a3传递参数。
先使用寄存器，最后在分配栈帧大小。。

s0-s7可以用来保存数据，比如函数A调用函数B，函数A中使用的s0-s7，在调用完函数B返回后，s0-s7原来的值是什么，现在就还是什么。
t0-t9就随用随操作。
在写循环的时候
先写循环的框架。。

## MIPS R2000 指令格式
每条指令都是32位长，由几个bits field构成，如下所示：
| | 6 bits|5 bits|5 bits|5 bits|5 bits|6 bits|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|寄存器|op|reg1|reg2|des|shift|funct|
|立即数|op|reg1|reg2|16位常数|
|jump|op|26位常数|

前六位是op域，确定这条指令是寄存器指令，立即数指令还是jump指令，从而确定后面的指令该如何被解释。
如果op域是0，表示这是一条寄存器指令，通常执行一个算术或者逻辑运算。funct域指定要执行的操作，reg1和reg2表示用作操作数的寄存器，des表示存储结果的寄存器。

## MIPS Register集合
MISP R200 CPU总共有32个寄存器，其中31个都是通用寄存器，可以用来存储任何指令，剩下的一个，叫做register zero，任何时候存放的总是0。
尽管理论上任意一个register都可以用作任何目的。MIPS程序员通常有一个默认的规定：
|symbolic name| 序号| 用途
|:--:|:--:|:--:|
|zero| 0| 一直存放的都是常数0。|
|at| 1|保留|
|v0-v1| 2-3|包含返回值，如果值是单字的，只有$v0有用|
|a0-a3| 4-7| 参数寄存器，包含一个子程序调用的前4个参数|
|t0-t9|8-15, 24-25|临时寄存器0到9|
|s0-s7| 16-23|保存寄存器0到7|
|k0-k1| 26-27|内核寄存器0,1，DO NOT USE|
|gp|28|全局数据指针用于定位static global varaibles。现在忽略它|
|sp| 29|栈指针|
|fp|30|帧指针|
|ra|31|返回地址|

## SPIM环境系统调用
|service | code |参数|结果|
|打印int| 1| $a0|在console打印出int|
|打印float| 2| $f12|在console打印出float|
|打印double| 3| $f12|在console打印出double|
|打印string| 4| $a0|在console打印出string|
|读入int| 5|none|结果存放在v0|
|读入float| 6|none|结果存放在f0|
|读入double| 7|none|结果存放在f0|
|读入string| 8|$a0存放地址，$a1存放长度|空|
|sbrk|9| $a0(长度)|$v0|
|exit|10|空|空|


## Stack Frames
每一次调用一个子程序，就会专门为当前子程序调用创建一个独一无二的栈帧。在递归调用的情况下，创建了很多个栈帧，每一次调用就有一个。栈帧的组织形式是很重要的，因为它们需要在调用者和被调者之间传递参数，定义寄存器如何在调用函数和被调用函数之间共享。此外，还需要在被调用函数的栈帧内定义局部变量的存储格式。每个栈帧的最小值是32。
一般来说，一个子程序调用的栈帧可能包含以下的内容：
- 传递给子程序的参数
- saved寄存器的值（$s0到$s7）
- 子程序调用返回地址（$ra）
- 局部变量

如下图所示，栈从高地址向低地址增长。
![]()
每个stack frame被分成了五个区域：
1. 参数区域，存放子程序调用的其他子程序所需要的参数。它们不会被当前子程序使用，而是用于当前子程序调用的子程序。前四个参数可以使用参数寄存器（$a0到$a3）进行传递，如果超过四个参数，就存放在sp+16, sp+20, 等等。也就是说当前栈帧最底部存放的是它调用的子程序所需要的参数，如果它没有调用子程序，那就不需要这个区域了。
2. 保留寄存器区域用于保存saved寄存器（$s0到$s7）的值。这些寄存器的值在进入当前子程序时就被保存，在子程序返回前，从这个区域将值复制到saved寄存器。在这期间，saved寄存器的值可以随意使用。这个很有用哦。
3. 返回地址区域用于存放返回地址寄存器（$ra）的值，这个值在当前子程序被执行的时候，就被复制到栈上，在当前子程序返回前被拷贝回去。如果当前子程序没有调用其他子程序，不需要这个区了。
4. Pad区域是保证栈帧的总大小是8的倍数。在这里插入是为了确保，局部变量的存储区域总是以双字开始的。
5. 局部存储区域用于局部变量的存储。当前子程序必须保留足够的字存放局部变量和局部寄存器（$t0到$t9）的值。这个区域也必须进行padding确保它的大小总是8 words的整数倍。

总结以下，当一个函数被调用时，为它创建一个栈帧，这个栈帧的最小值是32，最大值不限，要能容纳所有要保存的对象，包括上面介绍的五个部分。具体怎么操作呢。如果当前函数被调用了，首先移动`sp`到能容纳下所有要保存的内容，然后保存返回地址，保存可能使用到的saved寄存器。然后将传递给这个子程序的参数，$a0到$a3这几个参数的值存放在**调用当前函数的函数的栈帧底部**，然后当前函数用到这几个值时，从那里获取，超过4个参数的通过栈传递，直接存放在sp+16, sp+20等位置。
比如当前函数调用了两次子函数，需要用到传递给当前函数的这几个参数，在这几个子函数调用前后参数值不应该改变。

## MIPS Insturction Set
- 如果op包含 a(u)，这条指令是有符号数或者无符号数参与的算术运算，取决于有没有u。
- des 一定是一个寄存器
- src1 一定是一个寄存器
- reg2 一定是一个寄存器
- src2 可以是一个寄存器，也可以是一个32位整数
- addr一定是一个地址。

### 区别
``` assembly
    .data
array:  .space 1024*4
    .text
la  $t1, array  # la将数组的地址存放到一个寄存器
lw  $a0, ($t1)  # lw将数组地址的内容load出来
lw  $a0, 24($t1)

move $t2, $t1
```

### 算术指令
- mul des, src1, src2   将寄存器src1和寄存器或者数值src2的结果存放在寄存器des
- mult(u) src1, reg2    将寄存器src1和寄存器reg2的结果存放在lo和寄存器hi上。
- div src1, reg2
- div des, sr1, src2

### 比较指令

### 分支和跳转指令

#### 分支(branch)
- beq ssr1, src2, lab

- ble sr1, src2, lab    如果src1 <= src2，跳转到lab
- bgt sr1, src2, lab    如果src1 >= src1，跳转到lab

和零比
- beqz src1, lab    如果寄存器src1的值等于0，跳转到lab
- bnez src1, lab    如果寄存器src1的值不等于0，跳转到lab
- blez src1, lab    如果寄存器src1的值小于等于0，跳转到lab
- bgez src1, lab    如果寄存器src1的值大于等于0，跳转到lab


#### 跳转
- j label   跳转到一个label
- jr src1   跳转到一个位置src1
- jar label 跳转到label，并且存储下一条指令到$ra
- jalr src1 跳转一个位置src1，并且将下一条指令的位置存储到$ra

### Load,Store和数据移动

#### Load
- la des, addr  加载一个label的地址到des
- lw des, addr 加载地址addr处的一个word
- li des, const 加载一个而常数到des

#### Store

#### 数据移动
- move des, src1  将寄存器src1的内容复制到寄存器des中
- mfhi des      将hi中的内容复制到des中
- mflo des      将lo中的内容复制到des中
- mthi src1     将寄存器src1中的内容复制到hi中
- mtlo src1     将寄存器src1中的内容复制到lo中

### 异常处理

## SPIM Assembler
### 段和链接器指令

### 数据指令
.asciiz str，汇编给定的以null结尾的字符串，
.ascii str，汇编给定的不以null结尾的字符串
.byte byte1 ... byteN，汇编给定的字节（八位整数）
.half half1 ... halfN，汇编给定的半字（十六位整数）
.word word1 ... wordN，汇编给定的字（三十二位整数）
.space size，在当前的段中分配n字节的空间，在SPIM中，只允许在数据段中进行。


## 参考文献
1.https://sites.cs.ucsb.edu/~franklin/64/lectures/mipsassemblytutorial.pdf
2.https://courses.cs.washington.edu/courses/cse410/09sp/examples/MIPSCallingConventionsSummary.pdf
3.http://www.mrc.uidaho.edu/mrc/people/jff/digital/MIPSir.html
4.https://stackoverflow.com/questions/7748769/mips-programming-how-to-call-a-function-from-a-separate-file
