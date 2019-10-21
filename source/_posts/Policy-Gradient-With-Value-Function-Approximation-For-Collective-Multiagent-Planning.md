---
title: Policy Gradient With Value Function Approximation For Collective Multiagent Planning
date: 2019-01-26 19:33:50
tags:
 - 强化学习
 - actor-critic
 - 深度学习
 - 论文
categories: 强化学习
mathjax: true
---

## 摘要
分布式的部分可观测马尔科夫决策过程(Dec POMDP)为解决多智能体系统中的序列决策问题提供了一个框架。考虑到POMDP的计算复杂度，最近的研究主要集中在Dec-POMDP中一些易于处理但是比较实用的子问题。本文解决的就是其中的一个子问题叫做CDec-POMDP其中一系列智能体的共同行为影响了它们公共的reward和环境变化。本文的主要贡献是提出了一个actor-critic(AC)强化学习算法优化CDec-POMDP问题的policy。普通的AC算法对于大型问题收敛的很慢，为了解决这个问题，本文展示了如何将智能体的估计动作值函数进行分解从而产生有效的更新以及推导出一个基于局部奖励信号的新的critic训练方式。通过在一个合成的benchmark以及真实的出租车车队优化问题上和其他方法进行对比，结果表明本文的AC方法提供了比之前最好的方法还要高质量的方法。

## 引言 
近些年来，分布式的部分可观测马尔科夫决策过程已经发展成了解决多智能体协作的序列决策问题的一个很有前景的(promising)方法。Dec-POMDP对智能体基于环境和其他智能体的不同部分观测最大化一个全局的目标进行建模。Dec-POMDP的具体应用包括协调行星探测，多机器人协调控制以及无线网络的吞吐量优化。然而，解决分布式的部分马尔科夫决策过程是相当困难的，即使对于只有$2$个智能体的问题呢是NP难的。
为了增大规模和提高真实问题中的应用，过去的研究已经探索了智能体之间严格的交互，如状态转换和观测独立，事件驱动的的交互以及智能体之间的弱耦合性。最近，一系列工作开始关注于智能体的身份不影响它们之间的交互上，环境的变化主要受到智能体的共同影响，和著名的阻塞游戏很像。一些城市交通中的问题如出租车调度可以用这样的协同规划模型进行建模。
在本文中，作者着重于集中的Dec-POMDP框架将一类不确定情况下的集中多智能体序列决策问题形式化。Nguyen等人提出了一个采样方法优化CDec-POMDP模型中的policy。之前方法的一个主要缺点是policy是用表格形式展现的，随着智能体的observation spaces改变时，表格形式的policy不能很好的进行扩展。受到最近一些强化学习工作的启发，本文的贡献是一个AC框架的强化学习算法用来优化CDec-POMDP的policy。Policy用函数如神经网络来表示可以避免表格形式的policy的扩展性问题。我们推导出了策略梯度并且基于CDec-POMDP中智能体的交互提出了一个估计的因子动作值函数。普通的AC算法因为学习全局reward的原因，在解决大型多智能体系统问题时收敛的很慢。为了解决这个问题，本文提出了一种新的方式去训练critic，高效利用智能体的局部值函数的估计动作值函数。
我们在一个合成的多机器人导航领域和现实世界中一个亚洲城市的出租车调度问题上测试了本文的方法，结果展示了本文的方法可以扩展到大型多智能体系统上。根据经验，我们因式AC方法比以前最好的方法给出的解决方案都要好。因式AC方法收敛的也比普通的AC方法快很多，验证了我们提出的critic训练方法的有效性。
**相关工作** 我们的工作基于具有近似值函数的策略梯度框架。然而，根据以往的经验显示，直接应用原始的策略梯度到多智能体任务中，尤其是CDec-POMDP模型中会产生较高方差。在本文中，我们展示了一个和CDec-POMDP兼容的估计值函数，它能产生高效且低方差的策略梯度更新。Peshkin很早之前就研究过了应用于分布式policy的强化学习，Guestrin还提出使用REINFORCE从协调图中训练一个因子值函数的softmax策略。然而，这些以前的工作中，策略梯度都是从全局的经验回报而不是分解后的critic中估计的。我们在第四章中展示了一个分解后ciritc和基于训练这个critic得到的一个单个值函数对于高效的采样学习是很重要的。我们的实验结果表明了我们提出的critic训练方式比用全局经验回报训练收敛的还要快。

