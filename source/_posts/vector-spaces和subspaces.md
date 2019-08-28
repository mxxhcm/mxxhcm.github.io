---
title: vector spaces和subspaces
date: 2019-08-26 19:17:41
tags:
 - spaces
 - subspaces
 - 线性代数
categories: 线性代数
---

## Linear of Combinations
线性组合有两种：加法和数乘。

### 定义
如果$v$和$w$是column vectors，$c,d$是标量，那么$cv+dw$是$v$和$w$的线性组合。


## Spaces
Vector spaces是向量的集合，通常表示为$\mathbb{R}^1 , \mathbb{R}^2 , \mathbb{R}^n $。$\mathbb{R}^5 $表示所有$5$维的column vectors。
### 定义
Space $\mathbb{R}^n $是所有$n$维column vectors $v$组成的space。

## Subspaces
### 定义
某个vector space的subspace是满足以下条件的vectors的集合，如果$v$和$w$是subspace中的vectors，并且$c$是任意的salar，
1. $v+w$还在subspace中
2. $cv$还在subspace

也就是说subspace是对加法和数乘封闭的vectors set，所有的线性组合仍然还在这个subspace。

### 例子
1. 所有的subspace都包括zero vector。
2. 通过原点的直线都是subspace。
3. 包含$v$和$w$的subspace一定得包含所有的线性组合$cv+dw$
4. 给定两个subspace $S,T$
    1. S\cup T不是一个subspace
    2. S\cap T是一个subspace，证明
    假设$v,w$是$S\cap T$的，则$v,w\in S, v,w\in T$，$v+w\in S, v+w\in T, cv+dw \in S, cv+dw \in T$，所以$cv+dw \in S\cap T$

## Column Space
### 创建矩阵的subspace
取矩阵$A$的column vectors，计算它们的所有线性组合，借得到了一个subspace

### 定义
给定矩阵$A$，$A$的所有column vectors的linear combinations组成的subspace称为column space，用$C(A)$表示。$C(A)$由$Ax$的所有可能取值构成。

### 性质
1. 当且仅当$b$在$A$的column space中，$Ax=b$才有解。
2. 假设A$是$m\times n$矩阵，$A$的column space是$R^m$的subspace。

## Null Space
### 定义
矩阵$A$的null space是所有$Ax=0$的解构成的vector space，用$N(A)$表示。$N(A)$是$\mathbb{R}^n $的subspace，因为$x$是在$\mathbb{R}^n $中的$n$维向量，所以是$\mathbb{R}^n $的subspace。

## Special solution, Pivot variables和free variables, Pivot columns和free columns
### special solution
如果方程数量小于未知数数量，那么这个方程（组）有无穷多个解，为了表示这个方程组，指定special solution来表示它。
如方程组
\begin{cases}x_1+2x_2=0\\\\3x_1+6x_2 = 0 \end{cases}
上面的方程组其实是一个方程$x_1+2x_2=0$，两个未知数。随便的选择一个变量，让它的值为$1$，求出另一个$x$。比如令$x_2 = 1$，那么$x_1 = -2$。我们就称$(x_1=-2,x_2=1)$为一个special solution。
再给一个例子，$x+2y+3z=0$，有两个special solution，随机选择两个变量，分别让其中一个取$1$，剩余的另一个取$0$，求解出来最后的一个变量。

### 主元，主元列，自由变量，自由列
我们通常把选定的两个变量叫做free variables，其他的那些变量叫做pivot variables。比如第一个例子中，$x_2$是free variable，$x_1$是pivot variable。第二个例子中，$x_2, x_3$是free variables，$x_1$是pivot variables。主元所在的column叫做pivot column，free variables所在的column叫做free columns。

## 秩
矩阵$A$的秩（rank），用$r(A)$表示，它等于pivots的数量，等于column space的维度，等于row space的维度。

## 消元法解$Ax=0$
两个步骤：
1. 将矩阵$A$化为三交矩阵$U$
2. 解$Ux=0$或者$Rx=0$

### 示例
矩阵$A= \begin{bmatrix}1&1&2&3\\\\2&2&8&10\\\\ 3&3&10&13\end{bmatrix}$化成三角矩阵为：$U= \begin{bmatrix}1&1&2&3\\\\0&0&4&4\\\\ 0&0&0&0\end{bmatrix}$，第一列和第三列是pivot columns，第二列和第四列是free columns，然后求出special solutions，再计算出通解。
对于每个free variabled都有一个special solution，$Ax=0$共有$r$个pivots，以及$n-r$个free variables，$A$的nullspace $N(A)$包含$n-r$个special solutions，$N(A)$具有如下的形式：
$$N = \begin{bmatrix} -F\\\\I\end{bmatrix}$$
其中$F$为free variables取特值的时候，pivtos的取值，$I$为free variables的取值。

## Thre reduced row echelon matrix（行简化阶梯形矩阵）
行简化阶梯形矩阵是pivot colunmn恰好构成单位矩阵的矩阵，如：
$$U = \begin{bmatrix}1&1&0&1\\\\0&0&1&1\\\\0&0&0&0\end{bmatrix}$$
所有的pivots都是$1$，主元所在列的其余位置都是$0$。
行简化阶梯形矩阵给出了很多有用信息：
1. pivot columns
2. pivot rows
3. pivots是$1$
4. zero rows显示这一行是其他rows的lihear combination
5. free columns


## $Ax=b$的通解
先求出particular solution，让所有的free variables取$0$，求解出pivots，得到一个solution，我们称它为particular solution。然后求出所有的special solutions，则$Ax=b$的通解可以表示为：
$$x = x_p + a x_{special_solution_1} + b x_{special_solution} + \cdots=x_p+x_n$$
即particular solution加上nullspace组成的新的vector sets。当没有free variables的时候，也就没有special solutions，nullspace为空。

