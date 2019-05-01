---
title: Distral Robust Multitask Reinforcement Learning
date: 2019-04-18 11:04:22
tags:
 - 论文
 - 强化学习
categories: 强化学习
mathjax: true
---

这篇文章给出了多任务学习和迁移学习的一个框架。

## 摘要
使用共享网络参数的multitas learning，然后在不同的任务之间迁移可以提高效率。但是这种方法有几个缺点：
- 不同任务的gradient可能相互干扰，让学习变得不稳定，甚至很低效。
- 另一个问题是不同任务的reward scheme不同，可能会让其中的某一个reward占主导地位。

本文提出了Distral方法，并不是使用共享参数，而是使用一个提炼的policy去学习不同任务的公共行为。每一个worker解决它自己的任务，但是需要加一个限制条件，每一个单独的策略需要离提炼出的共享策略足够近，而提炼的共享策略需要在所有单独策略的质心上，通过优化一个联合的目标函数，这两个过程都可以实现。

## Introduction  
由于DRL需要的训练时间和训练数据很多，现在的DRL问题逐渐向单独的智能体（同时或者连续的）解决多个相关问题移动。由于巨大的计算开销，这个方向的研究需要设计出非常鲁棒的，与具体任务无关的算法。直观上来说，因为不同的相关任务有共同的结构，所以我们觉得它们一块应该能够促进学习，然而事实上，在实践中我们并不能总是得到这个结论。所以，multitask和transfer learning可以需要解决一个问题，在多个任务上训练会对单个任务的训练产生负面影响。所以就需要使用一些方法来结局这个问题。事实上，其他任务的梯度可能会被当做做一年，干扰学习，甚至极端情况下，其中一个任务可能会主导其他的任务。
在这篇论文中，作者提出了一种multitask和transfer RL算法，给出一些算法实例，它能够高效的跨任务共享behaviour structure。除了在grid world领域的一些指导性illustration(例证)，作者还在DeepMind Lab $3D$环境中详细分析了最终的算法和A3C baseline的比较。作者验证了Distral算法学习的很快，而且能够达到很好的收敛性能，而且对超参数很鲁棒，比multitask A3C baselines要稳定的多。

## Distral: Distill and Transfer Learning
作者给出了一个同时训练多个任务的框架，叫做Distral。如下图所示，作者给出了从四个任务中提取shared policy的一个例子。该方法用一个shared policy去提取task-specific的polices之间的common behaviour和representation，然后又用这个shared policy使用KL散度去正则化task-specific polices。这里使用KL散度的作用相当于shaping reward，鼓励exploration。最后，这些相关任务中的common knowledge都被提炼到shared policy中去了，然后可以迁移到其他任务中去。
![figure1](/figure1.png)

