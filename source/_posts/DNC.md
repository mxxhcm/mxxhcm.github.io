---
title: DNC
date: 2019-05-21 09:07:41
tags:
- 机器学习
categories: 机器学习
mathjax: true
---

## 摘要
## 引言
现代计算机将memory和computation分开，使用处理器进行计算，处理器使用可访问的memory存取数。这样子的好处是可以使用extensible storage写入新信息，可以将memory中的内容当做variables。Variables对于算法的通用性很有用，对于不同的数据，不需要更改算法操作的地址，只需要更改变量的取值即可。而neural network的computation和memory是通过network的weights和neuron activity耦合在一起的。如果memory需要增加的话，networks不能动态增加新的storage，也不能独立的学习network的参数。

这篇文章中作者提出了differentiable neural computer(DNC)--带可读写external memory的network，解决network不能表示variable和数据结构的问题。整个system是可导的，可以把DNC的memory看做RAM，把network看做CPU。
DNC有一个$N\times W$大小的memory matrix $M$，使用可导的attention mechanism，确定在这个memory上的distributions，也就是我们说的weighting（加权）,代表相应的操作在该位置上的权重。DNC提供了三种操作，查询，读和写，对应了三种不同的attention，使用三个head（头）,read head（读头），write head（写头）,lookup head（查找）实现对memory的相应操作。

## 参考文献
1.https://www.gwern.net/docs/rl/2016-graves.pdf
2.http://people.idsia.ch/~rupesh/rnnsymposium2016/slides/graves.pdf
3.https://deepmind.com/blog/differentiable-neural-computers/
