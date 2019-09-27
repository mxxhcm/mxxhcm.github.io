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
$$ L^{PG} (\theta) = \hat{\mathbb{E}}\_t \left[\log \pi_{\theta}(a_t|s_t)\hat{A}\_t \right] \tag{1}$$
约束条件：
$$\vert d\theta\vert^2 \le \delta \tag{2}$$

### Natural Policy Gradient
目标函数：
$$ L^{NPG} (\theta) = \hat{\mathbb{E}}\_t \left[\log \pi_{\theta}(a_t|s_t)\hat{A}\_t  \right]\tag{3}$$
约束条件：
$$\hat{\mathbb{E}}\_t\left[\text{KL}\left[\\pi_{old}(\cdot|s_t), \pi_{\theta}(\cdot|s_t)\right] \right] \tag{4}$$
等价于
$$\frac{1}{2} d\theta^T \text{H} d\theta \le \delta \tag{5}$$

### Trust Region Policy Optimization
目标函数：
$$ L^{PG} (\theta) = \hat{\mathbb{E}}\_t \left[\frac{\pi_{\theta}(a_t|s_t)}{\pi_{old}(a_t|s_t)}\hat{A}\_t \right]\tag{6}$$
约束条件：
$$\hat{\mathbb{E}}\_t\left[\text{KL}\left[\pi_{old}(\cdot|s_t), \pi_{\theta}(\cdot|s_t)\right] \right] \tag{7}$$

### Proximal Policy Optimization
目标函数：
$$L^{PPO}(\theta) =\hat{\mathbb{E}}\_t \left[L_t^{CLIP+VF+S}(\theta) - \beta\text{KL}\left[\pi_{old}(\cdot|s_t), \pi_{\theta}(\cdot|s_t) \right] \right] \tag{8}$$
其中$S$表示entropy bonus。
$$L_t^{CLIP}(\theta) = \hat{\mathbb{E}}\_t \left[\min(\frac{\pi_{\theta}(\cdot|s_t)}{\pi_{old}(\cdot|s_t)},\ clip(\frac{\pi_{\theta}(\cdot|s_t)}{\pi_{old}(\cdot|s_t)}, 1-\epsilon, 1+\epsilon) \hat{A}\_t) \right]\tag{9}$$
$$L_t^{VF} = (V_{\theta}(s_t) - V_t^{targ} )^2 \tag{10}$$


## 参考文献
1.https://arxiv.org/pdf/1707.06347.pdf
2.https://medium.com/@jonathan_hui/rl-proximal-policy-optimization-ppo-explained-77f014ec3f12