### 数学框架
一个multitask RL任务中，假设有$n$个任务，折扣因子为$\gamma$，不同任务的state space和action space都是相同的，但是每个任务$i$的状态转换概率$p_i(s'|s,a)$和奖励函数$R_i(s,a)$是不同的，用$\pi_i$表示task-specific的stochastic polices。给定从一些初始状态开始的state和action轨迹，用$\pi_i$表示dynamics和polices。
作者通过优化一个由expected return和policy regularization组成的目标函数将这两个过程联系起来。用$\pi_0$表示要提取的shared policy，然后通过使用$\pi_0$和$\pi_i$的KL散度$\mathbb{E}_{\pi_i}\left[\sum_{t\ge 0}\gamma^tlog\frac{\pi_i(a_t|s_t)}{\pi_0(s_t|a_t)}\right]$正则化项使用每个任务的策略$\pi_i$向$\pi_0$移动。此外，作者还使用了一个带折扣因子的entropy正则化项鼓励exploration。最后总的优化目标是：
\begin{align\*}
J(\pi_0, \{\pi_i\}_{i=1}^n) &=\sum_i\mathbb{E}_{\pi_i}\left[\sum_{t\ge 0}\gamma^tR_i(s_t,a_t) -c_{KL}\gamma^tlog\frac{\pi_i(a_t|s_t)}{\pi_0(a_t|s_t)}-c_{Ent}\gamma^tlog\pi(a_t|s_t)\right]\\
&=\sum_i\mathbb{E}_{\pi_i}\left[\sum_{t\ge 0}\gamma^tR_i(s_t,a_t) +\frac{\gamma^t\alpha}{\beta}log\frac{\pi_i(a_t|s_t)}{\pi_0(a_t|s_t)}-\frac{\gamma^t}{\beta}log\pi(a_t|s_t)\right], \tag{1}
\end{align\*}
其中，$c_{KL},c_{Ent}\ge 0$是确定KL和entropy regularization大小的标量，这里的$\alpha = \frac{c_{KL}}{c_{KL}+c_{Ent}},\beta = \frac{1}{c_{KL}+c_{Ent}}，log\pi_0(a_t|s_t)$可以查看reward shaping鼓励大概率的action，而entropy项$-log\pi_i(a_t|s_t)$鼓励exploration。在这个公式中，所有任务的正则化项$c_{KL}$和$c_{Ent}$都是相同的，如果不同任务的reward scale是不同的，可以根据具体情况给每个任务都设定不同的系数。

### Soft Q-Learing 
这一节给出了表格形式的情况下如何去优化这个目标函数。这里使用了和EM算法类似的策略去优化这个目标函数，给定$\pi_0$优化$\pi_i$，给定$\pi_i$然后优化$\pi_0$。
当$\pi_0$固定的时候，式子(1)可以分解成每个任务的最大化问题，即优化每个任务的entropy　regularized expected return 和regularized reward $R'_i(s,a) = R_i(s,a) + \frac{\alpha}{\beta}log\pi_0(a|s)$，可以使用G-learning来优化。根据G-learning的证明，给定$\pi_0$，我们能得到以下的关系：
$$\pi_i(a_t|s_t) = \pi_0^{\alpha}(a_t|s_t)e^{\beta Q_i(a_t|s_t)-\beta V(s_t)} = \pi_0^{\alpha}(a_t|s_t)e^{\beta A_i(a_t|s_t)} \tag{2}$$
其中$hiA_i(s,a) = Q_i(s,a)-V_i(s)$是advantage function，$\pi_0$可以看成是一个policy prior，**需要注意的是这里多了一个指数$\alpha \lt 1$，这是多出来的entropy项的影响，soften了$\pi_0$对$\pi_i$的影响**。
V和S是一种新定义的state value和action value。使用推导出来的softened Bellman公式更新state value $V_i$和action value $Q_i(s,a)$的值：
$$V_i(s_t) = \frac{1}{\beta} log\sum_{a_t}\pi_0^{\alpha}(a_t|s_t)e^{\beta Q_i(s_t,a_t)} \tag{3}$$
$$Q_i(s_t,a_t) = R_i(s_t, a_t)+ \gamma \sum_{s_t}p_i(s_{t+1}|s_t,a_t)V_i(s_{t+1}) \tag{4}$$
这个Bellman update公式是softened的，因为state value $V_i$在actions上的max操作被温度$\beta$倒数上的soft-max操作代替了，当$\beta\rightarrow\infty$时，就变成了max 操作，这里有些不明白。为什么呢？这个我不理解有什么关系，这是这篇文章给出的解释。
**按照我的理解，这个和我们平常使用Bellman 期望公式或者最优等式没有什么关系，只是给了一种新的更新Q值和V值的方法。实际上，这两个公式都是根据推导给出的定义。**
还有一点：$\pi_0$是学出来的，而不是手动选出来的。式子(1)中和$\pi_0$相关的只有：
$$\frac{\alpha}{\beta}\sum_i\mathbb{E_{\pi_i}}\left[\sum_{t\ge 0}\gamma^tlog\pi_0(a_t|s_t) \right]\tag{5}$$
可以看出来，这是使用$\pi_0$去拟合一个混合的带折扣因子$\gamma$的state-action分布，可以使用最大似然估计来求解，如果是非表格情况的话，可以使用stochastic gradient ascent进行优化，但是需要注意的是本文中作者使用的目标函数多了一个KL散度。另一个区别是本文的distilled policy可以作为下一步要优化的task policy的反馈。
这里考虑以下为什么要多加一个entropy regularization。假设如果没有正则化项的话，也就是式子(2)中的$\alpha = 1$，这里考虑以下$n=1$时的例子，这样子式子(5)在$\pi_0=\pi_1$的时候最大，KL散度为$0$，这样子就目标函数就退化成了一个没有正则化项的expected return，最终策略$\pi_1$会收敛到一个局部最优值。**和TRPO的一个比较？？？未完待续。。。。**
如果$\alpha\lt 1$，即式(1)中有一个额外的entropy项。这样即使$\pi_0=\pi_1$，$KL(\pi_1||\pi_0)=0$，式子(1)也无法通过greedy策略进行最大化，式子(1)这时候就变成了entropy正则化的expected return，正则化系数$\beta'=\frac{\beta}{1-\alpha} = \frac{1}{c_{Ent}}$，所以最优的策略就是在$\beta'$处的Boltzmann policy。通过添加这个entropy项，可以保证策略不是greedy的，可以通过调整$c_{Ent}$的大小来调整exploration。
最开始的时候，exploration是在multitask任务上加的，如果有多个任务，一个很简单，而其他的很复杂，如果先遇到了简单任务，没有加entropy的话，最后就会收敛到最简单任务的greedy策略，这样子就无法充分探索其他任务的，导致陷入到次优解。对于single-task的RL来说，在A3C中提出用entropy取应对过早的收敛，作者在这里推广到了multitask任务上。

### Policy Gradient and a Better Parameterization
上面一节讲的是表格形式的计算，因为这时候，我们可以首先求解出V和Q，然后给定$\pi_0$，就可以写出$\pi_i$的解析解了。但是如果我们用神经网络等函数去拟合V和Q，那么就无法求解出V和Q了，这里使用梯度下降同时优化task polices和distilled policy。这种情况下，$\pi_i$的梯度更新通过求带有entropy正则化的return即可求出来，并且可以放在如actor-critic之类的框架中。
每一个$\pi_i$都用一个单独的网络表示，$\pi_0$也用一个网络表示。考虑到式子(2)中的optimal Boltzmann policy，用$\theta_0$表示$\pi_0$的参数，相应的policy如下：
$$\hat{\pi}(a_t|s_t) = \frac{e^{(h_{\theta_0}(a_t|s_t)}}{\sum_{a'}e^{h_{\theta_0}(a'|s_t)}} \tag{6}$$
这里作者没有分别去估计V和Q的值，而是使用参数为$\theta_i$的神经网络，直接估计A=Q-V的值：
$$\hat{A}_i(a_t|s_t) = f_{\theta_i}(a_t|s_t) - \frac{1}{\beta}log\sum_a\hat{\pi}_0^{\alpha}(a|s_t)e^{\beta f_{\theta_i}(a|s_t)} \tag{7}$$
第$i$个任务的policy可以参数化为：
$$\hat{\pi}_i(a_t|s_t) = \hat{\pi}_0^{\alpha}(a_t|s_t)e^{(\beta\hat{A}_i(a_t|s_t))}=\frac{e^{(\alpha h_{\theta_0}(a_t|s_t) + \beta f_{\theta_i}(a_t|s_t))}}{\sum_a'e^{(\alpha h_{\theta_0}(a'|s_t) + \beta f_{\theta_i}(a'|s_t))}} \tag{8}$$
这可以看成policy的一个两列架构，一列是提取的policy，一列是应用到task i上需要的一些调整。
给定上面$\pi_0, \pi_i$的参数化，我们可以推导策略梯度：
\begin{align\*}
\nabla_{\theta_i}J& = \mathbb_{\hat{\pi}_i}\left[\left(\sum_{t\gt 1} \nabla_{\theta_i}log\hat{\pi}_i(a_t|s_t)\right)\left(\sum_{u\ge 1}\gamma^u \left(R^{reg}_i(a_u,s_u\right)\right) \right]\\
& = \mathbb_{\hat{\pi}_i}\left[\sum_{t\gt 1} \nabla_{\theta_i}log\hat{\pi}_i(a_t|s_t)\left(\sum_{u\ge 1}\gamma^u \left(R^{reg}_i(a_u,s_u)\right)\right) \right] \tag{9}\\
\end{align\*}
其中$R_i^{reg}(s,a) = R_i(s,a) + \frac{\alpha}{\beta}log\hat{\pi}_0(a|s) - \frac{1}{\beta}log\hat{\pi}_i(a|s)$是正则化后的reward，注意，这里$\mathbb{E}_{\hat{\pi}_i}\left[\nabla_{\theta_i}log\hat{\pi}_i(a_t|s_t)\right] = 0$，因为log-derivative trick。如果有一个value baseline，那么为了减少梯度的方差，可以从正则化后的returns中减去它。
关于$\theta_0$的梯度如下：
\begin{align\*}
\nabla_{\theta_0}J& = \mathbb_{\hat{\pi}_i}\left[\sum_{t\gt 1} \nabla_{\theta_i}log\hat{\pi}_i(a_t|s_t)\left(\sum_{u\ge 1}\gamma^u \left(R^{reg}_i(a_u,s_u)\right)\right) \right]\\
& + \frac{\alpha}{\beta}\sum_i\mathbb{E}_{\hat{\pi}_i}\left[\sum_{t\ge 1}\gamma^t\sum_{a'_t}\left(\hat{\pi}_i(a'_t|s_t)-\hat{\pi}_0(a'_t}s_t)\right)\nabla_{\theta_0}h_{\theta_0}(a'_t|s_t)\right]\tag{10}
\end{align\*}
其中的第一项和$\pi_i$一样，这里多了第二项，第二项是尽量匹配$\hat{\pi}_i,\hat{\pi}_0$的概率，如果不使用KL散度的话，这里就不会有第二项了。KL正则是为了让$\pi_0$在$\pi_i$的质心上，即$\hat{\pi}_0(a'_t|s_t) = \frac{1}{n}\sum_i\hat{\pi}_i(a'_t|s_t)$，最后第二项就为$0$了，可以快速的将公共信息迁移到新任务上。
和ADMM,EASGD等在参数空间上进行优化不同的是，Distral是在策略空间上进行优化，这样子在语义上更有意义，对于稳定学习过程很重要。
本文的方法通过添加了entropy正则化和KL正则化，使得算法可以分开控制每个任务迁移的信息大小和exploration程序。

## 算法
上面给出的框架可以对不同的目标函数，算法和架构进行组合，然后生成一系列算法实例。
- KL散度和entropy：当$\alpha=0$时，只有entorpy，没有耦合，在不同任务中进行迁移，当$\alpha=1$时，只有KL散度，耦合，在不同任务中进行迁移，但是如果$\pi_i,\pi_0$很相似的话，会过早的停止探索。当$0\lt \alpha \lt 1$时，KL散度和entropy都有。
- 迭代优化还是联合优化：可以选择同时优化$\pi_0,\pi_i$，也可以固定其中一个，优化另一个。迭代优化和actor-mimic以及policy-distilled有一些相似，但是Distral是迭代进行的，$\pi_0$会对$\pi_i$的优化提供反馈。尽管迭代优化可能会很慢，但是从actor-mimic等的结果来看，可能它会更稳定。
- Separate还是two-column参数化：这里的意思是$\pi_i$是否使用式子(8)中的$\pi_0$，如果用的话，$\pi_0$中提取到的信息可以立刻用到$\pi_i$上，transfer可以更快。但是如果transfer的太快的话，可能会抑制在单个任务上exploration的有效性。。

这里作者给出了使用到的一些算法组合，如下表和下图所示。这里作者和三个A3C baseline(三种架构)做了比较，作者做实验的时候，试了两种A3C，第一个是原始的A3C，第二个是A3C的变种，最后发现这两种A3C没啥差别，在实验部分就选择了原始的A3C作比较。

![figure2](/figure2.png)
![table](/table.png)
### Algorithm 
- A3C: 在每个任务上单独使用A3C训练的policy
- A3C_multitask: 使用A3C同时在所有任务上训练得到的policy
- A3C_2col: 使用了式子(8)中的two-column架构A3C在每个任务上训练的policy
- KL_1col: $\pi_0,\pi_i$分别用一个网络来表示，令$\alpha=1$，即只有KL散度的式子(1)进行优化，
- KL+ent_1col: 和KL_1col一样，只不过包括了KL散度和entropy项，并设置$\alpha = 0.5$。
- KL_2col: 和KL_1col一样，但是使用了式子(8)中的two-column架构
- KL+ent_2col: 和KL+ent_1col一样，只是使用了two-column架构。

## 实验
总共有两个实验，第一个是在grid world上使用soft Q-learning和policy distilltion的迭代优化，第二个是七个算法在三个3D部分可观测环境上的评估。
### 环境
#### Grid world
这个实验是在一些简单的grid world上进行的，每一个任务通过一个随机选择的goal location进行区分。
每一个MDP的state由map location, previous action和previous reward组成。一个Distral智能体通过KL正则化的目标函数进行训练，优化算法在Soft Q-learing和policy distilltion之间进行迭代。每次soft Q-learing 的展开长度是$10$。

#### 3D环境 
这个实验使用了三个第一人称的$3D$环境。所有的智能体都是用pytorch/tensorflow实现的，每个任务有$32$个workers，使用异步的RMSProp进行学习。每个网络由CNN和LSTM组成，在不同的算法和实验中都是相同的。作者尝试了三个$\beta$和三个学习率$\epsilon$，每一组超参数跑了四次，其他超参数和单任务的A3C都一样的，对于KL+ent 1col和KL+ent 2col算法，$\alpha$被固定为$0.5$。
##### Maze
八个任务，每个任务都是一个随机放置reward和goal的迷宫。作者给出了$7$个算法的学习曲线，每一个学习曲线是选出最好的$\beta,\epsilon$在$8$个任务跑$4$次的平均值。Distral学习的很快，并且超过了三个A3C baselies，而two-column算法比one-column学习的要快，不带entropy的Distral要比带entropy学得快，但是最终得分要低，这可能是没有充分explration的原因。
multitask A3C和two-column A3C学习的不稳定，有时候学的好，有时候学的不好，有时候刚开始就不好了。而Distral对于超参数也很鲁棒。
##### Navigation
四个任务，比Maze难度要大。
##### Laser-tag
DeepMind Lab中的八个任务，最好的baseline是单独在每个任务上训练的A3C。


## Discussion
有两个idea这里需要强调一下。在优化过程中，使用KL散度正则化使$\pi_i$向$\pi_0$移动，使用$\pi_0$正则化$\pi_i$。
另一个就是在深度神经网络中，它们的参数没有意义，所以作者不是在参数空间进行的正则化，而是在策略空间进行正则化，这样子更有语义意义。

## 参考文献
1.https://papers.nips.cc/paper/7036-distral-robust-multitask-reinforcement-learning.pdf
2.

