---
title: knn
date: 2018-1-20 23:08:49
tags:
 - 机器学习
categories: 机器学习
mathajax: true
---

## KNN
KNN的思路很简单，从训练集中（带标签）找到$k$个离待预测点最近的点，选出这$k$个最近点中最多的标签当做待遇测点的标签。
具体的思路：
1. 计算待预测点距离训练集中所有点的距离
2. 选择最近的$k$个训练集中的点
3. 选择$k$个点中最多的label
4. 这个label就是待遇测点的label

## 属性
- 基于实例：必须基于具体的样本才能进行预测，因为整个训练集（带标签）都属于KNN的一部分。
- 竞争学习：训练集中的每个元素都竞争预测结果。
- 懒惰学习：不需要训练，直到需要预测的时候才会建立模型。

## KNN classification
$$\text{Pr}(Y=j|X=X_0) = \frac{1}{K} \sum\_{x_i\in N_0}I(y_i=j)


## KNN regression
$$\hat{f}(x_0) = \frac{1}{K} \sum\_{x_i\in N_0} y_i$$

## 参考文献
1.https://zhuanlan.zhihu.com/p/36549000