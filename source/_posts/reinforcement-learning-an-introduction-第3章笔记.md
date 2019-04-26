---
title: reinforcement learning an introduction 第3章笔记
date: 2018-12-21 15:13:38
tags: 
  - 强化学习
  - MDP
  - MRP
  - Bellman Equation
categories: 强化学习
mathjax: true
---

## 马尔科夫过程(markov process)、马尔科夫链(markov chain)
马尔科夫过程或者马尔科夫链(markov chain)是一个tuple $\lt S,P\gt$,其中S是一个有限(或者无限)的状态集合,P是状态转移矩阵(transition probability matrix)或马尔科夫矩阵(markov matrix),$P_{ss'}= P[S_{t+1} = s'|S_t = s]$.

## 马尔科夫奖励过程(markov reward process)
马尔科夫奖励过程是一个tuple $\lt S,P,R,\gamma\gt$,和马尔科夫过程相比，它多了一个奖励R，R和某个具体的状态相关，MRP中的reward只和state有关,和action无关。
S是一个(有限)状态的集合。
P是一个状态转移概率矩阵。
R是一个奖励函数$R = \mathbb{E}[R_{t+1}|S_t = s]$, **这里为什么是t+1时刻的reward?这仅仅是一个约定，为了描述RL问题中涉及到的observation，action，reward比较方便。这里可以理解为离开这个状态才能获得奖励而不是进入这个状态即获得奖励。如果改成$R_t$也是可以的，这时可以理解为进入这个状态获得的奖励。**
$\gamma\$称为折扣因子(discount factor), $\gamma \epsilon [0,1]$.**为什么引入$\gamma$，David Silver的公开课中提到了四个原因:(1)数学上便于计算回报(return)；(2)避免陷入无限循环；(3)长远利益具有一定的不确定性；(4)符合人类对眼前利益的追求。**

### 奖励(reward) 
每个状态s在一个时刻t立即可得到一个reward,reward的值需要由环境给出,这个值可正可负。目前的强化学习算法中reward都是人为设置的。

### 回报(return)
回报是累积的未来的reward,其计算公式如下:
$$G_t = R_{t+1} + R_{t+2} + ... = \sum_{k=0}^{\infty}{\gamma^k R_{t+k+1}} \tag{1}$$
它是一个马尔科夫链上从t时刻开始往后所有奖励的有衰减(带折扣因子)的总和。

### 值函数(value function)
值函数是回报(return)的期望(expected return), 一个MRP过程中某一状态的value function为从该状态开始的markov charin return的期望，即$v(s) = \mathbb{E}[G_t|S_t=s]$.
MRP的value function和MDP的value function是不同的, MRP的value function是对于state而言的，而MDP的value function是针对tuple $\lt$state, action$\gt$的。
这里为什么要取期望,因为policy是stotastic的情况时，在每个state时，采取每个action都是可能的，都有一定的概率，next state也是不确定的了，所以value funciton是一个随机变量，因此就引入期望来刻画随机变量的性质。
为什么在当前state就知道下一时刻的state了?对于有界的RL问题来说，return是在一个回合结束时候计算的；对于无界的RL问题来说，由于有衰减系数，只要reward有界，return就可以计算出来。

