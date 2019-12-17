---
title: asymptotically efficient 渐进有效性
date: 2019-09-18 14:55:50
tags:
 - 概率论与统计
categories: 概率论与统计
mathjax: true
---

## 无偏估计的方差下界 cramer-rao bound
理论上可以证明，任何无偏估计的方差都有一个下界，这个下界叫做cramer-rao bound。具体的证明好复杂，这里只是简单说一下。如果证明算法无偏估计量的方差的下界是cramer-rao bound，说明这个算法的下界已经没有优化了。。。这个下界其实很多时候不知道什么时候能取到，到时能给我们一定的信息，就像期望一样。

它的最简单形式是：任何无偏估计的方差至少大于fisher information的倒数。

## Efficient
Efficient说的是在所有的无偏估计方法中，如果某种方法中无偏估计的方差等于cramer-rao bound，那么这个方法就是efficient。（应该是这样子吧。。。）

## Asymptotically 
在样本有效的情况下，统计量的方差不好计算。但是当样本不断增大时，方差会逐渐接近一个定值。用asymptotically修饰不断增大趋向于无穷的样本数量。

## Asymptotically  Efficient
Asymptotically efficient指得是某种方法在小样本时可能不是efficient的，但是随着样本数量不断增加，变成了efficient的，这种方式就是asymptotically efficient的。


## 参考文献
渐进有效性
1.https://www.zhihu.com/question/285834087/answer/446120288
2.https://bbs.pinggu.org/thread-2139008-1-1.html
3.https://www.zhihu.com/question/28908532/answer/254617423
4.https://en.wikipedia.org/wiki/Efficiency_(statistics)#Asymptotic_efficiency
5.https://cs.stackexchange.com/questions/69819/what-does-it-mean-by-saying-asymptotically-more-efficient
cramer-rao bound
6.https://www.zhihu.com/question/24710773/answer/117796031
7.http://www2.math.ou.edu/~kmartin/stats/cramer-rao.pdf
8.https://www.zhihu.com/question/311561435/answer/607730638
9.https://zhuanlan.zhihu.com/p/27644403