## 集中分布式POMDP模型
我们首先介绍一下Nguyen提出的CDec-POMDP模型。一个对应于这个模型的$T$步的动态贝叶斯网络如图所示。它由以下几个部分组成：
- 一个有限的计划范围$H$
- 智能的数量$M$，一个智能体m可能处在state space $S$中的任意一个状态，联合state space是$\times_{m=1}^MS$，我们用$i\in S$表示一个state。
- 每一个智能体m都有一个action spaceA，我们用$j\in A$表示一个action。
- 用$(s_{1:H},a_{1:H})^m=(s_1^m,a_1^m,\cdots,s_H^m,a_H^m)$表示一个智能体m完整的state-action轨迹。用随机变量$s_t^m,a_t^m$表示智能体$m$在$t$时刻的state和action。不同的指示函数$I_t(\cdot)$如表$1$所示。给定每一个智能体$m\in M$的轨迹，定义以下的计数方式：
$$n_t(i,j,i') = \sum_{m=1}^M I_t^m(i,j,i'),\forall i,i'\in S,j\in A.$$
如表$1$所示，计数器$n_t(i,j,i')$表示在$t$时刻处于state $i$，采取action $j$，转换到state $i'$的智能体数量。其他计数器$n_t(i)$和$n_t(i,j)$的定义类似。使用这些计数器，我们可以定义$t$时刻的计数表$\bf{n}_{s_t}$和$\bf{n}_{s_ta_t}$如表$1$所示。
- 我们假设一个普遍的部分观测环境，其中智能体基于其他智能体的总体影响可以有不同的ovservation。一个智能体观测到它的局部state $s_t^m$。此外在$t$时刻基于它的局部状态$s_t^m$和计数表$\bf{n}_{s_t}$观测到$o_t^m$。例如，一个智能体m在$t$时刻处于state $i$，可以观测到其他也处在state $i(=n_t(i))$的智能体或者其他处在state $i$临近状态$j$的智能体，即$n_t(j),\forall j\in Nb(i)$。
- 状态转换函数是$\Phi_t(s_{t+1}^m=i'|s_t^m=i,a_t^m=j,\bf{N}_{s_t})$。所有智能体的状态转换函数是一样的，注意它会受到$\bf{n}_{s_t}$的影响，而$\bf{n}_{s_t}$依赖于智能体的共同行为。
- 每一个智能体m有一个不平稳的policy $\pi_t^m(j|i,o_t^m(i,\bf{n}_{s_t}))$，表示在$t$时刻给定智能体m的observation $(i,o_t^m(i,\bf{n}_{s_t})$之后，智能体采取action $j$的概率。我们用$\pi^m=(\pi_1,\cdots,\pi_H)$表示智能体m水平范围的policy。
- 一个智能体接收到的reward $r_t^m=r_t(i,j,\bf{n}_{s_t}$取决于它的局部state和action，以及计数表$\bf{n}_{s_t}$。
- 初始的state分布，$b_o=(P(i)\forall i \in S)$，对于所有的智能体都是相同的。

我们在这里展示了最简单的版本，所有的智能体的类型都相同，并且有相似的state transition，observation和reward模型。模型也可以处理多种类型的智能体，不同类型的智能体有不同的变化。我们还可以引入一个不受智能体action影响的external state，如交通领域的出租车需求。我们的结果也可以扩展到解决类似的问题。
像CDec-POMDP之类的模型对于解决智能体数量很大或者智能体的身份不影响reward或者transition function之类的问题是很有用的。其中一个应用是出租车车队优化问题，这个问题是计算出出租车调度的policy使得车队的利润最大化。一个出租车的决策过程如下。在时刻$t$时，每个出租车观测到它当前的城市空间$z$，不同的空间构成了state space $S$，以及当前空间和它的相邻空间的其他出租车的计数和当前局部请求的一个估计。这构成了出租车基于计数的observation $o(\cdot)$。基于这个observation，出租车必须决定待在当前空间$z$寻找乘客还是移动到下一个空间。这些决策选择取决于不同的因子，如请求比率和当前空间其他出租车的计数。类似的，环境是随机的，在不同时间出租车请求是变化的。使用出租车车队的的GPS记录可以得到这些历史的请求数据。
**基于计数的统计数据用于规划** CDec-POMDP模型的一个关键属性是模型的变换取决于智能体的集中交互而不是智能体的身份。在出租车车队优化问题中，智能体数量可以相当大（大约有$8000$个智能体在现实世界的实验中）。给出这么大数量的智能体个数，为每一个智能体计算出独一无二的policy是不可能的。因此，和之前的工作类似，我们的目标是对所有智能体计算出一个相同的policy $\pi$。因为policy $\pi$取决于计数，它代表了一种富有表现力的policy。
对于一个固定的数量M来说，用$\{(s_{1:T},a_{1:T})^m\forall m\}$表示从图$1$的DBN网络中采样得到的不同智能体的state-action轨迹。用$\mathbf{n}_{1:T}=\{(\mathbf{n}_{s_t},\mathbf{n}_{s_ta_t},\mathbf{n}_{s_ta_ts_{t+1}})\forall t=1:T\}$表示每一个时间步$t$的结果计数表的组合向量。Nguyen等人展示了计数器$\mathbf{n}$中拥有足够的统计数据用来规划。也就是说，一个policy $\pi$在水平范围H内的联合值函数可以通过计数器的期望进行计算：
$$V(\pi) = \sum_{m=1}^M\sum_{T=1}^H E[r_T^m] = \sum_{\mathbf{n}\in \Omega_{1:H}}P(\mathbf{n};\pi) \left[\sum_{T=1}^H\sum_{i\in S,j\in A} n_T(i,j)r_T(i,j,\mathbf{n}_T)\right]$$
集合$\Omega_{1:H}$是所有允许的一致计数表的集合，如下所示：
$$\sum_{i\in S}n_T(i) = M \forall T;$$
$$\sum_{j\in A}n_T(i,j) = n_T(i) = \forall j \forall T;$$
$$\sum_{i'\in S}n_T(i,j,i') = n_T(i,j)\forall i\in S,\forall j \in A, \forall T;$$
$P(\mathbf{n},\pi)$是计数器的分布。这个结果的一个关键好处是我们可以直接从分布$P(\mathbf{n})$中对计数器$\mathbf{n}$采样而不是对单个不同智能体的轨迹$(s_{1:H},a_{1:H})进行采样来$评估policy $\pi$，这显著节省了计算开销。我们的目标是计算最优的policy $\pi$来最大化$V(\pi)$。我们假设一个集中式学习，分布式执行的强化学习设置。我们假设有一个模拟器可以从$P(\mathbf{n};\pi)$中提供计数器样本。

## CDec-POMDP的策略梯度
之前的工作提出了一个基于采样的EM算法来优化policy $\pi$。这个policy被表示成计数器$\mathbf{n}$空间中的一个线性分段表policy，其中每一个线性片段指定了下一个action的分布。然而，这种表格形式的表示限制了它的表达能力，因为片段的数量是固定的先验，并且每个范围都必须手动定义，这可能会对性能产生不利影响。此外，当observation o是多维的时候，即，一个智能体观测到它位置相邻区域的计数器时，需要指数多个片段。为了解决这个问题，我们的目标是优化函数形式（如神经网络）的policy。
我们首先扩展策略梯度理论到CDec-POMDP上，用$\theta$表示policy参数的向量。我们接下来展示如何计算$\Delta_\theta V(\pi)$。用$\mathbf{s}_t,\mathbf{a}_t$表示$t$时刻所有智能体的联合state和联合action。给定一个policy $\pi$，值函数表示形式如下：
$$V_t(\pi)=\sum_{\mathbf{s}_t,\mathbf{a}_t}P^{\pi}(\mathbf{s}_t,\mathbf{a}_t|b_o,\pi)Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_T)$$
其中$P^{\pi}(\mathbf{s}_t,\mathbf{a}_t|b_o)=\sum_{\mathbf{s}_{1:t-1},\mathbf{a}_{1:t-1}}P^{\pi}(\mathbf{s}_{1:t},\mathbf{a}_{1:t}|b_o)$是policy $\pi$下联合state $\mathbf{s}_t$，和联合action $\mathbf{a}_t$的分布。值函数$Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t)$的计算过程如下：
$$Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t) = r_t(\mathbf{s}_t,\mathbf{a}_t)+\sum_{\mathbf{s}_{t+1},\mathbf{a}_{t+1})}P^{\pi}(\mathbf{s}_{t+1},\mathbf{a}_{t+1}|\mathbf{s}_t,\mathbf{a}_t))Q_{t+1}^{\pi}(\mathbf{s}_{t+1},\mathbf{a}_{t+1})$$
接下来介绍以下CDec-POMDP的策略梯度理论：
**定理1.** 对于任何CDec-POMDP，策略梯度计算公式如下：
$$\Delta_{\theta}V_1(\pi)=\sum_{t=1}^HE_{\mathbf{s}_t,\mathbf{a}_t)|b_o,\pi}\left[Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t)\sum_{i\in S,j\in A}n_t(i,j)\Delta_{\theta}log\pi_{t}(j|i,o(i,\mathbf{n}_{s_t}))\right]$$
这个定理的证明和其他后续结果在附录中。
注意由于许多原因利用上述结果计算策略梯度是不切实际的。联合state-action $\mathbf{a}_t,\mathbf{s}_t$空间是组合的。考虑到智能体的个数可能有很多个，对每一个智能体的轨迹进行采样是计算上不可行的。为了补救，我们接下来会展示类似policy评估直接对计数器$\mathbf{n}~P(\mathbf{n};\pi)$进行采样计算梯度。类似的，也可以使用经验回报作为动作值函数$Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t)$的一个近似估计。这是标准的REINFORCE算法在CDec-POMDP上的应用。众所周知，REINFORCE可能比其他使用学习的动作值函数的方法学习的慢。因此，我们提出了一个$Q_t^{\pi}$的近似函数，展示了直接采样计数器$\mathbf{n}$来计算策略梯度。
### 使用估计动作值函数的策略梯度
估计动作值函数$Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t)$有几种不同的方式。我们考虑下列特征形式的近似值函数$f_w$：
$$Q_t^{\pi}(\mathbf{s}_t,\mathbf{a}_t)\approx f_w(\mathbf{s}_t,\mathbf{a}_t)=\sum_{m=1}^Mf_w^m(s_t^m,o(s_t^m,\mathbf{n_{s_t}}),s_t^m)$$
每一个智能体m都定义了一个$f_w^m$，它的输入是智能体的局部state，action和observation。注意不同的$f_w^m$是相关的，因为它们依赖于公共的计数器表$\mathbf{n}_{s_t}$。这样的一种分解方式是很有用的，因为它产生了有效的策略梯度计算方式。此外，CDec-POMDP中一类很重要的这种形式的估计值函数是兼容值函数最后会产生一个无偏的策略梯度。
**命题1** CDec-POMDP中的兼容值函数可以分解成：
$$f_w(\mathbf{s}_t\mathbf{a}_t) = \sum_mf_w^m(s_t^m,o(s_t^m,\mathbf{n}_{s_t}),a^m)$$
我们可以直接用估计值函数$f_w$取代$Q^{\pi}(\cdot)$。经验上来说，我们发现使用这个估计的方差很大。我们利用$f_w$的结构进一步分解策略梯度会有更好的效果。
**定理2** 对于任何具有如下的分解的值函数：
$$f_w(\mathbf{s}_t\mathbf{a}_t) = \sum_mf_w^m(s_t^m,o(s_t^m,\mathbf{n}_{s_t}),a^m)$$
策略梯度可以写成：
$$\Delta_{\theta}V_1(\pi)=\sum_{t=1}^HE_{\mathbf{s}_t,\mathbf{a}_t)|b_o,\pi}\left[\sum_m\Delta_{\theta}log\pi(a_t^m|s_t^m,o(s_t^m,\mathbf{n}_{s_t}))f_w^m(s_t^m,o(s_t^m,\mathbf{n}_{s_t}),a_t^m)\right]$$
上述结果展示了如果估计值函数被分解了，那么得到的策略梯度也是分解的。上述结果也可以应用到多种类型的智能体上，只要我们假设不同的智能体有不同的函数$f_t^m$。最简单的情况下，所有的智能体都是相同类型的，每一个智能体都有相同的函数$f_w$，推断出下式：
$$f_w(\mathbf{s}_t,\mathbf{a}_t) = \sum_{i,j}n_t(i,j)f_w(i,j,o(i,\mathbf{n}_{s_t}))$$
使用上式，我们可以将策略梯度简化成：
$$\Delta_{\theta}V_1(\pi) = \sum_tE_{\mathbf{s}_t,\mathbf{a}_t}\left[\sum_{i,j}n_t(i,j)\Delta_{\theta}log\pi (j|i,o(i,\mathbf{n}_{s_t}))f_w(i,j,o(i,\mathbf{n}_{s_t}))\right]$$
### 基于计数器的策略梯度计算
注意在上式中，期望仍然和联合state，action，$(\mathbf{s}_t,\mathbf{a}_t)$相关，当智能体的个数很大时效率很低。为了解决这个问题是
**定理3** 对于任何拥有形式$f_w(\mathbf{s}_t,\mathbf{a}_t) = \sum_{i,j}n_t(i,j)f_w(i,j,o(i,\mathbf{n}_{s_t}))$的值函数，策略梯度都可以用下式计算：
\begin{equation}
E_{\mathbf{n}_{1:H}\in \Omega_{1:H}} \left[\sum_{t=1}^H\sum_{i\in S,j\in A}n_t(i,j) \Delta_{\theta}log\pi (j|i,o(i,\mathbf{n}_t)) f_w(i,j,o(i,\mathbf{n}_t))\right]
\end{equation}
上述结果展示了策略梯度可以类似于计算policy的值函数一样通过从底层分布$P(\cdot)$中采样计数表向量$\mathbf{n}_{1:H}$来计算策略梯度，在智能体数量很大的情况下也是可行的。
## 训练动作值函数
在我们的方法中，在计数器样本$\mathbf{n}_{1:H}$生成用来计算策略梯度后，我们还需要调整critic $f_w$的参数。注意对于每一个动作值函数$f_w(\mathbf{s}_t,\mathbf{a}_t)$只取决于联合state，action $(\mathbf{s}_t,\mathbf{a}_t)$生成的计数器。训练$f_w$可以通过一个梯度步最下化下列loss函数实现：
\begin{equation}
min_w\sum_{\xi=1}^K\sum_{t=1}^H\left(f_w(\mathbf{n}_t^{\xi})-R_t^{\xi}\right)^2
\end{equation}
其中$\mathbf{n}_{1:H}^{\xi}$是从分布$P(\mathbf{n};\pi)$中生成的一个计数器样本；$f_w(\mathbf{n}_t^{\xi})$是动作值函数，$R_t^{\xi}$是用式子$(1)$计算的$t$时刻的所有经验回报：
\begin{equation}
f_w(\mathbf{n}_t^{\xi}) = \sum_{i,j}n_t^{\xi}(i,j)f_w(i,j,o(i,\mathbf{n}_t^{\xi});R_t^{\xi}=\sum_{T=t}^H]\sum_{i\in S,j\in A}n_T{\xi}(i,j)r_T(i,j,\mathbf{n}_T^{\xi})
\end{equation}
然而，我们发现公式$(11)$中的loss函数在训练较大问题的critic时表现并不好。需要一定数量的计数器样本可靠的训练$f_w$，这对于拥有较多数量智能体的大问题的扩展有不利影响。已知在多智能体强化学习中单独利用全局reward信号的算法要比利用局部reward信号的方法多用一些样本。受到这些现象的启发，接下来我们提出了一个基于策略的局部reward信号去训练critic $f_w$。
**单个值函数** 用$\mathbf{n}_{1:H}^{\xi}$表示一个计数器样本。给定计数器样本$\mathbf{n}_{1:H}^{\xi}$，用$V_t^{\xi}(i,j)=E\left[\sum_{t'=t}^Hr_{t'}^m|s_t^m=i,a_m^t=j,n_{1:H}^{\xi}\right]$表示一个智能体在时刻$t$处于state $i$，采取action $j$，所能得到的所有期望reward。这个单个的值函数可以用动态规划算法来计算。基于这个值函数，我们接下来展示了式子$(12)$中全局经验reward的重新参数化：
**引理(Lemma)1** 给定计数器样本$\mathbf{1:H}^{\xi}$，$t$时刻的经验回报$R_t^{\xi}$可以被重新参数化为：
$$R_t^{\xi} = \sum_{i\in S,j\in A}n_t^{\xi}(i,j)V_t^{\xi}(i,j).$$
**基于单个值函数的loss** 给出引理$1$，我们推导出式子$11$中真实loss的上界，它有效利用了单个值函数：
\begin{align\*}
&\sum_{\xi}\sum_t\left(f_w(\mathbf{n}^{\xi})-R_t^{\xi}\right)^2 \\
= &\sum_{\xi}\sum_t\left(\sum_{i,j}n_t^{\xi}(i,j)f_w(i,j,o(i,\mathbf{n}_t^{\xi}))-\sum_{i,j}n_t^{\xi}(i,j)V_t^{\xi}(i,h)\right)^2\\
= &\sum_{\xi}\sum_t\left( \sum_{i,j}n_t^{\xi}(i,j)(f_w(i,j,o(i,\mathbf{n}_t^{\xi}))-V_t^{\xi}(i,h))\right)^2\\
\le &M\sum_{\xi}\sum_{t,i,j}n_t(i,j)\left(f_w(i,j,o(i,\mathbf{n}_t^{\xi}))-V_t^{\xi}(i,j)\right)^2
\end{align\*}
其中最后一部用了柯西施瓦茨不等式。我们用式子(14)中修改过的loss训练critic。按照经验来说，对于较大的问题，式子(14)中的新loss比式子(13)中的原始loss要收敛的快很多。直观上来说，这是因为式子(14)中的新loss尝试调整每一个critic组件$f_w(i,j,o(i,\mathbf{n}_t^{\xi}))$更接近它的经验回报$V_t^{\xi}(i,j)$。然而，原始的式子(13)中的loss着重于最小化全局loss，而不是调整每一个单个的critic因子$f_w(\cdot)$到相对应的每一个经验回报。
算法$1$展示了CDec-POMDP中AC算法的大纲。第$7$行和第$8$行展示了两种不同的方式训练critic。第$7$行代表基于局部值函数的critic更新，也可以称为factored cirtic更新(fC)。第$8$行展示了基于全局reward或者全局critic的更新(C)。第$10$行展示了使用定理$2$(fA)计算的策略梯度。第$11$行展示了直接使用$f_w$计算的梯度。

