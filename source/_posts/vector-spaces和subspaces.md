---
title: vector spaces和subspaces
date: 2019-08-26 19:17:41
tags:
 - spaces
 - subspaces
 - 线性代数
categories: 线性代数
---

## Spaces of Vectors
Vector spaces是向量的集合，通常表示为$\mathbb{R}^1,\mathbb{R}^2,\mathbb{R}^n$。$\mathbb{R}^5$表示所有$5$个compoments的column vectors。
### 定义
Space $\mathbb{R}^n$是所有$n$个componments的column vectors $v$组成的space。

## Linear of Combinations
加法和数乘构成了线性组合。
### 定义
如果$v$和$w$是column vectors，$c,d$是标量，那么$cv+dw$是$v$和$w$的线性组合。

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

## Column Space
### 定义
给定矩阵$A$，$A$的Column space由$A$的所有columns的linear combinations组成，用$C(A)$表示，由$Ax$的所有可能取值构成。

### 性质
1. 当且仅当$b$在$A$的column space中，$Ax=b$才有解。
2. 假设A$是$m\times n$矩阵，$A$的column space是$R^m$的subspace。

## Null Space
### 定义
矩阵$A$的null space是所有$Ax=0$的解构成的vector space。这些vectors $x$在$\m athbb{R}^n$中，用$N(A)$表示。
