---
title: gradient method natrual policy gradient
date: 2019-09-22 19:38:03
tags:
 - gradient method
 - natural policy gradient
 - 强化学习
categories: 强化学习
mathjax: true
---

## A Natural Policy Gradient
论文名称：A Natural Policy Gradient 
论文地址：https://papers.nips.cc/paper/2073-a-natural-policy-gradient.pdf

## 摘要
作者基于参数空间的底层结构提出了natural gradient方法，找出下降最快方向。尽管gradient方法不能过大的改变参数，它还是能够朝着选择greedy optimal action而不是更好的action方向移动。基于兼容值函数的policy iteration，在每一个improvement step选择greedy optimal action。

## 引言
直接的policy gradient在解决大规模的MDPs时很有用，这种方法基于future reward的梯度在满足约束条件的一类polices中找一个$\pi$，但是这种方法是non covariant的，简单来说，就是左右两边的维度不一致。
这篇文章基于policy的底层参数结构定义了一个metric，提出了一个covariant gradient方法，通过将它和policy iteration联系起来，可以证明natural gradient朝着选择greedy optimal action的方向移动。通过在简单和复杂的MDP中进行测试，结果表明这种方法中没有出现严重的plateau phenomenon。

## A Natural Gradient
定义average reward $\eta(\pi)$为：
$$\eta(\pi) = \sum_{s,a}\rho^{\pi} (s) \pi(a;s) R(s, a) \tag{1}$$
其中$R(s,a) = \mathbb{E}\left[R_{t+1}\right|s_t=s, a_t = a]$，state action value和value function定义如下：
$$Q^{\pi} (s,a) = \sum_{t=0}^{\infty} \mathbb{E}\left[R_t - \eta(\pi)|s_0=s,a_0=a,\pi\right], \forall s\in S, a\in A \tag{2}$$
$$V^{\pi} (s) = \mathbb{E}\_{\pi(a';s)}\left[Q^{\pi}(s,a')\right] \tag{3}$$
计算average reward的精确梯度是（可以看[policy gradient]()的推导）：
$$\nabla\eta(\theta) = \sum_{s,a} \rho^{\pi} (s) \nabla \pi(a;s,\theta) Q^{\pi} (s,a) \tag{4}$$
使用$\eta(\theta)$代替了$\eta(\pi_{\theta})$。$\eta(\theta)$下降最快的方向定义为在$d\theta$的平方长度$\vert d\theta\vert^2 $ 等于一个常数时，使得$\eta(\theta+d\theta)$最小的$d\theta$的方向。平方长度的定义和一个正定矩阵$G(\theta)$有关，即：
$$\vert\theta\vert^2 = \sum_{ij} G_{ij} (\theta)d\theta_i d\theta_j = d\theta^T G(\theta) d\theta  \tag{5}$$
可以证明，最块的梯度下降方向是$G^{-1} \nabla \eta(\theta)$。标准的policy gradient假设$\text{G}=\text{I}$，所以最陡的下降方向是$\nabla\eta(\theta)$。作者的想法是选择一个其他的$\text{G}$，新的metric不根据坐标轴的选择而变化，而是跟着坐标参数化的mainfold变化。根据新的metric定义natural gradient。策略$\pi(a;s,\theta)$的fisher information是：
$$\text{F}\_s(\theta) = \mathbb{E}\_{\pi(a;s,\theta)} \left[\frac{\partial \log \pi(a;s,\theta)}{\partial \theta_i} \frac{\partial \log \pi(a;s,\theta)}{\partial \theta_j}\right] \tag{48}$$
显然$\text{F}_s$是正定矩阵，可以证明，FIM是概率分布参数空间上的一个invariant metric。不论两个点的坐标怎么选择，它都能计算处相同的distance，所以说它是invariant。
当然，$\text{F}\_s$只用了单个的$s$，而在计算average reward时，使用的是一个分布，定义metric为：
$$\text{F}(\theta) = \mathbb{E}\_{\rho^{\pi} (s)} \left[\mathbb{F}_s (\theta)\right] \tag{49}$$
每一个$s$对应的单个$\text{F}_s$都和MDP的transition model没有关系，期望操作引入了每一个transition model的参数。直观上来说，$\text{F}_s$测量的是在$s$上的probability manifold的距离，$\text{F}(\theta)$对它们进行了平均。对应的下降最快的方向是：
$$\hat{\nabla}\eta(\theta) =\text{F}(\theta)^{-1} \nabla\eta(\theta)  \tag{50}$$
为什么natural gradient下降最快是这个方向，接下来我们进行证明。$\text{KL}$散度在$\theta=\theta'$附近$\theta' +d, d\rightarrow 0$处的二阶泰勒展开是：
$$\text{KL}\left[p(x|\theta')||p(x|\theta'+d)\right] \approx \frac{1}{2}d^T \text{F}d \tag{6}$$
证明：
\begin{align\*}
\text{KL}\left[p\_{\theta'}||p\_{\theta'+d}\right] &\approx \text{KL}\left[p\_{\theta'}||p\_{\theta'}\right] + (\nabla\_{\theta}\text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'})^T (\theta'+d -\theta') \\\\
&\qquad\qquad\qquad+ \frac{1}{2} (\theta' +d -\theta')^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'})(\theta'+d-\theta')\tag{7}\\\\
& = \text{KL}\left[p\_{\theta'}||p\_{\theta'}\right] + (\nabla\_{\theta}\text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'})^T d + \frac{1}{2} d^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'}) d\tag{8}\\\\
& = \text{KL}\left[p\_{\theta'}||p\_{\theta'}\right] + (\int_x p(x|\theta')\nabla \log (p|\theta)|\_{\theta=\theta'} dx)^T d + \frac{1}{2} d^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'}) d\tag{9}\\\\
& = \text{KL}\left[p\_{\theta'}||p\_{\theta'}\right] + (\mathbb{E}\_{p(x|\theta')} \nabla\log p(x|\theta) dx|\_{\theta=\theta'})^T d + \frac{1}{2} d^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'}) d\tag{10}\\\\
& = 0 + 0 + \frac{1}{2} d^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'}) d\tag{11}\\\\
& = \frac{1}{2} d^T (\nabla\_{\theta}^2 \text{KL}\left[p\_{\theta}||p\_{\theta'}\right]|\_{\theta=\theta'}) d\tag{12}\\\\
& = \frac{1}{2} d^T \text{H} d\tag{13}\\\\
& = \frac{1}{2} d^T \text{F} d\tag{14}\\\\
\end{align\*}
我们想要找到使得loss函数$L(\theta)$最小的$d$，我们想要直知道哪个方向的$\text{KL}$散度下降的最快，就是使用$\text{KL}$散度当做一个metric，而不是使用欧几里得metric。目标函数是：
$$d^{*} = \arg \min L(\theta +d) \tag{15}$$
约束条件
$$\text{KL}\left[p_{\theta}||p_{\theta'}\right] = c \tag{16}$$
其中$c$是常数，确保更新在一定范围内，不受curvature的影响。目标函数的一节泰勒展开公式如下：
begin{align\*}
L_{\theta'}(\theta) & = L_{\theta'}(\theta') + \left[\nabla_{\theta}L_{\theta'}(\theta)|\_{\theta=\theta'}\right]^T (\theta' + d - \theta') + \cdots \\\\
& = L\_{\theta'}(\theta') + \left[\nabla_{\theta}L_{\theta'}(\theta)|\_{\theta=\theta'}\right]^T d + \cdots  \tag{17}\\\\
\end{align\*}
使用拉格朗日乘子法将约束条件带入：
\begin{align\*}
d^{\*} & = {\arg \min}\_d L(\theta'+d) + \lambda(\text{KL}\left[p\_{\theta'}||p\_{\theta'+d}\right] -c)\\\\
& = {\arg \min}\_d L\_{\theta'}(\theta') + \left[\nabla_{\theta}L_{\theta'}(\theta)|\_{\theta=\theta'}\right]^T d + \lambda(\left[\frac{1}{2} d^T \text{F} d\right] -c)\tag{18}\\\\
\end{align\*}
对$d$求导，令其等于$0$，得：
\begin{align\*}
&0 + \nabla_{\theta}L_{\theta'}(\theta)|\_{\theta=\theta'} + \text{F}d + 0\\\\
=& \nabla_{\theta}L_{\theta'}(\theta)|\_{\theta=\theta'} + \text{F}d \tag{19}\\\\
=& 0\\\\
\end{align\*}
求解得到：
$$d= - \frac{1}{\lambda}\text{F}^{-1} \nabla_{\theta'} L(\theta') \tag{20}$$
所以natural gradient定义为：
$$\hat{\nabla}\eta(\theta) = \text{F}^{-1} \nabla_{\theta}L(\theta) \tag{21}$$


## The Natural Gradient 和 Policy Iteration
使用$\omega$参数化的值函数$f^{\pi} (s,a;\omega)$近似$Q^{\pi} (s,a)$。
### Natural Gradient with Approximation（使用近似的自然梯度）
定义：
$$\psi(s,a)^{\pi} = \nabla \log \pi(a;s, \theta), \qquad f^{\pi} (s,a;\omega) = \omega^T \psi^{\pi} (s,a) \tag{51}$$
其中$\left[\nabla \log \pi(a;s, \theta)\right]\_i = \frac{\partial \log \pi(a;s, \theta)}{\partial \theta_i}$。找到最小化均方根误差函数的$\omega$，记为$\hat{\omega}$：
$$\epsilon(\omega, \pi) = \sum_{s,a}\rho^{\pi} (s)\pi(a;s,\theta)(f^{\pi} (s,a;\omega) - Q^{\pi} (s,a))^2 \tag{52}$$
如果使用$f^{\pi} $代替$Q$计算出来的grdient还是exact的，就称$f$是兼容的。

#### 定理1
如果$\hat{\omega}$是使得均方误差$\epsilon(\omega,\pi_\theta)$最小的$\omega$，可以证明：
$$\hat{\omega} = \hat{\nabla} \eta(\theta) =\text{F}(\theta)^{-1} \nabla\eta(\theta) =\text{F}(\theta)^{-1} \nabla\eta(\theta) \tag{53}$$
证明：
因为$\hat{\omega}$使得$\epsilon$最小，所以当$\omega = \hat{\omega}$时，$\frac{\partial \epsilon}{\partial \omega} = 0$，有：
$$\frac{\partial \epsilon}{\partial \omega} = \sum_{s,a}\rho^{\pi} (s) \pi(a|s;\theta) \psi^{\pi} (s,a) (\psi^{\pi} (s,a)^T \hat{\omega} - Q^{\pi} (s,a)) = 0 \tag{54}$$
移项合并同类项得：
$$\sum_{s,a}\rho^{\pi} (s) \pi(a|s;\theta) \psi^{\pi} (s,a) \psi^{\pi} (s,a)^T \hat{\omega} = \sum_{s,a}\rho^{\pi} (s) \pi(a|s;\theta) \psi^{\pi} (s,a)  Q^{\pi} (s,a) \tag{55}$$
根据定义$\psi(s,a)^{\pi} = \nabla \log \pi(a;s, \theta)$，而根据log-derativate trick：$\pi(a|s) \nabla \log \pi(a|s;\theta) = \nabla \pi(a|s;\theta)$，所以式子$(54)$右面就是$\nabla \eta$，而式子左面$\sum_{s,a}\rho^{\pi} (s) \pi(a|s;\theta) \psi^{\pi} (s,a) \psi^{\pi} (s,a)^T = \text{F}(\theta)$。最后得到：
$$ \text{F}(\theta)\hat{\omega} = \nabla\eta(\theta)$$

### Greedy Policy Improvement
在greedy policy improvement的每一步，在$s$处，选择$a\in \arg \max_{a'} f^{\pi}(s, a';\hat{\omega})$。这一节介绍natural gradient能够找到best action，而不仅仅是一个good action。
首先考虑指数函数：$\pi(s;a,\theta) \propto e^{\text{\theta}\^T \phi_{sa}}$，其中$\phi_{sa} \in \mathbb{R}^m $是特征向量。为什么使用指数函数，因为它是affine geometry。简单来说，就是$\pi(a;s,\theta)$的probability manifold可以被弯曲。接下来证明policy在natrual gradient方向上改进的一大步等价于进行一步greedy policy improvement的policy。

#### 定理2
假设$\pi(s;a,\theta) \propto e^{\text{\theta}\^T \phi_{sa}} $，$\hat{\nabla}\eta(\theta)$是非零的，并且$\hat{\omega}$是最小化均方误差的$\omega$。令
$$\pi_{\infty}(a;s) = lim_{\alpha\rightarrow \infty}\pi(a;s,\theta + \alpha\hat{\nabla}\eta(\theta)) \tag{56}$$
当且仅当$a\in \arg\max_{a'} f^{\pi} (s,a';\hat{\omega})$时，有$\pi_{\infty}(a;s)\neq 0$。
证明：
根据定义：$f^{\pi} (s,a,\omega) = \omega^T \psi^{\pi} (s,a)$，由定理$1$可知：$\hat{\omega} = \text{F}^{-1} \nabla \eta(\theta) = \hat{\nabla} \eta(\theta)$，所以$f^{\pi}(s,a,\hat{\omega}) = \hat{\nabla}\eta(\theta)^T \psi^{\pi} (s,a)$。而根据定义$\psi^{\pi} (s,a) = \nabla \log \pi(a|s;\theta) = \phi_{sa} - \mathbb{E}\_{\pi(a'|s;\theta)}(\phi_{sa'})$，$\mathbb{E}\_{\pi(a'|s;\theta)}(\phi_{sa'})$不是$a$的函数，所以就有：
$$\arg\max_{a'}f^{\pi} (s,a';\hat{\omega}) = \arg\max_{a'} \hat{\nabla}\eta(\theta)^T \phi_{sa}\tag{57}$$
和$\mathbb{E}\_{\pi(a'|s;\theta)}(\phi_{sa'})$无关。。经过一个gradient step：
$$\pi(a|s;\theta+\alpha \hat{\nabla}\eta(\theta)) \propto e^{(\theta+\alpha \hat{\nabla}\eta(\theta))^T \phi_{sa}} \tag{58}$$
因为$\hat{\nabla}\eta(\theta) \neq 0$，很明显，当$\alpha\rightarrow \infty$时，$\hat{\nabla}\eta(\theta)^T\phi_{sa}$会dominate，所以只有当且仅当$a\in \arg\max_{a'} f^{\pi} (s,a';\hat{\omega})$时，有$\pi_{\infty}(a;s)\neq 0$。
可以看出来natural gradient趋向于选择最好的action，而普通的gradient方法只能选出来一个更好的action。
使用指数函数的目的只是为了展示在极端情况下－－有无限大的learning rate情况下的结果，接下来给出的是普通的参数化策略的结果，natural gradient可以根据$Q^{\pi} (s,a)$的局部近似估计$f^{\pi}(s,a;\hat{\omega})$，近似找到局部best action。

#### 定理3
加入$\hat{\omega}$最小化估计误差，使用$\theta' = \theta + \alpha \hat{\nabla}\eta(\theta)$更新参数，可以得到：
$$\pi(a;s,\theta') = \pi(a;s,\theta)(1+f^{\pi}(a,s,\hat{\omega})) + O(\alpha^2)\tag{59}$$
证明：
根据定理$1$，得到$\Delta \theta = \alpha\hat{\nabla}\eta(\theta) = \alpha\hat{\omega}$，然后利用一阶泰勒展开：
\begin{align\*}
\pi(a|s;\theta') &= \pi(a|s;\theta) + \frac{\partial \pi(a|s;\theta)^T }{\partial\theta}\Delta\theta + O(\theta^2 ) \\\\
&= \pi(a|s;\theta) + \frac{\partial\log \pi(a|s;\theta)^T }{\partial\theta}\pi(a|s;\theta)\Delta\theta + O(\theta^2 ) \\\\
&= \pi(a|s;\theta)(1 + \frac{\partial\log \pi(a|s;\theta)^T }{\partial\theta}\Delta\theta) + O(\theta^2 ) \\\\
&= \pi(a|s;\theta)(1 +  \psi(s, a)^T \Delta\theta) + O(\theta^2 ) \\\\
&= \pi(a|s;\theta)(1 +  \psi(s, a)^T \alpha\hat{\omega}) + O(\alpha^2 ) \\\\
&= \pi(a|s;\theta)(1 +  \alpha f^{\pi} (s, a, \hat{\omega})) + O(\alpha^2 ) \\\\
\end{align\*}
这个相当于是根据$f^{\pi}(s,a) $选择每个state的action。当然，并不是选择greedy action就一定会改善policy，还有许多例外，这里就不细说了。

## Metrics和Curvatures
在不同的参数空间中，[fisher information](https://mxxhcm.github.io/2019/09/16/fisher-information/)都可以收敛到[海塞矩阵](https://mxxhcm.github.io/2019/09/10/Jacobian-matrix-and-Hessian-matrix/)，因此，它是[aymptotically efficient](https://mxxhcm.github.io/2019/09/18/asymptotically-efficient-%E6%B8%90%E8%BF%9B%E6%9C%89%E6%95%88%E6%80%A7/)，即到达了cramer-rao bound。
$\text{F}$是$\log \pi$对应的fisher information。Fisher information 和海塞矩阵有关系，但是都需要和$\pi$联系起来。是这里考虑$\eta(\theta)$的海塞矩阵，它和$\text{F}$两个之间有一定联系，但是不一样。
事实上，定义的新的$\text{F}$并不会收敛到海塞矩阵。但是因为海塞矩阵一般不是正定的，所以在非局部最小处附近，它提供的curvature信息用处不大。在局部最小处使用conjugate methods会更好。

## 参考文献
1.https://papers.nips.cc/paper/2073-a-natural-policy-gradient.pdf
2.https://wiseodd.github.io/techblog/2018/03/14/natural-gradient/
3.https://medium.com/@jonathan_hui/rl-trust-region-policy-optimization-trpo-part-2-f51e3b2e373a

