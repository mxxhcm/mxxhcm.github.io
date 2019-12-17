---
title: convex optimization chapter 1 Introduction
date: 2018-12-22 13:44:11
tags: 
  - 凸优化 
categories: 凸优化
mathjax: true
---

## 数学优化(mathematical optimization)
### 定义
一个数学优化问题（或者称为优化问题）通常有如下的形式：
\begin{align\*}
&minimize \quad f_0(x)\\\\
&subject \ to \quad f_i(x) \le b_i, i = 1,\cdots,m.
\end{align\*}
其中$x = (x_1, \cdots, x_m)$被称为优化变量(optimization variables), 或者决策变量(decision variables)。 $f_0(x):\mathbb{R}^n \rightarrow \mathbb{R}$是目标函数(object function), $f_i(x):\mathbb{R}^n \rightarrow \mathbb{R},i =1,\cdots,m$是约束函数(constraint functions)。 常量(constraints) $b_1,\cdots,b_m$是约束的限界(limits)或者边界(bounds), $b_i$可以为$0$，这个可以通过移项构造出新的$f_i(x)$实现。如果向量$x$使得目标函数取得最小的值，并且满足所有的约束条件，那么这个向量被称为最优解$x^{\*} $。 

#### 线性优化(linear program)
目标函数和约束函数$f_0,\cdots,f_m$是线性的, 它们满足不等式： 
$$f_i(\alpha x+\beta y) = \alpha f_i(x) + \beta f_i(y)$$
对于所有的$x,y \in \mathbb{R}^n $和所有的$\alpha, \beta \in\mathbb{R}$。 
线性优化是凸优化的一个特殊形式, 它的目标函数和约束函数都是线性的等式。

#### 非线性问题(non-linear problem)
如果优化问题不是线性的，就是非线性问题。只要目标函数或者约束函数至少有一个不是线性的，那么这个优化问题就是非线性优化问题。

#### 凸问题(convex problem)
凸问题是目标函数和约束函数都是凸的的优化问题，它们满足：
$$f_i(\alpha x + \beta y) \le \alpha f_i(x) + \beta f_i(y)$$
对于所有的$x,y \in \mathbb{r}^n $和所有的$\alpha, \beta \in \mathbb{r}$且$\alpha + \beta = 1, \alpha \ge 0, \beta \ge 0$。
凸性比线性的范围更广，不等式取代了更加严格的等式，不等式只有在$\alpha$和$\beta$取一些特定值时才成立。凸优化和线性问题以及非线性问题都有交集，它是线性问题的超集(superset)，是非线性问题的子集(subset)。技术上来说，nonlinear problem包括convex optimization(除了linear programming), 可以用来描述不确定是非凸的问题。
Nonlinear program > convex problem > linear problem


### 示例
#### 组合优化(portfolio optimization)
变量：不同资产的投资数量
约束：预算，每个资产最大/最小投资数量，至少要得到的回报
目标：所有的风险，获利的变化
#### 电子设备的元件大小(device sizing in electronic circuits)
变量：元件的宽度和长度
约束：生产工艺的炼制，时间要求，面积等
目标：节约能耗
#### 数据拟合(data fitting)
变量：模型参数
约束：先验知识，参数约束条件
目标：错误率

### 优化问题求解(solving optimization problems)
所有的问题都是优化问题。
绝大部分优化问题我们解不出来。
#### 一般的优化问题(general optimization problem)
- 很难解出来。
- 做一些compromise，比如要很长时间才能解出来，或者并不总能找到解。

#### 一些例外(some exceptions)
+ 最小二乘问题(least-squares problems)
+ 线性规划问题(linear programming problems)
+ 凸优化问题(convex optimization problems)

## 最小二乘(least-squares)和线性规划(linear programming) 
least-squares和linear programming是凸优化问题中最有名的两个子问题。

### 最小二乘问题(least-squares problems)
最小二乘问题是一个无约束的优化问题，它的目标函数是项$a_i^T x-b_i$的平方和。
\begin{align\*}
minimize \quad f_0(x) &= {||Ax-b||}^2_2 \\\\
&=\sum_{i=1}^k (a_i^T x-b_i)^2
\end{align\*}

