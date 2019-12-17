---
title: conjugate gradient
date: 2019-09-23 14:49:28
tags:
 - 机器学习
categories: 机器学习
mathjax: true
---

## 简介
共轭梯度(conjugate method)方法是一种[迭代法](https://mxxhcm.github.io/2019/09/23/%E8%BF%AD%E4%BB%A3%E6%B3%95/)。
共轭梯度收敛的快慢取决于系数矩阵谱分解的情况。特征集集中，系数矩阵的条件数很小，收敛的就快。

## 核心
解线性对称正定方程组（$A$是对称正定矩阵）：
$$Ax=b$$
可以转化为
$$\min_x f(x) = \frac{1}{2} x^T Ax - b^T x$$
因为：
$$f'(x) = Ax-b$$


## 和其他迭代法的对比

## 参考文献
1.https://tangxman.github.io/2015/11/18/optimize-gradient/
2.https://www.zhihu.com/question/27157047
3.https://www.zhihu.com/question/27157047/answer/121950241
4.https://www.zhihu.com/question/27157047/answer/171336970
5.https://blog.csdn.net/lusongno1/article/details/78550803
6.https://medium.com/@jonathan_hui/rl-conjugate-gradient-5a644459137a
