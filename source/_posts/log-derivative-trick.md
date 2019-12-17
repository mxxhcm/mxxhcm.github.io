---
title: log derivative trick
date: 2019-09-12 19:21:10
tags:
 - 强化学习
categories: 强化学习
mathjax: true
---

## 什么是log derivative trick
$$\nabla_{\theta} log p(\mathbf{x}; \theta) = \frac{\nabla_{\theta} p(\mathbf{x}; \theta)}{ p(\mathbf{x}; \theta)}$$

## 为什么？
导数公式：
$$(log_a x)' = \frac{1}{x ln\ a}$$
$$(ln\ x)' = \frac{1}{x}$$
所以：
$$(ln\ p(\mathbf{x};\theta))' = \frac{1}{p(\mathbf{x}; \theta)}$$
\begin{align\*}
\nabla_{\theta} ln\ p(\mathbf{x};\theta) &=\left[\frac{\partial ln\ p(\mathbf{x}; \theta)}{\partial \theta} \right]^T \\\\
&= \left[\frac{\partial ln\ p(\mathbf{x}; \theta)}{\partial p(\mathbf{x};\theta)}\frac{\partial p(\mathbf{x};\theta)}{\partial \theta} \right]^T  \\\\
&= \left[\frac{1}{ p(\mathbf{x};\theta)}\frac{\partial p(\mathbf{x};\theta)}{\partial \theta} \right]^T \\\\
&= \frac{1}{ p(\mathbf{x};\theta)}\left[\frac{\partial p(\mathbf{x};\theta)}{\partial \theta} \right]^T \\\\
&=\frac{\nabla_{\theta}p(\mathbf{x}; \theta)}{p(\mathbf{x}; \theta)}\\\\
\end{align\*}

其实就是$\nabla log\ p = \frac{\nabla p}{p}$。

## 参考文献
1.https://math.stackexchange.com/questions/2554749/whats-the-trick-in-log-derivative-trick