### 马尔科夫奖励过程的贝尔曼方程(bellman equation for MRP)
\begin{align\*}
v(s) &= \mathbb{E}[G_t|S_t = s]\\ 
&= \mathbb{E}[R_{t+1} + \gamma R_{t+2} + ... | S_t = s]\\
&= \mathbb{E}[R_{t+1} + \gamma (R_{t+2} + \gamma R_{t+3} + ...|S_t = s]\\
&= \mathbb{E}[R_{t+1} + \gamma G_{t+1} |S_t = s]\\
&= \mathbb{E}[R_{t+1} + \gamma v(S_{t+1})|S_t = s]\\
v(s) &= \mathbb{E}[R_{t+1} + \gamma v(S_{t+1})|S_t = s]
\end{align\*}
v(s)由两部分组成，一部分是immediate reward的期望(expectation)，$\mathbb{E}[R_{t+1}]$, 只与当前时刻state有关；另一部分是下一时刻state的value function的expectation。如果用s'表示s状态下一时刻的state，那么bellman equation可以写成：
$$v(s) = R_s + \gamma \sum_{s' \epsilon S} P_{ss'}v(s')$$
我们最终的目的是通过迭代使得t轮迭代时的v(s)和第t+1轮迭代时的v(s)相等。将其写成矩阵形式为：
$$v_t = R + \gamma P v_{t+1}$$
$$(v_1,v_2,...,v_n)^T = (R_1,R_2,...,R_n)^T + \gamma \begin{bmatrix}P_{11}&P_{12}&...&P_{1n}\\P_{21}&P_{22}&...&P_{2n}\\&&...&\\P_{n1}&P_{n2}&...&P_{nn}\end{bmatrix} (v_1,v_2,...,v_n)^T $$
MRP的Bellman方程组是线性的，可以直接求解:
\begin{align\*}
v &= R + \gamma Pv\\
(1-\gamma P) &= R\\
v &= (1 - \gamma P)^{-1} R
\end{align\*}
可以直接解方程，但是复杂度为$O(n^3)$，对于大的MRP方程组不适用，可以通过迭代法求解，常用的迭代法有动态规划,蒙特卡洛算法和时序差分算法等求解(动态规划是迭代法吗？）

## 马尔科夫决策过程(markov decision process) 
马尔科夫决策过程，比markov reward process多了一个A,它也是一个tuple $\lt S,A,P,R,\gamma\gt$, 在MRP中奖励R仅仅和状态S相关，在MDP中奖励R和概率P对应的是某个状态S和某个动作A的组合。
\begin{align\*}
P_{ss'}^a &= P[S_{t+1} = s' | S_t = s, A_t = a]\\
R_s^a &= \mathbb{E}[R_{t+1} | S_t = s, A_t = a]
\end{align\*}
这里的reward不仅仅与state相关，而是与tuple $\lt state，action\gt$相关。

### 回报
MDP中的$G_t$和式子$(1)$的$G_t$是一样的，将$G_t$写成和后继时刻相关的形式如下：
\begin{align\*}
G_t &= R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \gamma^3 R_{t+4} + ...\\
&= R_{t+1} + \gamma (R_{t+2} + \gamma^1 R_{t+3} + \gamma^2 R_{t+4} + ...)\\
&= R_{t+1} + \gamma G_{t+1} \tag{2}
\end{align\*}
这里引入$\gamma$之后，即使是在continuing情况下，只要$G_t$是非零常数，$G_t$也可以通过等比数列求和公式进行计算，即:
$$G_t = \sum_{k=1}^{\infty}\gamma^k = \frac{1}{1-\gamma} \tag{3}$$

### 策略(policy)
策略$\pi$的定义:给定状态时采取各个动作的概率分布。
$$\pi(a|s) = P[A_t = a | S_t = a] \tag{4}$$

### 值函数(value function)
这里给出的是值函数的定义，就是这么定义的。
MDP的值函数有两种，状态值函数(state value function)和动作值函数(action value function), 这两种值函数的含义其实是一样的，也可以相互转换。具体来说, 值函数定义为给定一个policy $\pi$，得到的回报的期望(expected return)。
一个MDP的状态s对应的值函数(state value function) $v_{\pi}(s)$是从状态s开始采取策略$\pi$得到的回报的期望。
\begin{align\*}
v_{\pi}(s) &= \mathbb{E}_{\pi}[G_t|S_t = s]\\
&=\mathbb{E}_{\pi}[\sum_{k=0}^{\infty} \gamma^{k}R_{t+k+1}|S_t=s] \tag{5}
\end{align\*}
这里的$G_t$是式子(2)中的回报。
一个MDP过程中动作值函数(action value function) $q_{\pi}(s,a)$是从状态s开始,采取action a，采取策略$\pi$得到的回报的期望。
<action value function $q_{\pi}(s,a)$ is the expected return starting from states, taking action a, and then following policy \pi.>
\begin{align\*}
q_{\pi}(s,a) &= \mathbb{E}_{\pi}\left[G_t | S_t = s, A_t = a\right]\\
&= \mathbb{E}_{\pi}\left[\sum_{k=0}^{\infty} \gamma^{k}R_{t+k+1}|S_t=s, A_t=a\right] \tag{6}
\end{align\*}

#### 状态值函数(state value function)
\begin{align\*}
v_{\pi}(s) &= \sum_{a \epsilon A} \pi(a|s) q_{\pi} (s,a) \tag{7}\\
v_{\pi}(s) &= \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_{\pi}(s') \right] \tag{8}\\
\end{align\*}
式子$(7)$是$v(s)$和$q(s,a)$的关系，式子$(8)$是$v(s)$和它的后继状态$v(s')$的关系。
式子$(8)$的推导如下：
\begin{align\*}
v_{\pi}(s) &= \mathbb{E}_{\pi}[G_t|S_t = s]\\
&= \mathbb{E}_{\pi}\left[R_{t+1}+\gamma G_{t+1}|S_t = s\right]\\
&= \sum_a \pi(a|s)\sum_{s'}\sum_rp(s',r|s,a) \left[r + \gamma \mathbb{E}_{\pi}\left[G_{t+1}|S_{t+1}=s'\right]\right]\\
&= \sum_a \pi(a|s)\sum_{s',r}p(s',r|s,a) \left[r + \gamma v_{\pi}(s') \right]\\
\end{align\*}

#### 动作值函数(action value function)
\begin{align\*}
q_{\pi}(s,a) &= \sum_{s'}\sum_r p(s',r|s,a)(r + \gamma  v_{\pi}(s')) \\
q_{\pi}(s,a) &= \sum_{s'}\sum_r p(s',r|s,a)(r + \gamma  \sum_{a'}\pi(a'|s')q(s',a')) \tag{10}\\
\end{align\*}
式子$(9)$是$q(s,a)$和$v(s)$的关系，式子$(10)$是$q(s,a)$和它的后继状态$q(s',a')$的关系。
以上都是针对MDP来说的，在MDP中，给定policy $\pi$下，状态s下选择a的action value function，$q_{\pi}(s,a)$类似MRP里面的v(s)，而MDP中的v(s)是要考虑在state s下采率各个action后的情况。

### 贝尔曼期望方程(Bellmam expectation equation)
\begin{align\*}
v_{\pi}(s) &= \mathbb{E}_{\pi}[R_{t+1} + \gamma v_{\pi}(S_{t+1})|S_t = s] \tag{11}\\
v_{\pi}(s) &= \mathbb{E}_{\pi}\left[q_{\pi}(S_t,A_t)|S_t=s,A_t=a\right]\tag{12}\\
q_{\pi}(s,a)&= \mathbb{E}_{\pi}\left[R+\gamma v_{\pi}(S_{t+1}) |S_t=s,A_t=a\right]\tag{13}\\
q_{\pi}(s,a) &= \mathbb{E}_{\pi}[R_{t+1} + \gamma q_{\pi}(S_{t+1},A_{t+1}) | S_t = s, A_t = a] \tag{14}
\end{align\*}
#### 矩阵形式
\begin{align\*}
v_{\pi} &= R^{\pi} + \gamma P^{\pi} v_{\pi}\\
v_{\pi} &= (I-\gamma P^{\pi})^{-1} R^{\pi}
\end{align\*}

## 最优策程的求解(how to find optimal policy)
### 最优价值函数(optimal value function)
$v_{\*} = max_{\pi}v_{\pi}(s)$,从所有策略产生的state value function中，选取使得state s的价值最大的函数
$q_{\*}(s,a) = max_{\pi} q_{\pi}(s,a)$,从所有策略产生的action value function中，选取使$\lt s,a\gt$价值最大的函数  
当我们得到了optimal value function，也就知道了每个state的最优价值，便认为这个MDP被解决了

### 最优策略(optimal policy)
对于每一个state s，在policy $\pi$下的value 大于在policy $\pi'$的value， 就称策略$\pi$优于策略$\pi'$， $\pi \ge \pi'$ if $v_{\pi}(s) \ge v_{\pi'}(s)$, 对于任意s都成立
对于任何MDP，都满足以下条件：
1. 都存在一个optimal policy，它比其他策略好或者至少相等；
2. 所有的optimal policy的optimal value function是相同的；
3. 所有的optimal policy 都有相同的 action value function.

### 寻找最优策略
寻找optimal policy可以通过寻找optimal action value function来实现： 
$${\pi}_{\*}(a|s) = 
\begin{cases}
1, &if\quad a = argmax\ q_{\*}(s,a)\\
0, &otherwise\end{cases}$$

### 贝尔曼最优方程(bellman optimal equation)
\*号表示最优的策略。
#### 状态值函数(state value function)
\begin{align\*}
v_{\*}(s) &= max_a q_{\*}(s,a)\\
&= max_a\mathbb{E}_{\pi_{\*}}\left[G_t|S_t=s,A_t=a\right]\\
&= max_a\mathbb{E}_{\pi_{\*}}\left[R_{t+1}+\gamma G_t|S_t=s,A_t=a\right]\\
&= max_a\mathbb{E}\left[R_{t+1} +\gamma v_{\*}(S_{t+1})|S_t=s,A_t=a\right]\\
&= max_a \left[\sum_{s',r} p(s',r|s,a){\*}(r+\gamma v_{\*}(s') )\right] \tag{15}\\
\end{align\*}
#### 动作值函数(action value function)
\begin{align\*}
q_{\*}(s,a) &= \sum_{s',r} p(s',r|s,a) (r + \gamma v_{\*}(s'))\\
&= \sum_{s',r} p(s',r|s,a) (r + \gamma max_a q_{\*}(s',a'))\\
&=\mathbb{E}\left[R_{t+1}+\gamma max_{a'}q_{\*}(S_{t+1},a')|S_t=s,A_t=a \right]\tag{16}\\
\end{align\*}

### 贝尔曼最优方程的求解(solution to Bellman optimal equation)
Bellman equation和Bellman optimal equation相比，一个是对于给定的策略，求其对应的value function,是对一个策略的估计，而bellman optimal equation是要寻找最优策略，通过对action value function进行贪心。
Bellman最优方程是非线性的，没有固定的解决方案，只能通过迭代法来解决，如Policy iteration，value iteration，Q-learning，Sarsa等。

## 参考文献
1.http://incompleteideas.net/book/the-book-2nd.html
2.https://www.bilibili.com/video/av32149008/?p=2
