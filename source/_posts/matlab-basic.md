---
title: matlab basic
date: 2019-10-26 16:16:58
tags:
 - 其他编程语言
categories: 其他编程语言
---

## Matlab基础

### 矩阵大小
``` matlab
x = ones(20, 3);
size(x);
```

### 矩阵减去向量

### 矩阵列拼接用,
``` matlab
x1 = ones(200, 1);
x2 = ones(200, 1);
x3 = ones(200, 1);
x4 = ones(200, 1);
x5 = ones(200, 3);
x = [x1(:, 1), x2(:, 1), x5(:, 2:3)]
```

### 矩阵行拼接用;
``` matlab
x1 = ones(1, 200);
x2 = ones(1, 200);
x3 = ones(1, 200);
x4 = ones(1, 200);
x5 = ones(3, 200);
x = [x1(1, :), x2(1, :), x5(2:3, :)]
```


### 指数运算
``` matlab
x = ones(10, 1);
x2 = x.^2;
```

## Matlab进行回归分析
- 多元线性回归
- 多项式回归
- 非线性回归
- 逐步回归

### 多元线性回归
#### regress介绍
使用regress
```
[b, bint, r, rint, stats] = regress(Y, X, alpha)
```
需要注意的是，X中还需要加上一列ones表示截距，否则会计算出错。
其中
b表示回归系数估计值，
bint表示回归系数的置信区间，
r表示残差，
rint表示残差的置信区间。
stats返回的是统计量，前三个值分别是$R^2 $，F，以及F对应的p

#### 示例
``` matlab
% 自变量
x=[143 145 146 147 149 150 153 154 155 156 157 158 159 160 162 164];
% 表示截距的ones
X=[ones(16,1) x];
% 因变量
Y=[88 85 88 91 92 93 93 95 96 98 97 96 98 99 100 102]';
% 回归分析
[b,bint,r,rint,stats]=regress(Y,X)
```

### 多项式回归
#### 一元多项式
使用polyfit或者polytool

#### 多元二项式
使用rstool或者rsmdemo

### 非线性回归
使用nlinfit

### 逐步回归
stepwise

## 标准差怎么计算
得到了参差平方和之后，可以计算标准差。
$$sd = \sqrt{\frac{R^2}{n-k} }$$
其中$n$是样本说，$k$是参数个数。

## 参考文献
1.https://www.cnblogs.com/hezhiyao/p/6824083.html
2.http://mcm.dept.ccut.edu.cn/u_newsfiles/1283049677/20120508/20120508204298149814.pdf
2.http://blog.sina.com.cn/s/blog_03f96e310106lktb.html
3.https://bbs.pinggu.org/thread-3065519-1-1.html
5.https://zhidao.baidu.com/question/1819455048729653268.html