#### 求解(solving least-squares problems)
- 最小二乘问题的解可以转换为求线性方程组$(A^T A)x = A^T b$的解。线性代数上我们学过该方程组的解析解为$x=(A^T A)^{-1} A^T b$。
- 时间复杂度是$n^2 k = n\*k\*n+n\*k+n\*n\*n, (k > n)$(转置，求逆，矩阵乘法)。
- 该问题具有可靠且高效的求解算法。
- 是一个很成熟的算法

#### 应用(using least-squares)
很容易就可以看出来一个问题是最小二乘问题，我们只需要验证目标函数是不是二次函数，以及对应的二次型是不是正定的即可。

##### 加权最小二乘(weighted least-squares)
加权最小二乘形式如下:
$$\sum_{i=1}^k \omega_i(a_i^T x-b_i)^2 ,$$
其中$\omega_1,\cdots,\omega_k$是正的，被最小化。 这里选出权重$\omega$来体现不同项$a_i^T x-b_i$的比重, 或者仅仅用来影响结果。

##### 正则化(regularization)
目标函数中被加入了额外项, 形式如下：
$$\sum_{i=1}^k (a_i^T x-b_i)^2 + \rho \sum_{i=1}^n x_i^2 ,$$
正则项是用来惩罚大的$x$, 求出一个仅仅最小化第一个求和项的不出来的好结果。合理的选择参数$\rho$在原始的目标函数和正则化项之间做一个trade-off, 使得$\sum_{k=1}^i (a_i^T - b_i)^2 $和$\rho \sum_{k=1}^n  x_i^2 $都很小。
正则化项和加权最小二乘会在第六章中讲到，它们的统计解释在第七章给出。

### 线性规划(linear programming)
线性规划问题装目标函数和约束函数都是线性的：
\begin{align\*}
&minimize \quad c^T x\\\\
&subject \ to \quad a_i^T \le b_i, i = 1, \cdots, m.
\end{align\*}
其中向量$c,a_1,\cdots,a_m \in \mathbb{R}^n $, 和标量$b_1,\cdots, b_m \in \mathbb{R}$是指定目标函数和约束函数条件的参数。

#### 求解线性规划(solving linear programs)
- 除了一个特例，没有解析解公式(和least-squares不同)；
- 有可靠且高效的算法实现；
- 时间复杂度是$O(n^2 m)$, m是约束条件的个数, m是维度$；
- 是一个成熟的方法。

#### 应用(using linear programs)
一些应用直接使用线性规划的标准形式,或者其中一个标准形式。在很多时候，原始的优化问题没有一个标准的线性规划形式，但是可以被转化为等价的线性规划形式。比如切米雪夫近似问题(Chebyshev approximation problem)。它的形式如下：
$$minimize \quad max_{i=1,\cdots,k}|a_i^T x-b_i|$$
其中$x\in \mathbb{R}^n $是变量，$a_1,\cdots,a_k \in \mathbb{R}^n , b_1,\cdots,b_k \in \mathbb{R}$是实例化的问题参数,和least-squares相似的是，它们的目标函数都是项$a^T_i x-b_i$。不同之处在于，least-squares用的是该项的平方和作为目标函数，而Chebyshev approximation中用的是绝对值的最大值。Chebyshev approximation problem的目标函数是不可导的(max operation), least-squares problem的目标函数是二次的(quadratic), 因此可导的(differentiable)。

