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
multitask learning的目的是在不同taskes之间共享网络参数，在不同的task之间迁移可以提高效率。但是这种方法有几个缺点：
- 不同taskes的gradient可能有负干扰，让学习变得不稳定，更低效。
- 另一个问题是不同taskes的reward scheme不同，其中的某一个reward可能占主导地位。

本文提出Distral方法，在不同taskes的worker之间共享一个distilled policy而不是共享参数，学习不同taskes的common behaviours。每一个worker解决它自己的task，但是需要限制每一个单独的policy离distilled policy足够近，而distilled policy需要通过所有的single policies得到，它在所有policies的质心上。通过优化一个联合目标函数，实现这两个过程。

## Introduction  
由于DRL需要的训练时间和训练数据很多，现在的DRL问题开始转向单个智能体同时或者连续的解决多个相关问题。由于巨大的计算开销，这个方向的研究需要设计出非常鲁棒的，与具体任务无关的算法。直观上来说，相似的taskes之间有共同的结构，所以我们觉得一起训练它们应该能够促进学习，然而实践表明并不是这样。
所以，multitask和transfer learning需要解决一个问题：在多个taskes上训练会对单个任务的训练产生负面影响。这个问题可能的原因有：其他任务的梯度可能会被当做噪音干扰学习，极端情况下，其中一个任务可能会主导其他的任务。
这篇论文作者提出了一种multitask和transfer RL算法，它能够高效的在多个taskes之间共享behaviour structure。除了在grid world领域的一些指导性illustration(例证)，作者还在DeepMind Lab 3D环境中详细分析了算法，并且和A3C baseline的比较。作者验证了Distral算法学习的很快，而且能够达到很好的收敛性能，而且对超参数很鲁棒，比multitask A3C baselines要稳定的多。

## Distral: Distill and Transfer Learning
作者给出了一个multi-taskes的框架Distral，如下图所示，作者给出了从四个taskes中提取shared policy的一个例子。该方法用一个shared policy去提取task-specific的polices之间的common behaviour和representation，然后又用这个shared policy和task-specific polices之间的KL散度正则化task-specific polices。KL散度的作用相当于shaping reward，鼓励exploration。最后，这些taskes之间的common knowledge都被distilled到shared policy中了，然后可以迁移到其他任务中去。
![figure1](/figure1.png)