## 实验
这一节中比较了我们的AC算法和另外两个解决CDec-POMDP问题的算法，Soft-Max based flow update(SMFU)，和期望最大化方法。SMFU只能优化智能体的action依赖于局部state的policy，$\pi(a_t^m|s_t^m)$，因为它通过计算在规划阶段单个最有可能的计数器向量来估计计数器$\mathbf{n}$的作用。EM方法优化基于计数器的分段线性policy，其中$\pi(a_t^m|s_t^m,\cdot)$是所有可能的计数器observation $o_t$空间上的一个分段函数。
算法$1$展示了更新critic的两种方式（第$7$行和第$8$行）和更新actor的两种方式（第$10$行和第$11$行），所以就有四种可能的AC方法－fAfC,AC,FfC,fAC。我们也研究了不同actor-critic方法的属性。在附录中有神经网络的结构和其他一些实验设置。
为了和之前方法公平的进行比较，我们使用了三种不同的模型用于基于计数的observation $o_t$。在$o0$设置中，policy只取决于智能体的局部state $s_t^m$并不需要计数器。在$o1$设置中，policy取决于局部state $s_t^m$和单个计数器observation $n_t(s_t^m)$。也就是说，智能体只能观测到其他也在当前状态$s_t^m$的智能体的计数器。在$oN$设置中，智能体能观测到它的局部state $s_t^m$和当前状态$s_t^m$的局部相邻状态内其他智能体的计数器。$oN$ observation模型提供给智能体最多的信息。然而，它也是最难优化的因为policy有更多的参数。SMFU方法只能在$o0$设置中起作用，EM方法和本文中的AC方法在所有设置中都能起作用。
<!--
**出租车调度** 我们在第二节中介绍的现实世界中的域测试了本文的方法。在这个问题中，目标是计算出租车policy优化整个车队的收入。数据包含亚洲一个大城市超过一年的出租车轨迹数据。我们使用了从数据集中提取到的车辆请求信息。平均来说，每天大概有$8000$辆出租车。整个城市被划分为$81$个空间，时间范围是$24$个小时划分为$48$个半小时的区间。
图$2(a)$中展示了不同方法在不同的观测模型（$'o0','o1','oN'$)上的量化比较。我们测试了$4000$和$8000$辆出租车来验证是否出租车的数量会影响不同方法的性能。$y$轴展示了整个车队每天的利润。在$'o0'$设置下，所有的方法（fAfC-$o0$,SMFU,EM-$o0$）给出质量差不多的解，在$8000$个出租车上fAfC-$o0$和EM-$o0$表现的比SMFU稍微好一些。在$'o1'$设置下，
-->
