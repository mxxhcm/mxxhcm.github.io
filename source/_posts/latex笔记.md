---
title: latex笔记
date: 2018-12-22 10:08:26
tags: 
  - Latex
categories: 工具
mathjax: true
---

## 命令重命名
在写博客时也能用
``` Latex
\newcommand{\mmm}{\mathbf}
\mmm{x}
\bf{x}
```
$\newcommand{\mmm}{\mathbf}$
$\mmm{x}$
$\bf{x}$

## 常用Latex符号

### 等号
$\sim$  \sim
$\simeq$    \simeq
$\approx$   \approx
$\cong$ \cong
$\equiv$    \equiv
$\propto$ \propto
### 各种乘法
$\times$ \times
$\*$ *
$\cdot$ \cdot
$\bullet$ \bullet
$\otimes$ \otimes
$\circ$ \circ
$\odot$ \odot

### 上下花括号
$\overbrace{x+y}\^{1+2}=\underbrace{z}_3$ \overbrace{x+y}\^{1+2}=\underbrace{z}_3

### 括号
\left(\frac{1}{2}\right)    $\left(\frac{1}{2} \right)$
\left[\frac{1}{2} \right]    $\left[\frac{1}{2} \right]$
\left\\{\frac{1}{2} \right\\}    $\left\\{\frac{1}{2} \right\\}$
``` Latex
\begin{cases}x=1\\\\y=x\end{cases}    
```
$$\begin{cases}x=1\\\\y=x\end{cases}$$

### 矩阵
``` Latex
\begin{matrix}1&2\\\\3&4\end{matrix}
```
$$\begin{matrix}1&2\\\\3&4\end{matrix}$$
``` Latex
\begin{pmatrix}1&2\\\\3&4\end{pmatrix}
```
$$\begin{pmatrix}1&2\\\\3&4\end{pmatrix}$$
``` Latex
\begin{bmatrix}1&2\\\\3&4\end{bmatrix}
```
$$\begin{bmatrix}1&2\\\\3&4\end{bmatrix}$$
``` Latex
\begin{Bmatrix}1&2\\\\3&4\end{Bmatrix}
```
$$\begin{Bmatrix}1&2\\\\3&4\end{Bmatrix}$$
``` Latex
\begin{vmatrix}1&2\\\\3&4\end{vmatrix}
```
$$\begin{vmatrix}1&2\\\\3&4\end{vmatrix}$$
``` Latex
\begin{Vmatrix}1&2\\\\3&4\end{Vmatrix}
```
$$\begin{Vmatrix}1&2\\\\3&4\end{Vmatrix}$$

### 希腊字母
$\eta$ \eta
$\gamma$ \gamma
$\Gamma$ \Gamma
$\epsilon$ \epsilon
$\varepsilon$ \varepsilon
$\lambda$ \lambda
$\Lambda$ \Lambda
$\sigma$ \sigma
$\Sigma$ \Sigma
$\phi$ \phi
$\varphi$ \varphi
$\Phi$ \Phi
$\Delta$ \Delta
$\delta$ \delta
$\nabla$ \nabla
$\zeta$ \zeta
$\xi$ \xi

### 字体
$\mathbf{A}$ \mathbf{A}
$\boldsymbol{A}$ \boldsymbol{A}
$\mathit{A}$ \mathit{A}
$\mathrm{A}$ \mathrm{A}
$\mathcal{A}$ \mathcal{A}

### 数学运算
求积$\prod$ \prod
求和$\sum$ \sum
积分$\int$ \int
根号$\sqrt{x}$ \sqrt{x}
根号$\sqrt[4]{y}$ \sqrt[4]{y}
分数$(\frac{1}{2})$ (\frac{1}{2})
分数$\left(\frac{1}{2}\right)$ \left(\frac{1}{2}\right)
无穷$\infty$ \infty
期望$\mathbb{E}$ \mathbb{E}
范数$\Vert$ \Vert
$\mathbb{\pi}$ \mathbb{\pi} # 可以看出来，没有起作用，因为mathbb没有只支持大写字母。
$\pm$ \pm
$\mp$ \mp


### 集合
真含于$\subset$ \subset
含于$\subsetneqq$ \subsetneqq
真包含$\supset$ \supset
包含$\supsetneqq$ \supsetneqq
交$\cap$ \cap
并$\cup$ \cup
属于$\in$ \in
$\succ$ \succ
$\succeq$ \succeq
$\prec$ \prec
$\preceq$ \preceq
空集$\emptyset$ \emptyset

### 谓词逻辑
否定$\neg$ \neg
任意$\forall$ \forall
存在$\exists$ \exists
合取$\wedge$ \wedge
析取$\vee$ \vee

### 空格
$a\qquad b$ a\qquad b
$a\quad b$ a\quad b
$a\ b$ a\ b
$a\;b$ a\;b
$a\,b$ a\,b
$ab$ ab
$a\!b$ a\!b

### 其他
长竖线$\big|$ \big|
长竖线$\Big|$ \Big|
长竖线$\bigg|$ \bigg|
长竖线$\Bigg|$ \Bigg|
双箭头$\Leftrightarrow$ \Leftrightarrow
左箭头$\leftarrow$ \leftarrow 
右箭头$\rightarrow$ \rightarrow 
上划线$\overline{A}$ \overline{A}
下划线$\underline{A}$ \underline{A}
$\backslash$ \backslash
$\sim$ \sim

## 列表
### 有序列表
``` Latex
\begin{enumerate}
 \item First.
 \item Second.
 \item Third.
\end{enumerate}
```
效果如下：
1. First.
2. Second.
3. Third.

### 无序列表
``` Latex
\begin{itemize}
 \item {First.}
 \item {Second.}
 \item {Third.}
\end{itemize}
```
效果如下：
+ First.
+ Second.
+ Third.

## 跨多行公式对齐
**注意：不要忘了每行后面的两个\\**
### 示例1
``` Latex
\begin{align*}
f(x) &= (3 + 4)\^2 + 4\\
&= 7\^2 + 4\\
&= 49 + 4\\
&= 53
\end{align*}
```
效果如下：
\begin{align\*}
f(x) &= (3 + 4)\^2 + 4\\\\
&= 7\^2 + 4\\\\
&= 49 + 4\\\\
&= 53
\end{align\*}
### 示例2
``` Latex
\begin{align*}
v &= R + \gamma Pv\\
(1-\gamma P) &= R\\
v &= (1 - \gamma P)\^{-1} R
\end{align*}
```
\begin{align\*}
v &= R + \gamma Pv\\\\
(1-\gamma P) &= R\\\\
v &= (1 - \gamma P)\^{-1} R
\end{align\*}


## 参考文献
1.http://blog.huangyuanlove.com/2018/02/27/LaTeX笔记-六/
2.https://blog.csdn.net/xxzhangx/article/details/52778539
3.https://blog.csdn.net/hunauchenym/article/details/7330828
4.http://geowu.blogspot.com/2012/10/latex_25.html
5.https://math.stackexchange.com/questions/20412/element-wise-or-pointwise-operations-notation