### 数学框架
一个multitask RL setting中有$n$个任务，折扣因子为$\gamma$，它们的state space和action space是相同的，但是每个任务$i$的状态转换概率$p\_i(s'|s,a)$和奖励函数$R\_i(s,a)$是不同的，用$\pi\_i$表示第$i$个任务的stochastic polices。给定从一些初始状态开始的state和action联合分布的轨迹，用$\pi\_i$表示dynamics和polices。
作者通过优化一个expected return和policy regularization组成的目标函数将学习不同任务的policy联系起来。用$\pi\_0$表示要提取的shared policy，然后通过使用$\pi\_0$和$\pi\_i$的KL散度$\mathbb{E}\_{\pi\_i}\left[\sum\_{t\ge 0}\gamma^t\log\frac{\pi\_i(a\_t|s\_t)}{\pi\_0(s\_t|a\_t)}\right]$对$\pi\_i$进行约束，使所有的策略 $\pi\_i$ 接近 $\pi\_0$ 。此外，作者还使用了一个带折扣因子的entropy正则化项鼓励exploration。系统越混乱，entropy越大，所以exploration越多，采取的动作越随机，entropy就越大。为什么，举个例子，系统采取两个动作的概率分别是：[0, 1], [0.5, 0.5], [0.25, 0.75]，他们对应的熵分别是$0, \log 2, \log 4- 0.75 \log 3 < \log 2$，也就是系统最混乱的时候，熵最大，为了鼓励探索，也就是让其他action出现的概率变大，也就是让熵变大。
最后总的优化目标就变成了：
\begin{align\*}
J(\pi\_0, \{\pi\_i\}_{i=1}^n) &=\sum\_i\mathbb{E}\_{\pi\_i}\left[\sum\_{t\ge 0}\gamma^tR\_i(s\_t,a\_t) -c\_{KL}\gamma^t \log\frac{\pi\_i(a\_t|s\_t)}{\pi\_0(a\_t|s\_t)}-c\_{Ent}\gamma^t \log\pi\_i(a\_t|s\_t)\right]\\\\
&=\sum\_i\mathbb{E}\_{\pi\_i}\left[\sum\_{t\ge 0}\gamma^tR\_i(s\_t,a\_t) - c\_{KL}\gamma^t\log{\pi\_i(a\_t|s\_t)} + c\_{KL}\gamma^t\log{\pi\_0(a\_t|s\_t)} - c\_{Ent}\gamma^t\log\pi\_i(a\_t|s\_t)\right]\\\\
&=\sum\_i\mathbb{E}\_{\pi\_i}\left[\sum\_{t\ge 0}\gamma^tR\_i(s\_t,a\_t) + c\_{KL}\gamma^t\log{\pi\_0(a\_t|s\_t)} - (c\_{Ent}\gamma^t + c\_{KL}\gamma^t)\log\pi\_i(a\_t|s\_t)\right]\\\\
&=\sum\_i\mathbb{E}\_{\pi\_i}\left[\sum\_{t\ge 0}\gamma^tR\_i(s\_t,a\_t) +\frac{\gamma^t \alpha}{\beta}\log{\pi\_0(a\_t|s\_t)}-\frac{\gamma^t}{\beta}\log\pi\_i(a\_t|s\_t)\right], \tag{1}
\end{align\*}
其中$c\_{KL},c\_{Ent}\ge 0$是控制KL散度正则化项和entropy正则化项大小的超参数，$\alpha = \frac{c\_{KL}}{c\_{KL}+c\_{Ent}},\beta = \frac{1}{c\_{KL}+c\_{Ent}}。变换后的公式$(1)中出现的\log\pi\_0(a\_t|s\_t)$可以看成reward shaping，鼓励大概率的action；而entropy项$-\log\pi\_i(a\_t|s\_t)$鼓励exploration。在这个公式中，设置所有任务的正则化系数$c\_{KL}$和$c\_{Ent}$都是相同的，如果不同任务的reward scale不同，可以根据具体情况给相应任务设定相应系数。
让$R'\_i(s,a) = R\_i(s,a) + \frac{\alpha}{\beta}\log\pi\_0(a|s)$，目标函数可以看成new reward $R'$的正则化约束问题。


### Soft Q-Learing 
有很多方法可以最大化上面给出的目标函数，这一节介绍表格形式下，如何使用和EM算法类似的策略优化目标函数－－固定$\pi\_0$优化$\pi\_i$，固定$\pi\_i$然后优化$\pi\_0$。
当$\pi\_0$固定的时候，式子(1)可以分解成每个task的最大化问题，即优化每个任务的entropy正则化return，return使用的是正则化reward：
正则化后的return可以使用G-learning来优化（这里说的应该是原来的return和R什么都没，这里都加上了正则化）。给定$\pi\_0$，根据Soft Q-Learning(G-learning)的证明，我们能得到以下的关系：
$$\pi\_i(a\_t|s\_t) = \pi\_0^{\alpha} (a\_t|s\_t)e^{\beta Q\_i(a\_t|s\_t)-\beta V(s\_t)} = \pi\_0^{\alpha} (a\_t|s\_t)e^{\beta A\_i(a\_t|s\_t)} \tag{2}$$
其中$A\_i(s,a) = Q\_i(s,a)-V\_i(s)$是advantage function，$\pi\_0$可以看成是一个policy prior，**需要注意的是这里多了一个指数$\alpha \lt 1$，这是多出来的entropy项的影响，soften了$\pi\_0$对$\pi\_i$的影响**。$V$和$Q$是新定义的一种state value和action value，使用推导的softened Bellman公式更新：
$$V\_i(s\_t) = \frac{1}{\beta} \log\sum\_{a\_t}\pi\_0^{\alpha} (a\_t|s\_t)e^{\beta Q\_i(s\_t,a\_t)} \tag{3}$$
$$Q\_i(s\_t,a\_t) = R\_i(s\_t, a\_t)+ \gamma \sum\_{s\_t}p\_i(s\_{t+1}|s\_t,a\_t)V\_i(s\_{t+1}) \tag{4}$$
这个Bellman update公式是softened的，因为state value $V\_i$在actions上的max操作被温度$\beta$倒数上的soft-max操作代替了，当$\beta\rightarrow\infty$时，就变成了max 操作，这里有些不明白。为什么呢？这个我不理解有什么关系，这是这篇文章给出的解释。**按照我的理解，这个和我们平常使用Bellman 期望公式或者最优等式没有什么关系，只是给了一种新的更新Q值和V值的方法。实际上，这两个公式都是根据推导给出的定义。**
还有一点：$\pi\_0$是学出来的，而不是手动选出来的。式子(1)中和$\pi\_0$相关的只有：
$$\frac{\alpha}{\beta}\sum\_i\mathbb{E\_{\pi\_i}}\left[\sum\_{t\ge 0}\gamma^t\log\pi\_0(a\_t|s\_t) \right]\tag{5}$$
可以看出来，这是使用$\pi\_0$去拟合一个混合的带折扣因子$\gamma$的state-action分布，每个$i$代表一个任务，可以使用最大似然估计来求解，如果是非表格情况的话，可以使用stochastic gradient ascent进行优化，但是需要注意的是本文中作者使用的目标函数多了一个KL散度。另一个区别是本文的distilled policy可以作为下一步要优化的task policy的反馈。
多加一个entropy正则项的意义？如果不加entropy正则化，也就是式子$(2)$中的$\alpha = 1$，考虑$n=1$时的例子，式子$(5)$在$\pi\_0=\pi\_1$的时候最大，KL散度为$0$，目标函数退化成了一个没有正则化项的expected return，最终策略$\pi\_1$会收敛到一个局部最优值。**和TRPO的一个比较？？？未完待续。。。。**如果$\alpha\lt 1$，式(1)中有一个额外的entropy项。这样即使$\pi\_0=\pi\_1$，$KL(\pi\_1||\pi\_0)=0$，因为有entropy项，也无法通过greedy策略最大化式子$(1)$。式子$1$的entropy正则化系数变成了$\beta'=\frac{\beta}{1-\alpha} = \frac{1}{c\_{Ent}}$（第一个等号是为什么？？是因为$\pi\_0=\pi\_1$，然后就可以将$\pi\_0,\pi\_i$的系数合并了），最优的策略就是$\beta'$处的Boltzmann policy。添加这个entropy项可以保证策略不是greedy的，通过调整$c\_{Ent}$的大小可以调整exploration。
最开始的时候，exploration是在multitask任务上加的，如果有多个任务，一个很简单，而其他的很复杂，如果先遇到了简单任务，没有加entropy的话，最后就会收敛到最简单任务的greedy策略，这样子就无法充分探索其他任务的，导致陷入到次优解。对于single-task的RL来说，在A3C中提出用entropy取应对过早的收敛，作者在这里推广到了multitask任务上。

### Policy Gradient and a Better Parameterization
上面一节讲的是表格形式的计算，给定$\pi\_0$，首先求解出$\pi$对应的$V$和$Q$，然后写出$\pi\_i$的解析。但是如果我们用神经网络等函数去拟合$V$和$Q$，$V$和$Q$的求解特别慢，这里使用梯度下降同时优化task polices和distilled policy。这种情况下，$\pi\_i$的梯度更新通过求带有entropy正则化的return即可求出来，并且可以放在如actor-critic之类的框架中。
每一个$\pi\_i$都用一个单独的网络表示，$\pi\_0$也用一个单独的网络表示，用$\theta\_0$表示$\pi\_0$的参数，对应的policy表示为：
$$\hat{\pi\_0}(a\_t|s\_t) = \frac{e^{(h\_{\theta\_0}(a\_t|s\_t))} }{\sum\_{a'}e^{h\_{\theta\_0}(a'|s\_t)}} \tag{6}$$
使用参数为$\theta\_i$的神经网络表示$Q$值，用$f\_{\theta\_i}$表示第$i$个策略$\pi$的$Q$值，用$Q$表示$V$，再估计$A=Q-V$的值：
$$\hat{A}\_i(a\_t|s\_t) = f\_{\theta\_i}(a\_t|s\_t) - \frac{1}{\beta}\log\sum\_a\hat{\pi}\_0^{\alpha} (a|s\_t)e^{\beta f\_{\theta\_i}(a|s\_t)} \tag{7}$$
将式子$(7)$代入式子$(2)$得第$i$个任务的policy可以参数化为：
\begin{align\*}
\hat{\pi}\_i(a\_t|s\_t) 
& = \hat{\pi}\_0\^{\alpha} (a\_t|s\_t)e\^{\left(\beta \hat{Q}\_i(a\_t|s\_t)-\beta \hat{V}(s\_t)\right)}\\\\
& = \hat{\pi}\_0\^{\alpha} (a\_t|s\_t)e\^{\left(\beta \hat{A}\_i(a\_t|s\_t)\right)}\\\\
& = \hat{\pi}\_0\^{\alpha} (a\_t|s\_t)e\^{\left(\beta \left(f\_{\theta\_i}(a\_t|s\_t) - \frac{1}{\beta}\log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}\right)\right)}\\\\
& = \hat{\pi}\_0\^{\alpha} (a\_t|s\_t)e\^{\left(\beta f\_{\theta\_i}(a\_t|s\_t) - \log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}\right)}\\\\ 
& = \left(\frac{e\^{(h\_{\theta\_0}(a\_t|s\_t))} }{\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}}\right)\^{\alpha}e\^{\left(\beta f\_{\theta\_i}(a\_t|s\_t) - \log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}\right)}\\\\ 
& = \left(\frac{e\^{(h\_{\theta\_0}(a\_t|s\_t))} }{\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}}\right)\^{\alpha} 
      \cdot 
       \frac{e\^{\beta f\_{\theta\_i}(a\_t|s\_t)} }   {e\^{\log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}}}\\\\
