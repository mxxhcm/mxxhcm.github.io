---
title: regression
date: 2018-10-21 18:47:01
tags:
 - 机器学习
 - regression
categories: 机器学习
---

## linear regression
这一部分主要介绍线性回归。

## simple linear regression
第一节介绍两个变量的linear regression，也叫simple linear regression，假设$X$和$Y$之间存在线性关系。用
$$Y = \beta_0 + \beta_1 X \tag{1}$$
表示。然后根据我们已有的数据集划分训练集和测试集，使用训练集上的$n$组数据学习相应的参数，使得学习出的直接尽可能进行这$n$组数据点，即$y_i\approx \hat{\beta_0} + \hat{\beta_1}x_i$。

### 参差平方和
最小二乘法的目标是最小化残差平方和：
$$ \text{RSS} = e_1^2 + e_2^2 + \cdots e_n^2 \tag{2}$$
即
$$ \text{RSS} = (y_1-\hat{\beta_0}- \hat{\beta_1}x_1)^2 + (y_2-\hat{\beta_0}- \hat{\beta_1}x_2)^2 + \cdots (y_n-\hat{\beta_0}- \hat{\beta_1}x_n)^2 \tag{3}$$
最小二乘法通过最小化残差平方和，得到：
$$\hat{\beta_1} = \frac{\sum\_{i=1}^n (x_i-\bar{x}(y_i-\bar{y}}{\sum\_{i=1}^n(x_i-\bar{x})^2 } \tag{4}$$
$$\hat{\beta_0} = \bar{y} - \hat{\beta_1}\bar{x}\tag{5}$$

### 标准差
$$SE(\bar{\mu}) = \sqrt{var(\bar{\mu})}$$

### 置信区间

### 假设检验

### Residual standard error (RSE)
RSE定义如下：
$$ \text{RSE} = \sqrt{\frac{1}{n-2} RSS} = \sqrt{\frac{1}{n-2} \sum\_{i=1}^n(y_i-\hat{y_i})^2 }
其中$\text{RSS} =\sum\_{i=1}^n(y_i-\hat{y_i})^2 $。 

### $R^2 $
$$R^2 = \frac{\text{TSS}-\text{RSS}}{\text{TSS}} = 1- \frac{\text{RSS}}{\text{RSS}}$$



##

## 参考文献
1.
