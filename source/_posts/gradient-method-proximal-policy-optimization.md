---
title: gradient method proximal policy optimization
date: 2019-09-23 16:57:43
tags:
 - proximal policy optimization
 - ppo
 - policy gradient
categories: 强化学习
mathjax: ture
---

## Abstract
标准的policy gradietn每一次更新需要一个样本，本文提出的PPO能够使用minibatch更新。PPO有着TRPO的优势，但是更容易实现，更普遍，更好的采样复杂度。

## Introduction
### Policy Gradient
目标函数：
$$ L^{PG} (\theta) = \hat{\mathbb{E}}_t \left[\log \pi_{\theta}(a_t|s_t)\hat{A}_t \right] \tag{1}$$
约束条件：
$$\vert d\theta\vert^2 \le \delta \tag{2}$$

### Natural Policy Gradient
目标函数：
$$ L^{NPG} (\theta) = \hat{\mathbb{E}}_t \left[\log \pi_{\theta}(a_t|s_t)\hat{A}_t  \right]\tag{3}$$
约束条件：
$$\frac{1}{2}\vert d\theta \text{H} \vert^2 \le \delta \tag{4}$$

### Trust Region Policy Optimization
目标函数：
$$ L^{PG} (\theta) = \hat{\mathbb{E}}_t \left[\frac{\pi_{\theta}(a_t|s_t)}{\pi_{old}(a_t|s_t)}\hat{A}_t \right]\tag{5}$$
约束条件：
$$\frac{1}{2}\vert d\theta \text{H} \vert^2 \le \delta \tag{6}$$

### Proximal Policy Optimization


## 参考文献
1.https://arxiv.org/pdf/1707.06347.pdf
2.https://medium.com/@jonathan_hui/rl-proximal-policy-optimization-ppo-explained-77f014ec3f12