& = \frac{\left(e\^{(h\_{\theta\_0}(a\_t|s\_t))}\right)\^{\alpha}}  {\left(\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}\right)\^{\alpha}} 
      \cdot 
         \frac{e\^{\beta f\_{\theta\_i}(a\_t|s\_t)}}   {e\^{\log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}}}\\\\ 
& = \frac{e\^{\alpha \cdot(h\_{\theta\_0}(a\_t|s\_t))}}   {\left(\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}\right)\^{\alpha}} 
      \cdot 
         \frac{e\^{\beta f\_{\theta\_i}(a\_t|s\_t)}}{e\^{\log\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}}}\\\\ 
& = \frac{e\^{(\alpha h\_{\theta\_0}(a\_t|s\_t))}}  {\left(\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}\right)\^{\alpha}} 
      \cdot 
        \frac{e\^{\beta f\_{\theta\_i}(a\_t|s\_t)}}{\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t)e\^{\beta f\_{\theta\_i}(a|s\_t)}}\\\\
& = \frac{e\^{(\alpha h\_{\theta\_0}(a\_t|s\_t))} \cdot e\^{\beta f\_{\theta\_i}(a\_t|s\_t) }}
         {\left(\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}\right)\^{\alpha} \cdot {\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t) e\^{\beta f\_{\theta\_i}(a|s\_t)}}}\\\\
