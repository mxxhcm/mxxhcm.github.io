---
title: reinforcement learning an introduction 第8章笔记
date: 2019-08-07 16:32:52
tags:
 - 强化学习
 - model based
categories: 强化学习
mathjax: true
---

## Planning and Learning with Tabular Methods
前面几章，介绍了MC算法，TD算法，再用$n$-step TD框架把它们统一了起来，此外，这些算法都属于model free的方法。这一章要介绍model based方法，model based方法主要用于planning，而model free的方法主要用于learning。

## Models和Planning
### Model
#### 定义
Environment的model是agents用来预测environment会如何对agent的action做出响应的任何东西。给定一个state和一个action，model可以预测下一个state和action。如果model是stochastic的话，有多个可能的next states和rewards，每一个都有一定概率。Distribution models计算所有可能性以及相应概率，sample models根据概率进行采样，只计算采样的结果。
#### 用处
Model可以用来模仿或者仿真。给定一个start state和action，sample model产生一个transition，distribution model产生所有可能的transitions，并使用概率进行加权。给定start state和一个policy，sample model产生一个episode，distribution model产生所有可能的episodes以及相应的概率。Model被用来模拟environment或者产生simulated experience。
#### 示例
DP中使用的$p(s', r|s,a)$就是distribution model，第五章中blackjack例子中使用的模型是sample model。Distribution model可以用来产生samples，但是sample model要比distribution models好获得。考虑投掷很多骰子的和，distribution models计算所有可能值和相应的概率，而sample model只计算根据概率产生的一个样本。

### Planning
#### 定义
Planning的定义是给定model，不断的与environment交互生成或者改进poilcy的过程。有两种planning的方法，state space在state space中寻找一个optimal policy或者optimal path，这本书中介绍的方法都是这类。Plan-space planning在plans space中search。Plan-sapce 方法很难高效的应用到stochastic sequential decision problems，在这本书中不做过多介绍。
这一章要介绍的state-space planning方法具有相同的结构，这个结构在learing和planning中都有。基本的想法是：计算value funtions，通过应用simulated experience的update以及backup操作计算value functions。

#### 示例
如DP方法，扫描整个state space，然后生成每一个state可能的transitions的distribution，用于计算update target，更新state's estimated value。这一章介绍的其他方法，也满足这个结构，只不过计算target的方式顺序以及长度不同而已。

#### planning和learning的关系
将planning方法表示成这种形式主要是为了强调planning方法和learning方法之间的联系。learning和planning的重点都是使用backup update op更新estimations of value functions。区别在于plannning使用了model生成的simulate experience，而learning使用environment生成的real experience。同时perfomance measure以及experience的灵活性也不同，但是由于相同的结构，许多learning的方法可以直接应用到planning上去，使用simulated experience代替real experience即可。
给出一个利用sample model和Q-learning结合起来的planning算法：

### learning算法示例
Random-sample one-step tabular Q-planning
Loop forever
$\qquad 1.$随机选择初始state $S\in \mathbb{S}, A\in \mathbb{A}$
$\qquad 2.$将$S,A$发送给sample model，得到next state和reward $S', R$
$\qquad 3.$应用one-step Q-learning更新公式：
$\qquad\qquad Q(S,A) \leftarrow Q(S,A) + \alpha \left[R+\gamma mx_a Q(S', a) - Q(S, A)\right]$

## Dyna
### 简介
Online的planning更新需要不断的与environment交互，从交互中获得的information可能会改变model，以及与environment的交互，所以可能model也需要不断的学习。这一节主要介绍Dyna-Q，将online planning需要的内容都整合了起来，Dyna算法中包含了planning, acting以及learning。
Planning中real experience至少有两个作用，一个是改进model，叫做model-learning，另一个是使用learning的方法直接改进value function和policy，叫做direct reinforcement learning。相应的关系如下图所示：
![model_learning](model_learning.png)
如图所示，experience可以直接改进value function，也可以通过model间接改进value fucntion，叫做indirect reinforcement learning。Direct learning和indirect learning各有优势，indirect方法能够充分利用有限的experience，得到一个更好的policy；direct方法更简单，不会受到model bias的影响。

### Dyna-$Q$简介
Dyna-$Q$包含planning, acting, model-learning和direct RL。planning是random-sample one-step tabular Q-planning；direct RL就是one-step tabular Q-learning，model-learning是table-based并且假设environment是deterministic，对于每一个transition $S_t,A_t\rightarrow R_{t+1}, S_{t+1}$，model用表格的形式记录下$S_t,A_t$的prediction值是$R_{t+1}, S_{t+1}$。如下图是Dyna算法的整体框架图（Dyna-Q是一个示例）。
![dyna_q](dyna.png)
左边使用real experience进行direct RL，右边是model-based的方法，从real experience中学习出model，然后利用model生成simulated experience。search control指的是从model生成的simulated experiences中选择指定starting state和actions的experience。最后，planning使用simulated experience更新value function。从概念上来讲，Dyna agents中planning, acting, model-learning以及direct RL几乎同时发生。但是在实现的时候，还是需要串行的进行。Dyna-$Q$中，计算量主要集中在planning上。具体的算法如下：
### Tabular Dyna-Q
初始化$Q(s,a), Model(s,a), s\in \mathbb{S}, a\in \mathbb{A}$
Loop forever
$\qquad (a)S\leftarrow$ current state
$\qquad (b)A\leftarrow \epsilon$-greedy$(S,Q)$
$\qquad ( c )$采取action $A$，得到下一时刻的state $S'$和reward $R$
$\qquad (d)Q(S,A) \leftarrow Q(S,A) + \alpha\left[R+\gamma max_a Q(S', a) -Q(S,A)\right]$
$\qquad (e)Model(S,A)\leftarrow R,S'$（假设deterministic environment)
$\qquad (f)$Loop repeat $n$ 次
$\qquad\qquad S\leftarrow$ 任意之前的观测状态
$\qquad\qquad A\leftarrow$ 在$S$处采取的任意action $A$
$\qquad\qquad R,S'\leftarrow Model(S,A)$
$\qquad\qquad Q(S,A) \leftarrow Q(S,A) + \alpha\left[R+\gamma max_a Q(S', a) -Q(S,A)\right]$

其中$Model(s,a)$表示预测$(s,a)$的next state。(d)是direct RL，(e)是model-learning，(f)是planning。如果忽略了(e,f)，就是one-step tabular Q-learning。

## 如果model出错
前面给的例子很简单，model是不会错的。但是如果environment是stochastic的，或者samples很少的话，或者function approximation效果不好，或者environment刚刚改变，新的behaviour还没有被观测到，model就可能会出错。当model是错的话，就可能会产生suboptimal policy。在一些情况下，suboptimal会发现并且纠正model的error。当模型预测的结果比真实的结果好的时候就会发生这种情况。这里给出两个例子。一个environment变坏，一个environment变好。
第一个例子，有一个迷宫，如图所示。
![blocking mase](blocking_maze.png)
刚开始，路在右边，1000步之后，右边的路被堵上了，左边有一条新的路。
![blocking mase](shortcut_maze.png)
第二个例子，刚开始路在左边，3000步之后，右边有一条新的路，左边的路也被保留。
这又是一个exploration和exploitation问题。在planning中，exploration意味着尝试那些让model变得更好的actions，而exploitation意味着给定当前的model，选择optimal action。我们想要agents能够explore environment的变化，但是不影响performance。
### 启发式搜索
Dyna-Q+ 使用了一个简单有效的heuristics，agent记录每一个state-action pair 在real environment中从上次使用到现在经历了多少个time steps，累计的时间步越多，说明这个pair改变的可能性越大，model不对的可能性越大。为了鼓励使用很久没有用的action，这里加了一个bonus reward，如果一个transition的reward是$r$，这个transition已经有$\tau$步没有试过，在planning进行update的时候，用一个新的reward $r + k\sqrt{\tau}$，$k$很小。不过新添加的bonus会有一定的cost，而且会对value function造成影响，但是在上面的两个例子中，这个cost比performance的提升要好。

## 优先级
在Dyna进行planning时，所有的state-action pair被选中的概率是一样的，显然是不合理的，planning的效率太低，可以有效的集中在某些state-action pair中。举个例子，在dyna_maze例子中，在第二个episode开始的时候，只有goal state前的那个state会产生正的reward，其余的都仍然是$0$，这里意味着很多updates都是无意义的。从一个value为$0$的state转移到另一个value为$0$的state，这个updates是没有意义的。只有那些在goal前面的state才会被更新，或者，那些value不为$0$的state的前面的state的value的更新才有意义。在planning过程中，有用的更新变多了，但是离effcient还差得多。在真实应用中，states可能相当大，这种没有重点的更新是非常低效的。。
### backward focusing
Dyna maze的例子给了我们一个提示，从goal state backward的进行更新。这里的goal state不是一个具体的goal state，指的是抽象的goal state。一般来说，包括goal state以及value改变的state。假设model的value都是正确的，如果agent发现了environment的一个变化以及相应state value的变化，首先更新这个state的predecessor states value，然后一直往前利用value改变的state进行更新就行了。这种想法叫做backward focusing。对于那些低效的state，不更新就是了。
Backward过程很快，会有很多state-action被更新，但并不是所有的pair是等价的。有的state value变化很大，有的变化很小。在stochastic环境中，对transiton probability变化的估计也会导致change大小的变化和紧急性的变化。所以，根据紧急性对不同的pair排一个优先级，然后根据这个优先级进行更新。用一个queue记录如果更新某个state-action pair的话，它的estimated value会变多少，根据这个值的大小排优先级。队头的元素取出来进行更新以后，计算它的predecessor pair变化大小。如果这个大小大于某个阈值，就使用新的优先级，把它加入队列，如果queue中有这个pair了，queue中保存大的优先级。完整的算法如下所示：

### deteriministic environment下的优先级
初始化$Q(s,a), Model(s,a), \forall s, a$，置$PQueue$为空
Loop forever
$\qquad(a) S\leftarrow$ currnet state
$\qquad(b) A\leftarrow policy(S,Q)$
$\qquad( c )$采取action $A$；得到下一个reward $R$和state $S'$
$\qquad(d)Model(S,A) \leftarrow R, S'$
$\qquad(e) P\leftarrow |R+\gamma max_aQ(S',a) - Q(S,A)|$
$\qquad$(f)如果$P\gt \theta$，将$S,A$以$P$的优先级插入$PQueue$
$\qquad$(g) Loop repeat $n$次，当$PQueue$非空的时候
$\qquad S, A\leftarrow fisrt(PQueue)$
$\qquad R,S'\leftarrow Model(S,A)$
$\qquad Q(S,A) \leftarrow Q(S,A) +\alpha \left[R+\gamma max_a Q(S',a) -Q(S,A)\right]$
$\qquad$Loop for all 预计能到$S$的$\bar{S}, \bar{A} $
$\qquad\qquad \bar{R} \leftarrow $predicted reward for $\bar{S}, \bar{A}, S$
$\qquad\qquad P\leftarrow |\bar{R}+\gamma max_aQ(S,a) -Q(\bar{S}, \bar{A})$
$\qquad\qquad$如果$P\gt \theta$，将$\bar{S},\bar{A}$以$P$的优先级插入到$PQueue$

推广到stochastic environment上，用model记录state-action pair experience的次数，以及next state，计算出概率，然后进行expected update。Expected update会计算许多低概率transition上，浪费资源，所以可以使用sample update。
Sample update相对于expected update的好处，相当于将完整的backup分解成单个更小的transition。这个idea是从van Seijen和Sutton(2013)的一篇论文中得到的，叫做"small backups"，使用单个的transition进行更新，像sample update，但是使用的是这个transition的概率，而不是sampling的$1$。

## Expected updates和Sample updates
这本书介绍了不同的value function updates，就one-step方法来说，主要有三个binary dimensions。第一个是估计state values还是action values；第二个是估计optimal policy还是random policy的value，对应四种组合:$q_{\*}, v_{\*}, q_{\pi}, v_{\pi}$；第三个是使用expected updates还是sample updates，也是这节的内容。总共有八种组合，其中七种对应具体的算法，如下图所示：
![seven_bakcup](seven_backup.png)
这些算法都可以用来planning，之前介绍的Dyna-$Q$使用的是$q_{\*}$ sample update，也可以用$q_{\*}$ expected update，还可以用$q_{\pi}$ sample updates。

### 简介和比较
Expected update对于每一个$(s,a)$ pair，考虑所有可能的next state和next action $(s',a')$，需要distributions model进行精确计算；而sample update仅仅需要sample model，考虑一个next state，会有采样误差。所以expected upadte一定要比sample update好，但是需要的计算量也大。当环境是deteriministic的话，expected udpaet和sample update其实是一样的，只有在stochastic环境下才有区别。
假设每一个state有$b$个next state，expected upadte要比sample update的计算量大$b$倍。如果有足够的时间进行完全的expected update，进行一次完全的expected update一定比进行$b$次sample update好，因为虽然计算次数相等，但是sample update有sampling error；如果没有足够的时间的话，在计算次数小于$b$次的时候，sample update要比expected update好，sample update至少进行了一部分improvement，而sample update只有完全进行$b$次计算后才会得到正确的value function。而当state-action pairs很多的情况下，进行完全的expected update是不可能的，sample update是一个可行的方案。
给定一个state-action pair，到底是进行一些（小于$b$次）expected update好呢，还是进行$b$次sample updates好呢？如下图所示，展示了expected update和sample update在不同的$b$下，estimation error关于计算次数的函数。
![fig_8_7](fig_8_7.png)
在这个例子中，所有$b$个后继状态出现的可能性相等，开始的error为$1$。假设所有的next state value都是正确的，expected update完成之后，error从$1$变成了$0$。而sample updates根据$\sqrt{\frac{b-1}{bt}}$减少error。对于$b$很大的值来说，进行$b$的很小比例次数的更新，error就会下降很快。

### Sample updates的好处
上图中，sample update的结果可能要比实际结果差一些。在实际问题中，后继状态的value会被它们自身更新。他们的estimate value会更精确，从后继状态进行backup也会更精确。

### 结论
在stochastic环境下，如果每个state处可能的next state数量非常多，并且有很多个states，那么sample update可能要比expected update好。

## Trajectory Sampling
这里比较两种distributing updates。
DP进行update的时候，每一次扫描整个state spaces，很多state其实是没有用的，没有focus，所有的states地位一样。原则上，只要确保收敛，任何distributed方法都行，然而在实际上常用的是exhaustive sweep。
第二种是根据一些distributions从state space或者state action space中进行采样。Dyna-$Q$使用均匀分布采样，和exhaustive sweep问题一样，没有focus。使用on-policy distribution是一种很不错的想法，根据当前的policy不断的与model交互，产生一个trajectory。Sample state transitions以及reward是model给出来的，sample action是当前的policy给出来的。这种方法叫做generating experience和update trajectory sampling。

### 原因
如果on-policy的distribution是已知的，可以根据这个distribution对所有的states进行加权，但是这和exhaustive sweeps的计算量差不多。或者从distribution中采样state-action paris，这比simulating trajectories好在哪里呢？事实上我们不知道distribution，当policy改变，计算distribution的计算量和policy evaluation相等。

### 分析
使用on-policy distribution可以去掉很多我们不感兴趣的内容，或者一遍又一遍的进行无用的重复更新。通过一个小例子分析它的效果。使用one-step expected updates，公式如下：
$$Q(s,a) \leftarrow \sum_{s',r} \hat{p}(s',r|s,a)\left[r+\gamma max_{a'}Q(s',a')\right]$$
使用均匀分布的时候，对所有的state-action pair进行in place的expected update更新；使用on-policy的时候，使用当前$\epsilon$-greedy policy（$\epsilon=0.1$）直接生成episodes，对episodes中出现的state-action pair进行expected update。也就是说一个是随机选的，一个是on-policy中出现的，选择的方式不一样，但是都是进行expected update。
简单介绍一下environment。每个state都有两个action，等可能性的到达$b$个next states，$b$对于所有state-action pair都是一样的，所有的transition都有$0.1$的概率到达terminal state。Expected reward服从均值为$0$,方差为$1$的高斯分布。
在planning过程中的任何一步都可以停止，根据当前估计的action value，计算greedy policy $\hat{\pi}$下start state的value $v_{\hat{\pi}}(s_0)$，也就是说告诉我们使用greedy重新开始一个新的episodes，agent的表现会怎么样。（假设model是正确的）。
![fig_8_8](fig_8_8.png)
上半部分是进行了$200$次sample任务，$1000$个states，$b$分别为$1,3,10$的结果。在图中根据on-policy 采样更新的方法，一开始很快，时间一长，就慢下来了。当$b$越小的时候，效果越好，越快。另一个实验中表明，随着states数量的增加，on-policy distribution采样的效果也在变好。
使用on-policy distribtion进行采样，能够帮助我们关注start state的后继状态。如果states很多，并且$b$很小，这个效果很好并且会持续很久。在长时间的计算中，使用on-policy distribution采样可能会有副作用，因为一直在更新那些value已经正确的states。对它们进行采样没用了，还不如采样一些其他的states。这就是为什么在长时间的运行中使用均匀分布进行采样的效果更好。

## Real-time DP
Real time DP是DP的on-policy trajectory sampling版本。RLDP和上一节介绍的on-policy expected update的区别在update的方式不同，上一届实际上使用的是state-action pair，而这一节DP用的是state，它的更新公式是：
$$v_{k+1}(s) = \max_a \sum_{s',r}p(s',r|s,a) \left[r+\gamma v_k(s')\right]$$
一个是更新action value function，一个是更新state valut function。

### irrelevant states
如果trajectories可以仅仅从指定的states开始，我们感兴趣的仅仅是给定policy下states的value，on-policy trajectory sampling可以完全跳过给定policy下不能到达的states。这些states跟prediction问题无关。对于control问题，目标是找到optimal policy而不是evaluating一个给定的policy，可能存在无论从哪个start states开始都无法到达的states，所以就没有必要给出这些无关states的action。我们需要的是一个optimal partial policy，对于relevant states，给出optimal policy，而对于irrelevant states，给出任意的actions。
找到这样一个policy，按道理来说需要访问所有的state-action pairs无数次，包括那些无关状态。[Korf](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.137.1955&rep=rep1&type=pdf)证明了在满足以下条件的时候，能够在很少访问relevant states甚至不访问的时候，找到这样一个policy。
1. goal state的初始value为$0$
2. 存在至少一个policy保证从任何start state都能到达goal state
3. 所有到达的非goal states的reward是严格为负的
4. 所有states的初始value要大于等于optimal values（可以设置为$0$，一定比负值大）

如果满足这些条件，RTDP一定会收敛到relevant states的optimal policy。

## 决策时进行planning
### backgroud planning
进行planning至少有两种方法。第一种是已经介绍的DP和Dyna这些算法，使用planning基于从model得到的simulated experience不断的改进policy和value function。通过比较某一个state处不同state-action pairs value值的大小选择action。在action被选择之前，planning更新所有的$Q$值。这里planning的结果被很多个states用来选择action。这种planning叫做background planning。

### decision-time planning
第二种方法是使用planning输出单个state的action，遇到一个新的state $S_t$，输出是单个的action $A_t$，然后再下一个时间步根据$S_{t+1}$继续计算$A_{t+1}$。最简单的一个例子是当只有states value可以使用的时候，通过比较model预测每一个action能够到达的后继state的value（也就是使用after state value）选择相应的action，。这种方法叫做decision-time planning。
事实上，decision-time planning和background planning的流程是一样的，都是使用simulated experience到backup values再到policy。只不过decision-time planning只是对当前可访问的单个state和action的value和policy进行planning。在许多decision-time planning的过程中，用于选择当前state对应的action时，使用到的value和policy用过以后都被丢弃了，这并不会造成太大的计算损失，因为在大部分任务中很多states在短时间内都不会被再次访问到。

### 应用场景
decision-time planning适用于不需要快速实时相应的场景，比如各种下棋。如果需要快速响应的，那么最好使用backgroud planning计算一个policy可以快速应用到新的states。

## 启发式搜索
AI中经典的state-space方法都是decision-time planning方法，统称为heuristic search。在启发式搜索中，会使用树进行搜索，近似的value function应用到叶子节点，进行backup到根节点（当前state），相应的backup diagram和optimal的expected updates类似。根据计算的backed-up值，选择相应的action之后，丢弃这些值。
传统的启发式搜索并不保存backup value，它更像是多步的greedy policy，目的是更好的选择actions。如果我们有一个perfect model以及imperfect action value function，搜索的越深结果越好。如果search沿着所有可能的路一直到episode结束，那么imperfect value function的结果被抵消了，选出来的action也是optimal。如果搜索的深度$k$足够深，$\gamma^k$接近于$0$,那么找到的action也是近乎于最优的。当然，搜索的深度越深，需要的计算资源就越多。
启发式搜索的的updates集中在current state，它的search tree集中在接下来可能的states和actions，这也是它的结果为什么很好的原因。在某些特殊的情况下，我们可以将具体的启发式搜索算法构建出一个tree，自底向上的执行one-step update，如下图所示：
![heuristic_search_tree](heuristic_search_tree.png)
如果update是有序的，并且使用tabular representation，整个updates可以看成深搜，所有的state-space搜索可以看成很多个one-step updates的组合。我们得出了一个结论，搜索深度越深，性能越好的原因不是multistep updates的使用，因为它实际上使用的是多个one-step update。真正的原因是更新都集中在current state downstream的states和actions上，所有的计算都集中在candidate actions相关的states和actions上。

## Rollout算法

### 什么是Rollout算法
Rollout算法是将Monte Carlo Control应用到从current state开始的simulate trajectories上的decision-time planning算法。Rollout算法根据给定的policy，这个policy叫做rolloutpolicy，从当前state可能采取的所有action开始生成很多simulated trajectories，对得到的returns进行平均估计aciton values。当action value估计的足够精确的时候，选择最大的那个action执行。
和MC Control的区别在于，MC Control的目的是估计整个action value function $q_{\pi}$或者$q_{\*}$，而Rollout算法的目的是对于每一个current state，在一个给定policy下估计每一个可能的action的value。Rollout是decision-time planning算法，计算完相应的estimate action value之后，就丢弃它们。

### Rollout算法做了什么
Rollout算法和policy iteration差不多。在policy improvement theorem理论中，如果在一个state处采取新的action，它的value要比原来的value高，那么就说这个新的policy要比老的policy好。Rollout算法在每个current state处，估计不同的state action value，然后选择最好的，其实就相当于one-step的policy iteration，或者更像on-step的asynchronous value iteration。
也就是说，rollout算法的目的是改进rollout policy，而不是寻找最优的policy。Rollout算法非常有效，但是它的效果也取决于rollout policy，roloout policy越好，最后算法生成的policy就越好。

### 如何选择好的rollout policy
更好的rollout policy也就需要更多的资源，因为是decision-time算法，时间约束一定要满足，rollout算法的计算时间取决于每一个decision需要选择的action数量，sample trajectories的长度，rollout policy做决策的事件，以及足够的sample trajectories的数量。接下来给出几种方法去权衡这些影响因素：
第一个方法，MC trials都是独立的，所以可以使用多个分开的处理器运行多个trials。第二个方法是在simulated trajectories结束之前截断，通过一个分类评估函数对truncated returns进行修正。第三个方法是剪枝，剪掉那些不可能是最优的actions，或者那些和当前最优结果没啥差别的acitons。

### rollout算法和learning算法的关系
Rollout算法并不是learning算法，因为它没有保存values和polices。但是rollout算法具有rl很多好的特征。作为MC Control的应用，他们使用sample trajectories，避免了DP的exhausstive sweeps，同时不需要使用distributin models，使用sample models。最后，rollout算法还使用了policy improvement property，即选择当前estimate action values最大的action。

## Monte Carlo Tree Search
MCTS是decision-time planning算法，实际上，MCTS是一个rollout算法，它在上一节介绍的rollout算法上，加上了acucumulating value estimates的均值。MCTS是AlphaGO的基础算法。
每到达一个新的state，MCTS根据state选择action，到达新的state，再选择action，持续下去。在大多数情况下，simulated trajectories使用rollout policy生成actions。当model和rollout policy都不需要大量计算的时候，可以在短时间内生成大量simulated trajectories。只保留在接下来的几步内最后可能访问到的state-action pairs的子集，形成一棵树，如下图所示。任意simulated trajectory都会经过这棵树，并且从叶子节点退出。在tree的外边，使用rollout policy选择actions，在tree的内部使用一个新的policy，称为tree policy，平衡exploration和exploitation。Tree policy可以使用$\epsilon$-greedy算法。
![mcts](mcts.png)
MCTS总共有四个部分，一直在迭代进行，第一步是Selection，根据tree policy生成一个episode的前半部分；第二步是expansion，从选中的叶子节点处的上一个节点探索其他没有探索过的节点；第三步是simulation，从选中节点，或者第二步中增加的节点处，使用rollout policy生成一个episode的后半部分；第四步是backup，从第一二三步得到的episode进行backup。这四步一直迭代下去，等到资源耗尽，或者没有time的时候，就退出，然后根据生成的tree中的信息选择相应的aciton，比如可以选择root node处action value最大的action，也可以选择最经常访问的action。
MCTS是一种rollout算法，所以它拥有online，incremental，sample-based和policy improvement等优点。同时，它保存了tree边上的estimate action value，并且使用sample update进行更新。优势是让trivals的初始部分集中在之前simulated的high-return trajectories的公共部分。然后不断的expanding这个tree，高效增长相关的action value table，通过这样子，MCTS避免了全局近似value funciton，同时又能够利用过去的experience指定exploraion。

## 参考文献
1.《reinforcement learning an introduction》第二版

