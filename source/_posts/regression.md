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

### 系数准确性
#### 标准差
$$SE(\bar{\mu}) = \sqrt{var(\bar{\mu})}$$

#### 置信区间

#### 假设检验
零假设(hull hypothesis)：
$H_0:X$和$Y$之间没有关系，
备择假设(alternative hypothesis)：
$H_a:X$和$Y$之间有一定关系，
数学上，就相当于检验：
$H_0:\beta_1 = 0$和$H_a:\beta_1 \neq 0$
如果$\beta_1 = 0$，则$Y=\beta_0 + \epsilon$，即说明$X,Y$不相关。怎么判断$\beta_1$是否为$0$。这取决于$\beta_1$的准确性，和$\text{SE}(\hat{\beta_1})$相关。在实践中，计算$t$统计量：
$$t=\frac{\hat{\beta_1} -0}{\text{SE}(\hat{\beta_1})$$
假设$\beta_1=0$，观测任意观测值大于$\vert t\vert$的概率就行了，这个值称为$p$值。一个很小的$p$值表示，自变量和因变量之间的真实关系未知时，不太可能完全由于偶然观察到自变量和因变量之间的强相关。$p$值足够小，就决绝零假设。
所以方法是：
1. 计算$t$统计量
2. 计算$p$值
3. p很小，决绝零假设，接收备择假设。

### 模型准确性
如果拒绝零假设，选择备择假设，想要量化模型拟合的程度。给出两个指标：
#### Residual standard error (RSE)
RSE定义如下：
$$ \text{RSE} = \sqrt{\frac{1}{n-2} RSS} = \sqrt{\frac{1}{n-2} \sum\_{i=1}^n(y_i-\hat{y_i})^2 }
其中$\text{RSS} =\sum\_{i=1}^n(y_i-\hat{y_i})^2 $。 

#### $R^2 $
$R^2$的取值在$0$和$1$之间，越接近于$1$越好。
$$R^2 = \frac{\text{TSS}-\text{RSS}}{\text{TSS}} = 1- \frac{\text{RSS}}{\text{RSS}}$$
其中$\text{TSS} = \sum(y_i - \bar{y})^2$是 total sum of squares。
### 相关性


## 多元线性回归
假设有$p$个自变量，一个因变量$Y$，则假设多元线性回归方程为：
$$Y = \beta_0 + \beta_1 X_1+ \beta_2 X_2 + \cdots+ \beta_p X_p + \epsilon  \tag{1}$$
其中$X_j$表示第$j$个自变量，$\beta_j$表示第$j$个自变量和因变量之间的关系，$\beta_j$可以解释为，所有其他自变量不变的情况下，$X_j$增加一个单位，对$Y$产生的平均效果。


### 计算回归系数
有时候多元线性回归和简单线性回归可能会得到相反的结果，即某些变量的取值，在一个中为$0$，在另一个中不为$0$。但是实际上，它们表述的东西是一样的，多元线性回归中的某两个变量可能和简单线性回归中的一个变量起到了相同的作用，这样子多元线性回归中这两个变量可能一个起作用，一个不起作用。

### 分析
#### 假设检验
对于多元线性回归来说，我们需要判定是否所有的回归系数都为$0$，同样使用假设检验，
零假设：
$$H_0: \beta_1 = \beta_2 = \cdots \beta_p  =0$$
备择假设：
$H_a:$至少有一个$\beta_j$不为$0$。

#### F统计量
$$F=\frac{(TSS-RSS)/p}{RSS/(n-p-1)}$$
其中$\text{TSS} = \sum(y_i-\bar{y})^2 , \text{RSS}= \sum(y_i - \bar{y_i})^2$。
如果假设是正确的，有：
$$\mathbb{E}\left[\text{RSS}/(n-p-1)\right] = \sigma^2$$
且
$$\mathbb{E}\left[(\text{TSS} - \text{RSS})/p\right] = \sigma^2$$
因此，如果零假设正确，有$F$接近于$1$；否则，$F \gt 1$。

#### 重要变量选择

### 模型拟合
#### RSE
$$ \text{RSE} = \sqrt{\frac{1}{n-p-1}\text{RSS}}$$

## 其他注意事项
### 定性自变量
#### 二值自变量
假设我们考虑女性和男性的信用卡债务差异，只考虑性别原因，忽略所有其他的因素，可以根据性别创建一个指标，或者叫做哑变量(dummy variable)：
$x_i = 1$表示女性，$x_i=0$表示男性。
在回归方程中使用这个变量，得到：
$$y_i = \beta_0 + \beta_1 x_i + \epsilon_i = \begin{cases}\beta_0+\beta_1 + \epsilon_i, 女性\\\\\beta_0 + \epsilon_i,男性 \end{cases}$$
其中$beta_0$可能看成男性的信用卡债务，$\beta_0+\beta_1$是女性的平均信用卡债务，$\beta_1$就是男性和女性的差别了。
也可以用$1,-1$代替$0,1$。

#### 多值定性自变量
比如种族，用两个哑变量：
$x\_{i1}= \begin{cases}1,Asian\\\\ 0, non-Asian\end{cases}$
第二个哑变量：
$x\_{i1}= \begin{cases}1,white\\\\ 0, non-white\end{cases}$
得到模型：
$y_i = \beta_0 + \beta_1 x\_{i1} + \beta_2 x\_{i2} + \epsilon_i$= \begin{cases}\beta_0 + \beta_1 +\epsilon_i \\\\ \beta_0 + \beta_2+\epsilon_2 \\\\\beta_0+\epsilon_i\end{cases}$

#### 线性模型的扩展
##### 去除可加性
线性回归模型假设所有的自变量和因变量之间的关系是可加和线性的。但是，可能两个自变量之间会有一定的作用，考虑两个变量的标准线性回归模型：
$$ Y = \beta_0 + \beta X_1 + \beta X_2 + \epsilon$$
根据这个模型，改变$X_1$，$X_2$的存在不影响结果，可以对该模型做一个扩展，加一个有$X_1,X_2$的乘积组成的交互项：
$$Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_3 X_1X_2 + \epsilon$$
如果$X_1,X_2$之间的交互作用很重要，那么即使$X_1,X_2$各自系数的$p$值很大，这两个变量也应该在模型中。

##### 去除非线性
多项式拟合

### 可能的问题
#### 非线性拟合
绘制参差图，如果参差有规律，说明不适合当前的拟合方法。

#### 误差项自相关

#### 误差项方差不恒定

#### 外点
误差特别大，真实值$y_i$异常。

#### 高杠杆点
观测点$x_i$的值是异常的，就是$x_i$的取值和绝大部分$x_i$的取值不同。
计算杠杆统计量(leverage statistic)：
$$h_i = \frac{1}{n} + \frac{(x_i-\bar{x})^2 }{ \sum\_{j=1}^n (x_j - \bar{x})^2 }$$
$h_i$的取值总是在$\frac{1}{n}$和$1$之间，求所有观测的平均杠杆值是$\frac{p+1}{n}$，如果给定观测的杠杆量大大超过$\frac{p+1}{n}$，那么该点有较高的杠杆作用。

#### 共线性
共线性指的是两个或者更多的自变量高度相关。
检测共线性的一个方法是看预测变量的相关系数矩阵。但是这种方法只能检测出两个变量之间的共线性。
可能有三个或者更多自变量之间的共线性，叫多重共线性。可以计算方差膨胀因子variance inflation factor(VIF)，最小值是$1$，表示完全不存在共线性。一般来说，VIF超过$5$或者$10$就表示有共线性问题。
$$ VIF(\hat{\beta_j}) = \frac{1}{1-R^2\_{X_j|X\_{-j} } }$$
其中$R^2\_{X_j|X\_{-j} }$表示对$X_j$对所有自变量回归的$R^2$，如果它接近于$1$,就存在共线性。

## KNN regression vs linear regression
真实情况是线性情况下，KNN可能会略逊于linear regression。
在真实情况是非线性情况下，当变量个数$p$取$1$或者$2$时，KNN优于linear regression。取$3$时，不确定。当$p\ge 4$时，linear regression优于KNN。更高的话会有curse of dimension。
如果变量个数很少，样本数很多，多项式拟合可能会拟合到误差。
当$K=1,2$时，


## 参考文献
1.
