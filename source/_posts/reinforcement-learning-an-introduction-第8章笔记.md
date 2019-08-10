---
title: reinforcement learning an introduction 第8章笔记
date: 2019-08-07 16:32:52
tags:
 - 强化学习
 - model based
categories: 强化学习
---

## Planning and Learning with Tabular Methods
前面几章，介绍了MC算法，TD算法，最后又用一个框架把它们统一了起来。这些算法都是model free的方法。这一章要介绍的是model based方法。Model based方法主要用于planning，而model free的方法主要用于learning。这两类方法虽然差异很大，但是也有很多相似性。

## Models和Planning
### Model
Environment的model是任何agents可以用来预测environment会如何对agent的action做出响应的东西。给定一个state和一个action，model可以预测下一个state和action。如果model是stochastic的话，可能有多个next states和rewards，每一个都有一定概率发生。计算所有可能性以及相应概率的models，我们成为distribution models。只计算一个可能的结果，根据概率进行采样的方法，我们叫做sample models。
Model可以用来模仿或者仿真。给定一个start state和action，sample model产生一个可能的transition，distribution model产生所有可能的结果，并通过发生概率进行加权。给定一个start state和一个policy，sample产生一个episode，distribution产生所有可能的episodes以及相应的概率。不论在哪种情况下，我们都可以说model被用来simulate environment或者用来产生simulated experience。
DP中使用的$p(s', r|s,a)$就是distribution model。第五章中blackjack例子中使用的模型是sample model。distribution model要比sample model好，distribution model可以用来产生samples，但是sample model要比distribution models好获得。考虑投掷a dozen dice的和。distribution models计算所有可能的和以及相对应的概率。而sample model只计算根据概率产生的一个样本的和。

### Planning
Planning指的是输入model，不断的与environment交互产生或者改进poilcy的过程。在AI中，有两种方法进行planning，State-space planning和plan-space planning。State space是在state space中进行search寻找一个optimal policy或者optimal path，是这本书中讲的方法。而plan-space planning是在plans space中进行search。Plan-sapce 方法很难高效的应用到stochastic sequential decision problems，在这本书中不做过多介绍。
这一章要介绍的全是结构相同的state-space planning方法，这个结构不仅在planning中出现了，也在learning中出现了。基本的idea是：所有的state-space planning方法都有计算value funtions；并且通过应用simulated experience的update以及backup操作计算value functions。
DP方法显然遵循这个结构，首先扫描整个state space，然后生成每一个state可能的transitions的distribution，每一个distribution用于计算update target，更新state's estimated value。这一章会介绍其他不同的方法，也满足这个结构，只不过是计算target的方式顺序以及长度不同而已。
将planning方法表示成这种形式主要是为了强调planning方法和learning方法之间的联系。learning和planning的重点都是使用backup update op更新estimations of value functions。区别在于plannning使用了model生成的simulate experience，而learning使用environment生成的real experience。同时perfomance measure以及experience的灵活性也不同，但是由于相同的结构，许多learning的方法可以直接应用到planning上去，使用simulated experience代替real experience即可。给出一个利用sample model和Q-learning结合起来的planning算法如下：

### Random-sample one-step tabular Q-planning
Loop forever
随机选择初始state $S\in \mathbb{S}, A\in \mathbb{A}$
将$S,A$发送给sample model，得到next state和reward $S', R$
应用one-step Q-learning更新公式：
$Q(S,A) \leftarrow Q(S,A) + \alpha \left[R+\gamma mx_a Q(S', a) - Q(S, A)\right]$

## Dyna:整合planning, acting以及learning
当进行online的planning更新时，不断的与environment交互，就会出现一系列的问题。从交互中获得的information可能会改变model，以及与environment的交互。所以可能model也需要不断的学习。如果decision making以及model learning都是需要计算资源的话，可能需要把资源分给两个过程。这一节主要介绍Dyna-Q，将online planning需要的函数都整合了起来。
planning中real experience至少有两个作用，一个是改进model，叫做model-learning，并且可以用learning的方法改进value function和policy，叫做direct reinforcement learning。相应的关系如下图所示：
![](model_learning.png)
experience不仅可以直接改进value function，还可以通过model间接的改进value fucntion，叫做indirect reinforcement learning。Direct learning和indirect learning都各有优势。Indirect 方法能够充分利用有限的experience，得到一个更好的policy。另一方面，direct方法更简单，不会受到model bias的影响。
Dyna-Q包含了所有planning, acting, model-learning和direct RL，planning方法就是random-sample one-step tabular Q-planning；direct RL就是one-step tabular Q-learning，model-learning是table-based并且假设environment是deterministic，对于每一个transition $S_t,A_t\rightarrow R_{t+1}, S_{t+1}$，model用表格的形式记录下$S_t,A_t$的prediction值是$R_{t+1}, S_{t+1}$。
如下图是Dyna算法的整体框架图（Dyna-Q是一个示例）。
![](dyna.png)
左边使用real experience直接进行direct RL，右边是model-based的方法，从real experience中学习出model，然后生成simulated experience。search control指的是从model生成的simulated experiences中选择指定starting state和actions的experience。最后，planning使用simulated experience更新value function。
从概念上来讲，Dyna agents中planning, acting, model-learning以及direct RL几乎同时发生。但是在实现的时候，还是需要串行的进行。Dyna-$Q$中，计算量主要集中在planning上。具体的算法如下：
### Tabular Dyna-Q
初始化$Q(s,a), Model(s,a), s\in \mathbb{S}, a\in \mathbb{A}$
Loop forever
$\qquad (a)S\leftarrow$ current state
$\qquad (b)A\leftarrow \epsilon$-greedy$(S,Q)$
$\qquad (c)(c)$采取action $A$，得到下一时刻的state $S'$和reward $R$
$\qquad (d)Q(S,A) \leftarrow Q(S,A) + \alpha\left[R+\gamma max_a Q(S', a) -Q(S,A)\right]$
$\qquad (e)Model(S,A)\leftarrow R,S'$（假设deterministic environment)
$\qquad (f)$Loop repeat $n$ 次
$\qquad\qquad S\leftarrow$ 任意之前的观测状态
$\qquad\qquad A\leftarrow$ 在$S$处采取的任意action $A$
$\qquad\qquad R,S'\leftarrow Model(S,A)$
$\qquad\qquad Q(S,A) \leftarrow Q(S,A) + \alpha\left[R+\gamma max_a Q(S', a) -Q(S,A)\right]$

其中$Model(s,a)$表示预测$(s,a)$的next state。(d)是direct RL，(e)是model-learning，(f)是planning。如果忽略了(e,f)，就是one-step tabular Q-learning。

## 如果model出错
前面给的例子很简单，model是不会错的。但是如果environment是stochastic的，以及samples很少的话，或者function approximation效果不好，或者environment刚刚改变，新的behaviour还没有被观测到，model就有可能会出错。当model是错的话，就可能会产生suboptimal policy。在一些情况下，suboptimal会发现并且纠正model的error。当模型预测的结果比真实的结果好的时候就会发生这种情况。
这里给出两个例子。一个model变坏，一个model变好。
第一个例子，有一个迷宫，如图所示。
![blocking mase](blocking_maze.png)
刚开始，路在右边，1000步之后，右边的路被堵上了，左边有一条新的路。
![blocking mase](shortcut_maze.png)
第二个例子，刚开始路在左边，3000步之后，右边有一条新的路，左边的路也被保留。
这又是一个exploration和exploitation问题。在planning中，exploration意味着尝试那些让model变得更好的actions，而exploitation意味着给定当前的model，选择optimal action。我们想要agents能够explore environment的变化，但是不影响performance。
Dyna-Q+ 使用了一个简单的heuristics，非常有效，agent记录每一个state-action pair 在real environment中从上次使用到现在经历了多少个time steps，累计的时间步越多，说明这个pair改变的可能性越大，model不对的可能性越大。为了鼓励尝试很久没有用的action，这里加了一个bonus reward，如果一个transition的reward是$r$，这个transition已经有$\tru$步没有试过，在planning进行update的时候，用一个新的reward $r + k\sqrt{\tau}$，$k$很小。不过新添加的bonus会对value function造成影响，而且会有一定的cost，但是在上面的两个例子中，这个cost比performance的提升要好。


## 优先级
Dyna agents中，所有的state-action pair都被均匀随机的选中。但是，显然这里不合理的，planning可以更有效的集中在某些state-action pair中。举个例子，在dyna_maze例子中，在第二个episode开始的时候，只有goal state前的那个state会产生正的reward，其中的都仍然是$0$，这里意味着很多updates都是无意义的。从一个value为$0$的state转移到另一个value为$0$的state，这个updates没有意义。只有哪些在goal掐面的state才会被更新，或者，那些value不为$0$的state的前面的state的value的更新才有意义。在planning过程中，有用的更新变多了，但是离effcient还差得多。我们的真实应用中，states可能相当大，这种没有重点的更新是非常低效的。。
从dyna maze的例子中，我们可以得到一个idea，从gloal state backward的进行更新。这里的goal state不是一个具体的goal state，指的是抽象的goal state。一般来说，包括goal state以及value改变的state进行更新。假设model的value都是正确的，如果agent发现了environment的一个变化以及相应state value的变化，首先更新这个state的predecessor states value，然后一直往前利用value改变的state进行更新就行了。这种想法叫做backward focusing。
Backward过程增长的很快，会有很多state-action被更新。但是并不是所有的pair都是等价的。有的state value变得很大，有的变得很小。在stochastic环境中，transiton probability的估计也会导致pair大小和紧急性的变化。所以，根据紧急性对不同的pair排一个优先级，根据优先级进行更新。用一个queue记录如果更新某个state-action pair的话，它的estimated value会变多少，根据这个值的大小排优先级。对头的元素取出来进行更新以后，计算它的predecessor pair变化大小。如果这个大小大于某个阈值，就使用新的优先级，把它加入队列，如果queue中有这个pair了，queue中保存大的优先级。完整的算法如下所示：

### prioritized sweeping for a deteriministic environment
初始化$Q(s,a), Model(s,a), \forall s, a$，$PQueue$置为空
Loop forever
$\qquad(a) S\leftarrow$ currnet state
$\qquad(b) A\leftarrow policy(S,Q)$
$\qquad$(c)采取action $A$；得到下一个reward $R$和state $S'$
$\qquad$(d)Model(S,A) \leftarrow R, S'
$\qquad(e) P\leftarrow |R+\gamma max_aQ(S',a) - Q(S,A)|$
$\qquad$(f)如果$P\gt \theta$，将$S,A$以$P$的优先级插入$PQueue$
$\qquad$(g) Loop repeat $n$次，当$PQueue$非空的时候
$\qquad S, A\leftarrow fisrt(PQueue)$
$\qquad R,S'\leftarrow Model(S,A)$
$\qquad Q(S,A) \leftarrow Q(S,A) +\alpha \left[R+\gamma max_a Q(S',a) -Q(S,A)$
$\qquad$Loop for all预计能到$S$的$\bar{S}, \bar{A} $
$\qquad\qquad \bar{R} \leftarrow $predicted reward for $\bar{S}, \bar{A}, S$
$\qquad\qquadP\leftarrow |\bar{R}+\gamma max_aQ(S,a) -Q(\bar{S}, \bar{A})$
$\qquad\qquad$如果$P\gt \theta$，将$\bar{S},\bar{A}$以$P$的优先级插入到$PQueue$





## 参考文献
1.《reinforcement learning an introduction》第二版
