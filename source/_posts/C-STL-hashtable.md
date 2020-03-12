---
title: C STL hashtable
date: 2020-03-10 21:22:47
tags:
 - C/C++
categories: C/C++
---

## hash
什么是hash，给定不定长的输入，得到定长的输出。
哈希的实现方式：数组，链表等。
hash可以用来干嘛？
1. 用时间换空间，实现常数时间的查找。主要是利用哈希表数据结构。
2. 保存密码。
3. 服务器扩容（一致性哈希）。

碰撞：给定不同的输入，得到相同的输入。这是一定会发生的，因为输入的范围是无穷大的。

解决碰撞的方法：
线性探测法：
二次散列法：
开链法：SGI STL就采用这种方法。

## hashtable
SGI STL中，通过维护一个链表的数组。每个链表叫做一个bucket，通过KEY对buckets取余计算bucket_number。所以不同的KEY可能会被映射到同一个bucket中，这就是冲突。
为了避免冲突太严重，STL中采用了一个方法，当元素的数量大于bucket的数量时，就要进行rehash。插入一个对象时，先判断是否需要rehash。如果需要rehash，需要对元素中的所有元素重新进行hash。
count怎么实现？find怎么实现？比如找某一个val，先找到val对应的key，找到相应bucket中的序号，判断这个bucket中的元素的key是否和要找的key相同。

hashtable的迭代器是ForwardIterator。


## 一致性hash
一致性hash经常用在服务器增加或者减少时，服务器失效的问题。为什么要取2的32次方？我的理解是，这样子的话，对于同一个数据key，不管有服务器的个数是多少，得到的数据的hash值都是一样的。所以增加服务器不影响数据的hash值。

## 参考文献
1.《STL源码剖析》
2.https://zhuanlan.zhihu.com/p/34985026
