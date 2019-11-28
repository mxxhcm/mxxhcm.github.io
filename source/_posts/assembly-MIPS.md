---
title: assamble MIPS
date: 2018-10-24 20:42:37
tags:
- 汇编语言
- MIPS
categories: 汇编
---

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
|v0-v1| 2-3|结果寄存器|
|a0-a3| 4-7| 参数寄存器1,2,3,4|
|t0-t9|8-15, 24-25|临时寄存器0到9|
|s0-s7| 16-23|保存寄存器0到7|
|k0-k1| 26-27|内核寄存器0,1|
|gp|28|全局数据指针|
|sp| 29|栈指针|
|fp|30|帧指针|
|ra|31|返回地址|


## MIPS Insturction Set
- 如果op包含 a(u)，这条指令是有符号数或者无符号数参与的算术运算，取决于有没有u。
- des 一定是一个寄存器
- src1 一定是一个寄存器
- reg2 一定是一个寄存器
- src2 可以是一个寄存器，也可以是一个32位整数
- addr一定是一个地址。


### 算术指令
- mul des, src1, src2   将寄存器src1和寄存器或者数值src2的结果存放在寄存器des
- mult(u) src1, reg2    将寄存器src1和寄存器reg2的结果存放在lo和寄存器hi上。

### 比较指令

### 分支和跳转指令

#### 分支(branch)
- beq ssr1, src2, lab

和零比
- beqz src1, lab    如果寄存器src1的值等于0，跳转到lab
- bnez src1, lab    如果寄存器src1的值不等于0，跳转到lab
- blez src1, lab    如果寄存器src1的值小于等于0，跳转到lab
- bgez src1, lab    如果寄存器src1的值大于等于0，跳转到lab


#### 跳转

### Load,Store和数据移动

#### Load
#### Store
#### 数据移动
- move des, src1  将寄存器src1的内容复制到寄存器des中
- mfhi des      将hi中的内容复制到des中
- mflo des      将lo中的内容复制到des中
- mthi src1     将寄存器src1中的内容复制到hi中
- mtlo src1     将寄存器src1中的内容复制到lo中

### 异常处理

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
2.http://www.mrc.uidaho.edu/mrc/people/jff/digital/MIPSir.html