## 凸优化(Convex optimization)
凸优化问题是优化问题的一种,它的目标函数和优化函数都是凸的。
具有以下形式的问题是一种凸优化问题：
\begin{align\*}
&minimize \quad f_0(x)\\\\
&subject \ to \quad f_i(x) \le b_i, i = 1,\cdots,m.
\end{align\*}
其中函数$f_0,\cdots,f_m:\mathbb{R}^n \rightarrow \mathbb{R}$是凸的(convex), 如满足
$$f_i(\alpha x+ \beta y) \le \alpha f_i(x) + \beta f_i(y)$$
对于所有的$x,y \in \mathbb{R}^n $和所有的$\alpha, \beta \in \mathbb{R}$且$\alpha + \beta = 1, \alpha \ge 0, \beta \ge 0$。
或者：
$$f_i(\theta x+ (1-\theta) y) \le \theta f_i(x) + (1 - \theta) f_i(y)$$
其中$\theta \in [0,1]$。
课上有人问这里为$\theta$是0和1, 有没有什么物理意义，Stephen Boyd回答说这是定义，就是这么定义的。
The least-squares和linear programming problem都是convex optimization problem的特殊形式。线性函数(linear functions)也是convex，它们正处在边界上，它们的曲率(curvature)为0。一种方式是用正曲率去描述凸性。

### 凸优化求解(solving convex optimization problems)
- 没有解析解；
- 有可靠且有效的算法；
- 时间复杂度正比于$max{n^3 , n^2 m,F},$F$是评估$f$和计算一阶导数和二阶导数的时间；
- 有成熟的方法，如interior-point methods。

### 凸优化的应用(using convex optimization)
将实际问题形式化称凸优化问题。

## 非线性优化(Nonlinear optimization)

### 非线性优化
非线性优化用来描述目标函数和约束函数都是非线性函数(但不是凸的)优化问题。因为凸优化问题包括least-squares和linear programming, 它们是线性的。刚开始给出的优化问题就是非线性优化问题，目前没有有效的方法解该问题。目前有一些方法来解决一般的非线性问题，但是都做了一些compromise。

#### 局部优化(local optimization)
局部优化是非线性优化的一种解法，compromise是寻找局部最优点，而不是全局最优点，在可行解附近最小化目标函数，不保证能得到一个最小的目标值。
局部优化需要随机初始化一个初值，这个初值很关键，很大程度的影响了局部解得到的目标值, 也就是说是一个初值敏感的算法。关于初始值和全局最优值距离有多远并没有很多有用的信息。局部优化对于算法的参数值很敏感，需要根据具体问题去具体调整。
使用局部优化的方法比解least-squares problems, linear program, convex optimization problem更有技巧性，因为它牵扯到算法的选择，算法参数的选择，以及初值的选取。

#### 全局优化(global optimization)
全局优化也是非线性优化的一种解法, 在全局优化中，优化目标的全局最优解被找到， compromise是效率。
#### 凸优化问题在非凸优化问题中的应用(role of convex optimization in nonconvex problems)
##### 初始化局部优化(initialization for local optimization)
##### 用于非凸优化的凸的启发式搜索(convex heuristics for nonconvex optimization)
##### 全局最优的边界(bounds for global optimization)

## 大纲(outline)
### 理论(part one: Theory)
第一部分是理论，给出一些概念和定义，第一章是Introduction, 第二章和第三章分别介绍凸集(convex set)和凸函数(convex function), 第四章介绍凸优化问题， 第五章引入拉格朗日对偶性。 

### 应用(part two: Applications)
第二部分主要给出凸优化在一些领域的应用，如概率论与数理统计，经济学，计算几何以及数据拟合等领域。
凸优化如何应用在实践中。

### 算法(part three: Algorithms) 
第三部分给出了凸优化的数值解法，如牛顿法(Newton's algorithm)和内点法(interior-point)。
第三部分有三章，分别包含了无约束优化，等式约束优化和不等式约束优化。章节之间是递进的，解一个问题被分解为解一系列简单问题。二次优化问题(包括，如least-squares)是最底层的基石，它可以通过线性方程组精确求解。牛顿法，在第十章和第十一章介绍到，是下个层次，无约束问题或者等式约束问题被转化成一系列二次优化问题的求解。第十一章介绍了内点法，是最顶层, 这些方法将不等式约束问题转化为一系列无约束或者等式约束的问题。

## 参考文献
1.stephen boyd. Convex optimization