& = \frac{e\^{(\alpha h\_{\theta\_0}(a\_t|s\_t) + \beta f\_{\theta\_i}(a\_t|s\_t)) }}
         {\left(\sum\_{a'}e\^{h\_{\theta\_0}(a'|s\_t)}\right)\^{\alpha} \cdot {\sum\_a\hat{\pi}\_0\^{\alpha}(a|s\_t) e\^{\beta f\_{\theta\_i}(a|s\_t)}}}(Why to blow equation???)\\\\
& = \frac{e\^{(\alpha h\_{\theta\_0}(a\_t|s\_t) + \beta f\_{\theta\_i}(a\_t|s\_t))}}  {\sum\_{a'}e\^{(\alpha h\_{\theta\_0}(a'|s\_t) + \beta f\_{\theta\_i}(a'|s\_t))}} 
\end{align\*}
所以：
$$\hat{\pi}\_i(a\_t|s\_t) = \hat{\pi}\_0\^{\alpha}(a\_t|s\_t)e\^{(\beta\hat{A}\_i(a\_t|s\_t))}=\frac{e\^{(\alpha h\_{\theta\_0}(a\_t|s\_t) + \beta f\_{\theta\_i}(a\_t|s\_t))}}{\sum\_{a'}e\^{(\alpha h\_{\theta\_0}(a'|s\_t) + \beta f\_{\theta\_i}(a'|s\_t))}} \tag{8}$$
这可以看成policy的一个两列架构，一列是提取的shared policy，一列是将$\pi\_0$应用到task $i$上需要做的一些修改。
使用参数化的$\pi\_0, \pi\_i$，首先推导策略相对于$\pi\_i$的梯度（policy gradient的推导，这里是直接应用了)：
\begin{align\*}
\nabla\_{\theta\_i}J& = \mathbb{E}\_{\hat{\pi}\_i}\left[\left(\sum\_{t\gt 1} \nabla\_{\theta\_i}\log{\hat{\pi}}\_i(a\_t|s\_t)\right) \left(\sum\_{u\ge 1}\gamma\^u \left(R\^{reg}\_i(a\_u,s\_u)\right)\right) \right]\\\\
& = \mathbb{E}\_{\hat{\pi}\_i}\left[\sum\_{t\gt 1} \nabla\_{\theta\_i}\log\hat{\pi}\_i(a\_t|s\_t)\left(\sum\_{u\ge t}\gamma\^u \left(R\^{reg}\_i(a\_u,s\_u)\right)\right) \right] \tag{9}\\\\
\end{align\*}
其中$R\_i\^{reg}(s,a) = R\_i(s,a) + \frac{\alpha}{\beta}\log\hat{\pi}\_0(a|s) - \frac{1}{\beta}\log\hat{\pi}\_i(a|s)$是正则化后的reward。注意，这里$\mathbb{E}\_{\hat{\pi}\_i}\left[\nabla\_{\theta\_i}\log\hat{\pi}\_i(a\_t|s\_t)\right] = 0$，因为\log-derivative trick。如果有一个value baseline，那么为了减少梯度的方差，可以从正则化后的returns中减去它。
关于$\theta\_0$的梯度如下：
\begin{align\*}
\nabla\_{\theta\_0}J
& = \mathbb{E}\_{\hat{\pi}\_i}
\left[
    \sum\_{t\gt 1} \nabla\_{\theta\_i}\log\hat{\pi}\_i(a\_t|s\_t) 
        \left(\sum\_{u\ge 1}\gamma\^u 
             \left(
                R\^{reg}\_i(a\_u,s\_u)
             \right)
        \right)
\right]\\\\
& \qquad +\frac{\alpha}{\beta}\sum\_i\mathbb{E}\_{\hat{\pi}\_i}
\left[
    \sum\_{t\ge 1}\gamma\^t\sum\_{a'\_t}
        \left(
             \hat{\pi}\_i(a'\_t|s\_t)-\hat{\pi}\_0(a'\_t|s\_t)
        \right)
            \nabla\_{\theta\_0}h\_{\theta\_0}(a'\_t|s\_t)
\right] \tag{10}
\end{align\*}
第一项和$\pi\_i$一样，第二项是让$\hat{\pi}\_i,\hat{\pi}\_0$的概率尽可能接近。如果不使用KL散度的话，这里就不会有第二项了。KL正则是为了让$\pi\_0$在$\pi\_i$的质心上，即$\hat{\pi}\_0(a'\_t|s\_t) = \frac{1}{n}\sum\_i\hat{\pi}\_i(a'\_t|s\_t)$，最后第二项就为$0$了，可以快速的将公共信息迁移到新任务上。
和ADMM,EASGD等在参数空间上进行优化不同的是，Distral是在策略空间上进行优化，这样子在语义上更有意义，对于稳定学习过程很重要。
本文的方法通过添加了entropy正则化和KL正则化，使得算法可以分开控制每个任务迁移的信息大小和exploration程序。

## 算法
上面给出的框架可以对不同的目标函数，算法和架构进行组合，然后生成一系列算法实例。
- KL散度和entropy：当$\alpha=0$时，只有entorpy，不同任务之间没有耦合，在不同任务中进行迁移。当$\alpha=1$时，只有KL散度，不同任务之间有耦合，在不同任务中进行迁移，但是如果$\pi\_i,\pi\_0$很像的话，会过早的停止探索。当$0\lt \alpha \lt 1$时，KL散度和entropy都有。
- 迭代优化还是联合优化：可以选择同时优化$\pi\_0,\pi\_i$，也可以固定其中一个，优化另一个。迭代优化和actor-mimic以及policy-distilled有一些相似，但是Distral是迭代进行的，$\pi\_0$会对$\pi\_i$的优化提供反馈。尽管迭代优化可能会很慢，但是从actor-mimic等的结果来看，可能它会更稳定。
- Separate还是two-column参数化：这里的意思是$\pi\_i$是否使用式子(8)中的$\pi\_0$，如果用的话，$\pi\_0$中提取到的信息可以立刻用到$\pi\_i$上，transfer可以更快。但是如果transfer的太快的话，可能会抑制在单个任务上exploration的有效性。。

这里作者给出了使用到的一些算法组合，如下表和下图所示。这里作者和三个A3C baseline(三种架构)做了比较，作者做实验的时候，试了两种A3C，第一个是原始的A3C，第二个是A3C的变种，最后发现这两种A3C没啥差别，在实验部分就选择了原始的A3C作比较。

![figure2](/figure2.png)
![table](/table.png)
### Algorithm 
- A3C: 在每个任务上单独使用A3C训练的policy
- A3C\_multitask: 使用A3C同时在所有任务上训练得到的policy
- A3C\_2col: 使用了式子(8)中的two-column架构A3C在每个任务上训练的policy
- KL\_1col: $\pi\_0,\pi\_i$分别用一个网络来表示，令$\alpha=1$，即只有KL散度的式子(1)进行优化，
- KL+ent\_1col: 和KL\_1col一样，只不过包括了KL散度和entropy项，并设置$\alpha = 0.5$。
- KL\_2col: 和KL\_1col一样，但是使用了式子(8)中的two-column架构
- KL+ent\_2col: 和KL+ent\_1col一样，只是使用了two-column架构。

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
有两个idea这里需要强调一下。在优化过程中，使用KL散度正则化使$\pi\_i$向$\pi\_0$移动，使用$\pi\_0$正则化$\pi\_i$。
另一个就是在深度神经网络中，它们的参数没有意义，所以作者不是在参数空间进行的正则化，而是在策略空间进行正则化，这样子更有语义意义。

## 参考文献
1.https://papers.nips.cc/paper/7036-distral-robust-multitask-reinforcement-learning.pdf