<!--当$b=0$的时候，我们可以求出通解$x$，当$b\neq 0$时，有点难。通过使用增广矩阵：$\left[A\ b\right]$，然后进行消元，得到$\left[R\ d\right]$，$R$是行间化阶梯形矩阵，$d$是$b$做了和$A$一样的变换后的结果。-->

## 列满秩
### 定义
对于$m\times n$的矩阵$A$，每一列都有一个pivot，rank $r=n$，matrix是瘦高的$(m\ge n)$，其实就相当于每一个column vector都用到了，没有多余的column vector。可以用以下的形式表示：
$$R = \begin{bmatrix}I\\\\0\end{bmatrix}=\begin{bmatrix}n\times n 单位矩阵\\\\m-n行零向量\end{bmatrix}$$

### 属性
当$A$列满秩的时候，有以下结论：
1. $A$的所有columns都是pivot columns
2. 没有free variables，free columns和special solutions
3. Nullspace只有$x=0$ 
4. 如果$Ax=b$有解，那么它只有一个解，或者一个解都没有


## 行满秩
### 定义
对于$m\times n$的矩阵$A$，如果$r=m$的话，$A$是一个矮胖的矩阵$(m\le n)$，每一行都有一个pivot。
$$R = \begin{bmatrix}I&F\end{bmatrix}=\begin{bmatrix}m\times m 单位矩阵&F\end{bmatrix}$$

### 属性
当$A$行满秩的时候，有以下结论：
1. $A$的所有row都有pivots，$R$没有$0$向量
2. 对于任何$b$，$Ax=b$都有解
3. column space就是整个$\mathbb{R}^m$
4. 总共有$n-r= n-m$个special solutions。


## 秩和方程解个数之间的关系
1. $r=m, r=n$,可逆方阵，$Ax=b$有且只有一个解；
2. $r=m, r\lt n$,矮胖，$Ax=b$有无穷多个解，一个particular solution加上nullspace中的无穷个。
3. $r\lt m, r=m$,瘦高，$Ax=b$没有或者只有一个解，如果$b$恰好在$A$的column space中有一个解，如果$b$恰好不在$A$的column space中无解，因为column vectors是相互独立的，所以$Ax=0$只有零解。
4. $r\l tm, r\lt n$,并不满秩，$Ax=b$无解或者有无穷多个解，无解的情况是不在$A$的column space中，有解的情况是 在$A$的column space中，而在这部分中，又有无穷多个零解，所以要不无解要不无穷多个解。

对应的行间化阶梯形矩阵如下所示：
$\begin{bmatrix}I\end{bmatrix},\begin{bmatrix}I&F\end{bmatrix},\begin{bmatrix}I\\\\0\end{bmatrix},\begin{bmatrix}I&F\\\\0&0\end{bmatrix}$


## Linear independence
矩阵$A$的columns是linear independent的，当且仅当$Ax=0$的唯一解是$x=0$时。也就是说$A$的nullspace只有零向量的时候。
### 定义
给定一系列向量$v_1, \cdots, v_n$，$c_1 v_1 +\cdots+c_n v_n=0$当且仅当$c_1, \cdots, c_n=0$时候成立。

## Span
### 定义
使用一系列vectors生成space的过程就叫做span。

## Row Space
### 定义
使用矩阵的row vector生成的subspace就叫做row space，表示维$C(A^T)$，它和$A^T$的column space是相同的。

## Basis
### 定义
生成space的最小vectors的independent vectors叫做这个space的一组basis，basis不是唯一的。

### 示例
矩阵的pivot columns是它的column space的一组basis。

## Dimension
### 定义
Space的dimension指的是每组basis中向量的个数。对于一个space，不同的baisis，它们的vectors不同，但是向量的个数都是相同的，这是space的属性。


## Rank和dimension的关系
和矩阵$A$相关的主要有四个subspace，分别是column space, nullspace, row space以及left nullspace。它们四个具有的属性如下所示：
1. row space和column space的dimension都是$r$
2. nullspace和left nullspace的dimension是$n-r, m-r$，为什么是$n-r,m-r$，解$Ax=0$得到$x$是$n$维向量，也就是nullspace是$\mathbb{R}^n$的subspace，$A$的column space的dimension是$r$，free variables，free columns的个数就是$n-r$，special solutions的个数就是$n-r$，而nullspace的basis其实就是所有的special solutions，所以nullspace的dimension就是$n-r$，$m-r$同理。


## $A$和$R$ dimension和basis的关系
这里的$A$是矩阵，$R$是行间化阶梯形矩阵
1. $A$和$R$的row space相同，dimension相同，为$r$，basis相同
2. $A$和$R$的column space不同，dimension相同，也为$r$，basis不同
3. $A$和$R$的nullspace相同，dimension相同，为$n-r$,basis相同
4. $A$和$R$的left nullspace不同，dimension相同，为$m-r$

Row space和nullspace都是$\mathbb{R}^n $的subspace，它们的dimension加起来等于n，但是这两个subspace加起来并不是$\mathbb{R}^n $。Row space是对$r$个$n$维pivot row vectors进行linear combination，而nullspace是对$n-r$个$n$维的解向量$x$进行linear combination，这里虽然都出现了$n$，但是第一个$n$是row vector的长度$n$，而第二个$n$是解向量的$n$。
同理，可以得column space和left nullspace都是$\mathbb{R}^m$的subspace。


## 参考文献
1.MIT线性代数公开课
