---
title: bayesian classifier bayesian networks
date: 2019-01-06 14:32:16
tags:
 - 人工智能
 - 概率图模型
 - 推理
 - 贝叶斯网络
 - 生成式模型
 - 概率论与统计
 - 贝叶斯分类器
categories: 机器学习
mathjax: true
---

## 概述
朴素贝叶斯分类假设属性之间是条件独立的，而在实际中，属性与属性之间难免会有一定的相关性，而不是完全独立，这时就不能用朴素贝叶斯分类器了。
贝叶斯网络不要求给定类的所有属性都条件独立，而是允许一些属性条件独立，一些属性不条件独立，它可以表现独立和条件独立之间的关系，然后根据这些关系计算后验概率。
概括的来说，本文的内容可以分为以下几个部分：
1. 首先定义了贝叶斯网络的语法(syntax)和语义(semantics)，并且展示了如何用贝叶斯网络表示不确定知识。贝叶斯网络是一个有向无环图，节点代表随机变量，边代表因果关系。贝叶斯公式的两个意义，一个是数值意义，用贝叶斯网络表示全概率分布，另一个是拓扑意义，给定某个节点的父节点，这个节点条件独立于所有它的非后裔节点，或者给定某个节点的马尔科夫毯，这个节点条件独立于所有其他节点。
2. 条件独立的有效表示，噪音或模型表示离散型父节点和离散型子节点之间的关系，用参数化模型表示连续型父节点和连续型子节点之间的关系，用probit模型或者logit模型表示连续型父节点和离散型子节点之间的关系。
3. 贝叶斯精确推理计算后验分布的两种方法，一种是枚举推理，一种是消元法。
4. 因为精确推理的复杂度太高了，没有实际应用价值，给出了估计推理的方法，直接采样，拒绝采样，以及可能性加权，还有另一类采样方法，蒙特卡洛算法，主要介绍了吉布森采样，大概就是这些。

## 贝叶斯网络的定义
我们可以根据联合概率分布(full joint probability distribution)算出任何想要的概率值（边缘分布什么的），但是随着随机变量个数的增加，联合概率分布可能会变得特别大。此外，一个一个的指定可能世界中的概率是不可行的。
如果在联合概率中引入独立和条件独立，会显著的减少定义联合概率分布所需要的概率。所以这节就介绍了贝叶斯网络来表示变量之间的依赖关系。本质上贝叶斯网络可以表示任何联合概率分布，而且在很多情况下是非常精确地表示。一个贝叶斯网络是一个有向图，图中的节点包含量化后的概率信息。具体的说明如下：
1. 每一个节点对应一个随机变量，这个随机变量可以是离散的也可以是连续的。
2. 有向边或者箭头连接一对节点。如果箭头是从节点$X$到节点$Y$，那么节点$X$称为节点$Y$的父节点。图中不能有环，因此贝叶斯网络是一个有向无环图(directed acyclic graph,DAG)。
3. 每一个节点$X\_i$有一个条件概率分布$P(x\_i|Parents(X\_i))$量化(quantifiy)父节点对其影响。

网络的拓扑，即节点和边的集合，指定了条件概率分布之间的关系。箭头的直观意义是节点$X$对节点$Y$有直接的影响，$Y$发生的原因是其父节点的影响。通常对于一个领域(domain)的专家来说，指出该域受哪些因素的直接影响要比直接给出它的概率值简单的多。一旦贝叶斯网络的拓扑结构定了，给出一个变量的父节点，我们仅仅需要给出每个节点的条件概率分布。我们能看出，拓扑和条件概率的组合能计算出所有变量的联合概率分布。

## 贝叶斯网络的示例
### 牙疼和天气
给定一组随机变量牙疼(Toothache)，蛀牙(Cavity)，拔牙(Catch)和天气(Weather)。Weather是独立于另外三个随机变量的，此外，给定Cavity，Catch和Toothache是条件独立的，即给定Cavity，Catch和Toothache是相互不受影响的，如下图所示。正式的：给定Cavity，Toochache和Catch是条件独立的，图中Toothache和Catch之间缺失的边体现出了条件独立。直观上，网络表现出Cavity是Toothache和Catch发生的直接原因，然而在Toothache和Catch之间没有直接的因果关系。 
![figure 14.1]() 
### 警报和打电话 
我家里有一个新安装的防盗警报(burglar alarm)，这个警报对于小偷的检测是相当可靠的，但是也会对偶然发生的微小的地震响应。我有两个邻居(Mary和John)，他们听到警报后会打电话给我。John有时会把电话铃和警报弄混了，也会打电话。Mary听音乐很大声，经常会错过警报。现在给出John或者Mary谁是否打电话，估计警报响了的概率。 
![figure 14.2]() 
该例子的贝叶斯网络如上图所示。该网络体现了小偷和地震两个因素会直接影响警报响的概率，但是John和Mary会不会打电话只取决于警报有没有响。贝叶斯网络展示出了我们的假设，即John和Mary不直接观察小偷有没有来，也不直接观察小的地震是否，也不受之前是否打过电话的影响。上图中的条件概率分布以一个条件概率分布表(conditional probability table,CPT)的形式展现了出来。这个表适合离散型的随机变量，但是不适合连续性随机变量。没有父节点的节点只有一行，用来表示随机变量的可能取值的先验概率(prior probabilities)。
注意到这个网络中没有节点对应Mary听音乐很大时，也没有节点对应John把电话铃声当成了警报。事实上这些因素都被包含在和边Alarm到JohnCalls和MaryCall相关的不确定性中了，概率包含了无数种情况可能让警报失灵（停电，老鼠咬坏了，等等）或者John和Mary没有打电话的原因（吃饭去了，午睡了，休假了等等），这些不确定性都包含在了概率中了。

## 贝叶斯网络的意义
有两种方式可以理解贝叶斯网络的意义。第一个是一种数值化的意义(numerical semantics)，把它当成联合概率分布的一种表示形式。第二个是一种拓扑的意义(topological semantics)，将它看成条件独立的一种编码方式。事实上，这两种方式是等价的，只不过第一种方式有助于理解如何构建贝叶斯网络，而第二种方式更有助于设计推理过程。

### 贝叶斯网络表示联合分布(Representing the full joint distribution)
#### 定义
一个贝叶斯网络是一个有向无环图，并且每个节点都有一个数值参数。数值方式给出这个网络的意义是，它代表了所有变量的联合概率分布。之前说过节点上的值代表的是条件概率分布$P(X\_i|Parents(X\_i)$，这是对的，但是当赋予整个网络意义以后，这里我们认为它们只是一些数字$\theta(X\_i|Parents(X\_i)$。
联合概率中的一个具体项(entry)表示的是每一个随机变量取某个值的联合概率，如$P(X\_1=x\_1 \wedge \cdots\wedge X\_n = x\_n)$，缩写为$P(x\_1,\cdots,x\_n)$。这个项的值可以通过以下公式进行计算：
$$P(x\_1,\cdots,x\_n) = \prod\_{i=1}^n \theta(x\_i|parents(X\_i)),$$
其中$parents(X\_i)$表示节点$X\_i$在$x\_1,\cdots,x\_n$中的父节点。因此，联合概率分布中的每一项都可以用贝叶斯网络中某些条件概率的乘积表示。从定义中可以看出，很容易证明$\theta(x\_i|parents(X\_i))$就是条件概率$P(x\_i|parents(X\_i))$，因此，我们可以把上式写成：
$$P(x\_1,\cdots,x\_n) = \prod\_{i=1}^n P(x\_i|parents(X\_i)),$$
换句话说：根据上上个式子定义的贝叶斯网络的意义，我们之前叫的条件概率表真的是条件概率表。（这句话。。。）
#### 示例
我们可以计算出警报响了，但是没有小偷或者地震发生，John和Mary都打电话了的概率。即计算联合分布$P(j,m,a,\neg b, \neg e)$（使用小写字母表示变量的值）：
\begin{align\*}
P(j,m,a,\neg b, \neg e) &=P(j|a)P(m|a)P(a|\neg b \wedge \neg e)P(\neg b)P(\neg e)\\\\
&=0.90\times 0.70\times 0.001 \times 0.999 \times 0.998\\\\
&=0.000628
\end{align\*}

#### 构建贝叶斯网络(Constructing Bayesian networks)
上面给出了贝叶斯网络的一种意义，接下来给出如何根据这种意义去构建一个贝叶斯网络。确定的条件独立可以用来指导网络拓扑的构建。首先，我们把联合概率的项用乘法公式写成条件概率表示：
$$P(x\_1,\cdots,x\_n) = P(x\_n|x\_{n-1},\cdots,x\_1)P(x\_{n-1},\cdots,x\_1)$$
接下来重复这个过程，将联合概率(conjunctive probability)分解成一个条件概率和一个更小的联合概率。最后得到下式：
\begin{align\*}
P(x\_1,\cdots,x\_n) &= P(x\_n|x\_{n-1},\cdots,x\_1)P(x\_{n-1}|,x\_{n-2}\cdots,x\_1)\cdots P(x\_2|x\_1)P(x\_1)\\\\
&= \prod\_{i=1}^nP(x\_i|x\_{i-1},\cdots,x\_1)
\end{align\*}
这个公式被称为链式法则，它对于任意的随机变量集都成立。对于贝叶斯网络中的每一个变量$X\_i$，如果给定$Parents(X\_i) \subset \{X\_{i-1},\cdots,X\_1\}$（每一个节点的序号应该和图结构的偏序结构一致），那么有：
$$P(x\_1,\cdots,x\_n) = \prod\_{i=1}^n P(x\_i|parents(X\_i)),$$
将它和上式对比，得出：
$$P(X\_i|X\_{i-1},\cdots,X\_1) = P(X\_i|Parents(X\_i).$$
这个公式成立的条件是给定每个节点的父节点，它条件独立于所有它的非父前置节点。这里给出一个生成贝叶斯网络的方式：
1. 节点：首先，确定需要对领域建模所需要的随机变量集合。对它们进行排序：$\{X\_1,\cdots,X\_n\}$，任意顺序都行，但是如果随机变量的因(causes)在果(effects)之前，最终的结果会更加紧凑。
2. 边：从$i = 1$到$n$，
  - 从$X\_1,\cdots,X\_{i-1}$中选出$X\_i$的最小父节点集合。
  - 对于每一个父节点，插入一条从父节点到$X\_i$的边。
  - 写下条件概率表，$P(X\_i| Parents(X\_i))$。

直观上，$X\_i$的父节点应该包含$X\_1,\cdots,X\_{i-1}$中所有直接影响$X\_i$的节点。因为每一个节点都只和它前面的节点相连，这就保证了每个网络都是无环的(acyclic)。此外，贝叶斯网络还不包含冗余的概率值，如果有冗余值，就会产生不一致：不可能生成一个违反概率论公理的贝叶斯网络。

#### 紧凑性和节点顺序(Compactness and node ordering)
##### 紧凑性(compactness)
因为不包含冗余信息，贝叶斯网络会比联合概率分布更加紧凑，这让它能够处理拥有很多变量的任务。贝叶斯网络的紧凑性是稀疏(sparse)系统或者局部结构化(local structured)系统普遍拥有的稀疏性的一个例子。在一个局部结构化系统中，每一个子部件仅仅和有限数量的其他部件进行交互，而不用管整个系统。局部结构化的复杂度通常是线性增加的而不是指数增加的。在贝叶斯网络中，一个随机变量往往最多受$k$个其他随机变量直接影响，这里的$k$是一个常数。为了简化问题，我们假设有$n$个布尔变量，指定一个条件概率表所需要的数字最多是$2^k$个，整个网络则需要$n2^k$个值；作为对比，联合概率分布需要$2^n$个值。举个例子，如果我们有$n=30$个节点，每一个节点至多有五个父节点(k=5)，那么贝叶斯网络只需要$960$个值，而联合概率分布需要超过十亿个值。
但是在某些领域，可能每一个节点都会被所有其他节点直接影响，这时候网络就成了全连接的网络(fully connected)，它和联合概率分布需要同样多的信息。有时候，增加一条边，也就是一个依赖关系，可能会对结果产生影响，但是如果这个依赖很弱(tenuous)，添加这条边的花费比获得的收益还要大，那么就没有必要加这条边了。比如，警报的那个例子，如果John和Mary感受到了地震，他们认为警报是地震引起的，所以就不打电话了。是否添加Earthquake到JohnCalls和MaryCalls这两条边取决于额外的花费和得到更高的警报率之间的关系。

##### 节点顺序(node ordering)
即使在一个局部结构化的领域，只有当我们选择好的节点顺序的时候，我们才能得到一个紧凑的贝叶斯网络。考虑警报的例子，我们给出下图：
![figure 14.3]()
Figure 14.2和Figure 14.3两张图中的三个贝叶斯网络表达的都是同一个联合分布，但是Figure 14.3中的两张图没有表现出来条件独立，尤其是Figure 14.3(b)中的贝叶斯网络，它需要用和联合分布差不多相同个数的值才能表现出来。可以看出来，节点的顺序会影响紧凑性。

### 贝叶斯网络中的条件独立 (Conditional independence relations in Bayesion networks)
贝叶斯网络的一个数值意义("numerical" semantics)是用来表示联合概率分布。根据这个意义，给定每个节点的父节点，使得每一个节点条件独立于它的父节点之外的节点，我们能构建一个贝叶斯网络。此外我们也可以从用图结构编码整个条件独立关系的拓扑意义出发，然后推导出贝叶斯网络的数值意义。拓扑语义说的是给定每个节点的父节点，则该节点条件独立于所有它的非后裔(non-descendants)节点。举例来说，Figure 14.2的警报例子中，给定alarm后，JohnCalls独立于Burglary,Eqrthquake和MaryCalls。如图Figuree 14.4(a)中所示。从条件独立断言(assertions)和网络参数$\theta(x\_i|parents(X\_i))$就是条件概率$P(x\_i|parents(X\_i))$的解释中，联合概率可以计算出来。在这种情况下，数值意义和拓扑语义是相同的。
另一个拓扑意义的重要属性是：给定某个节点的马尔科夫毯(Markov blanket)，即节点的父节点，子节点，子节点的父节点，这个节点条件独立于所有其他的节点。如图Figure 14.4(b)所示。

## 条件分布的高效表示(Efficient representation of conditional distributions)
即使每个节点有$k$个父节点，一个节点的CPT还需要$O(2^k )$，最坏的情况下父节点和子节点是任意连接的。一般情况下，这种关系可以用符合一些标准模式(standard pattern)的规范分布(canonical distribution)表示，这样子就可以仅仅提供分布的一些参数就能生成整个CPT。
最简单的例子是确定性节点(deterministic node)。一个确定性节点的值被它的父节点的值精确确定。这个确定性关系可以是逻辑关系：父节点是加拿大，美国和墨西哥，子节点是北美洲，它们之间的关系是子节点是父节点所在的洲。这个关系也可以是数值型的，一条河的流量是流入它的流量减去流出它的流量。
不确定关系通常称为噪音逻辑关系(noisy logical relationships)。一个例子是噪音或(noisy-OR)，它是逻辑或的推广。在命题逻辑中，当且仅当感冒(Cold)，流感(Flu)或者疟疾(Malaria)是真的时候，发烧(Fever)才是真的。噪音或模型允许不确定性，即每一个父节点都有可能让子节点为真，可能父节点和子节点之间的关系被抑制了(inhibited)，可能一个人感冒了，但是没有表现出发烧。这个模型做了两个假设。第一个，它假设所有的原因都被列了出来，有时候会加一个节点(leak node)包含所有的其他原因(miscellaneous causes)。第二个，抑制每一个父节点和子节点之间的原因是独立的，比如抑制疟疾产生发烧和抑制感冒产生发烧的原因是独立的。所以，当且仅当所有的父节点都是假的时候，发烧才一定不会发生。给出以下的假设：
$q\_{cold} = P(\neg fever| cold,\neg flu, \neg malaria) = 0.6$
$q\_{flu} = P(\neg fever|\neg cold, flu, \neg malaria) = 0.2$
$q\_{malaria} = P(\neg fever|\neg cold,\neg flu, malaria) = 0.1$
根据这些信息，以及噪音或的假设，整个CPT可以被创建。一般的规则是：
$P(x\_i|parents(X\_i)) = 1 - \prod\_{j:X\_j=ture} q\_j.$
最后生成如下的表：

|Cold|Flu|Malaria|P(Fever)|P($\neg$Fever)|
|:--: | :--: | :--: |:--:|:--:|
|F | F | F | $0.0$ | $1.0$ | 
|F | F | T | $0.9$ | $0.1$ | 
|F | T | F | $0.8$ | $0.2$ | 
|F | T | T | $0.98$ | $0.1\times 0.2=0.02$| 
|T | F | F | $0.4$ | $0.6$ | 
|T | F | T | $0.94$| $0.6\times 0.1 = 0.06 $| 
|T | T | F | $0.88$ | $0.5\times 0.2 = 0.12 $| 
|T | T | T | $0.988$ | $0.6\times 0.2\times 0.1 = 0.012$ | 
对于这个表，感觉自己一直有点转不过来圈。就是有症状不一定发烧，也可能不发烧，没有症状一定不发烧。什么时候不发烧呢，只有某个症状表现出来不发烧，如果多个症状的话，直接把有症状表现但不发烧的概率相乘。
一般情况下，噪声逻辑模型中，有$k$个父节点的变量可以用$O(k)$个参数表示而不是$O(2^k )$去表示整个CPT。这让访问(assessment)和学习(learning)更容易了。

### 连续性随机变量的贝叶斯网络(Bayesian nets with continuous variables)
#### 常用方法
现实中很多问题都是连续型的随机变量，它们有无数可能的取值，所以显式的指定每一个条件概率行不通。常用的总共有三种方法，第一个可能的方法是离散(discretization)连续型随机变量，将随机变量的可能取值划分成固定的区间。比如，温度可以分成，小于$0$度的，$0$度到$100$度之间的，大于$100$度的。离散有时候是可行的，但是通常会造成精度的缺失和非常大的CPT。第二个方法也是最常用的方法是通过指定标准概率密度函数的参数，比如指定高斯分布的均值和方差。第三种方法是非参数化(nonparametric)表示，用隐式的距离去定义条件分布。
#### 示例
一个同时拥有离散型和随机性变量的网络被称为混合贝叶斯网络(hybrid Bayesian network)。为了创建这样一个网络，我们需要两种新的分布。一种是给定离散或者连续的父节点，子节点是连续型随机变量的条件概率，另一种是给定连续的父节点，子节点是离散型随机变量的条件概率。
##### 连续型子节点
![figure 14.5]()
考虑Figure 14.5的例子，一个顾客买了一些水果，买水果的量取决取水果的价格(Cost)，水果的价格取决于收成(Harvest)和政府是否有补助(Subsidy)。其中，Cost是连续型随机变量，他有连续的父节点Harvest和离散的父节点Subsidy，Buys是离散的，有一个连续型的父节点Cost。
对于变量Cost，我们需要指定条件概率$P(Cost|Subsidy,Harvest)$。离散的父节点通过枚举(enumeration)来表示，指定$P(Cost|subsidy,Harvest)$和$P(Cost|\neg subsidy,Harvest)$。为了表示Harvest，可以指定一个分布来表示变量Cost的值$c$取决于连续性随机变量Harvest的值$h$。换句话说，将$c$看做一个$h$的函数，然后给出这个函数的参数即可，最常用的是线性高斯分布。比如这里，我们可以用两个不同参数的高斯分布来表示有补贴和没补贴时Harvest对Cost的影响：
$$P(c|h, subsidy) = N(a\_th+b\_t,\sigma\_t^2 )(c) = \frac{1}{\sigma\_t \sqrt{2\pi} }e^{- \frac{1}{2}(\frac{c-(a\_th+b\_t)}{\sigma\_t})^2 } $$
$$P(c|h,\neg subsidy) = N(a\_fh+b\_t,\sigma\_f^2 )(c) = \frac{1}{\sigma\_f \sqrt{2\pi} }e^{- \frac{1}{2}(\frac{c-(a\_fh+b\_f)}{\sigma\_f})^2 } $$
所以，只需要给出$a\_t,b\_t,\sigma\_t,a\_f,b\_f,\sigma\_f$这几个参数就行了，Figure 14.6(a)和(b)就是一个示例图。注意到坡度(slope)是负的，因为随着供应的增加，cost在下降，当然，这个线性模型只有在harvest在很小的一个区间内才成立，而且cost有可能为负。假设有补贴和没补贴的两种可能性相等，是$0.5$，那么就有了Figure 14.6(c)的图$P(c|h)$。
##### 连续型父节点
当离散型随机变量有连续型父节点时，如Figure 14.5中的Buys节点。我们有一个合理的假设是：当cost高的时候，不买，cost底的时候，买，在中间区域买不买是一个变化很平滑的概率。我们可以把条件分布当成一个软阈值函数(soft-threshold)，一种方式是用标准正态分布的积分(intergral)。
$$\Phi(x) = \int\_{-\infty}^{x} N(0,1)(x)dx$$
给定Cost买的概率可能是:
$$P(buys|Cost = c) = \Phi((-x+\nu)/ \sigma))$$
其中cost的阈值在$\nu$附近，阈值的区域和正比于$\sigma$，当价格升高的时候，买的概率会下降。这个probit distribution模型如Figure 14.7(a)所示。
另一个可选择的模型是logit distribution，使用logistic function $1/(1+e^{-x} )$来生成一个软阈值：
$$P(buys|Cost = c) = \frac{1}{1+exp(-2\frac{-c+u}{\sigma})}.$$
如Figure 14.7(b)所示，这两个分布很像，但是logit有更长的尾巴。probit更符合实际情况，但是logit数学上更好算。它们都可以通过对父节点进行线性组合推广到多个连续性父节点的情况。

## 贝叶斯网络的精确推理(Exact inference in bayesian networks)
概率推理系统的基本任务就是给出一些观察到的事件，即给证据变量(evidence variable)赋值，然后计算一系列查询变量(query variable)的后验概率。我们用$X$表示查询变量，用$\mathbf{E}$表示证据变量$E\_1,\cdots,E\_m$的集合，$\mathbf{e}$是一个特定的观测事件，$\mathbf{Y}$表示既不是证据变量，也不是查询变量的变量$Y\_1,\cdots,Y\_l$的集合（隐变量,hidden variables)。变量的所有集合是$\mathbf{X}=\{X\}\cup \mathbf{E}\cup \mathbf{Y}$。一个典型的查询是求后验概率$P(X|\mathbf{e})$。
在这一节中主要讨论的是计算后验概率的精确算法以及这些算法的复杂度。事实上，在一般情况下精确推理的复杂度都是很高的，为了降低复杂度，就只能进行估计推理(approximate inference)了，这个会在下一节中介绍到。

### 枚举实现精确推理(Inference by enumeration)
任何条件概率都可以用联合概率分布的项相加得到，即：
$$P(X|\mathbf{e}) = \alpha P(X,\mathbf{e}) = \alpha \sum\_{\mathbf{y}}P(X,\mathbf{e},\mathbf{y})$$
贝叶斯网络给出了所有的联合概率分布，任何项$P(x,\mathbf{e},\mathbf{y})$都可以用贝叶斯网络中的条件概率的乘积表示出来。比如警报例子中的查询$P(Burglary|JohnCalls=true,MaryCalls=true)$。隐变量是Earthquake和Alarm，我们可以算出：
$$P(B|j,m) = \alpha P(B,j,m) = \alpha \sum\_{e}\sum\_{a}P(B,j,m,e,a).$$
贝叶斯网络已经给出了所有CPT项的表达式，比如当Burglary = true时：
$$P(b|j,m) = \alpha \sum\_e\sum\_aP(b,j,m,e,a) = \alpha \sum\_e\sum\_aP(b)P(e)P(a|b,e)P(j|a)P(m|a).$$
为了计算这个表达式，我们得计算一个四项的加法，分别是e为true和false,a为true和false对应的$P(b,j,m)$的值，每一项都是五个数的乘法。最坏的情况下，所有的变量都用到了，那么拥有$n$个布尔变量的贝叶斯网络的时间复杂度是$O(n2^n )$。我们可以做一些简化，将一些重复的计算保存下来，比如将上面的式子变成：
$$P(b|j,m) = \alpha \sum\_e\sum\_aP(b,j,m,e,a) = \alpha P(b) \sum\_eP(e)\sum\_aP(a|b,e)P(j|a)P(m|a).$$
这样子可以按照顺序进行计算，具体的计算过程如Figure 14.8所示。这种算法叫做ENUMERATION-ASK，它的空间复杂度是线性的，但是它的事件复杂度是$O(2^n )$比$O(n2^n )$要好，却仍然是实际上不可行的。（这里我理解的是$O(2^n )$而不是$O(n2^n )$的原因是，总共有$n$个布尔变量，所以总共有$2^n $个可能的取值，每次算一个，存一个，而原来的是算完之后不存。）
事实上，Figure 14.8中的计算过程还有很多重复计算，比如$P(j|a)P(m|a)$和$P(j|\neg a)P(m|\neg a)$这两项被计算了两次。我原来在想这里是不是和上面一段说的冲突了，事实上是没有的，这$2^n $个值，其中可能会有$P(b,j,m,e,a)$和$P(b,j,m,e,\neg a)$，这两个概率中都用到了$P(j|a)P(m|a)$，但是这里就会计算两次，事实上有很多值都会被重复计算很多次。下面就介绍一个避免这种运算的方法。

### 消元法(The variable elimination algorithm)
上面问题的解决思路就是保存已经计算过的值，实际上这是一种动态规划。还有很多其他方法可以解决这个问题，这里介绍了最简单的消元算法。消元法对表达式进行从右至左的计算，而枚举法是自底向上的。所有的中间值被报存起来，最对和每个变量有关的表达式进行求和。例如对于下列表达式：
$$P(B|j,m) = \alpha \underbrace{P(B)}\_{f\_1(B)} \sum\_e\underbrace{P(e)}\_{f\_2(E)} \sum\_a\underbrace{P(a|B,e)}\_{f\_3(A,B,E)} \underbrace{P(j|a)}\_{f\_4(A)} \underbrace{P(m|a)}\_{f\_5(A)}.$$
表达式的每一部分都是一个新的因子，每一个因子都是由它的参数变量(argument variables)决定的矩阵，参数变量指定的取值是没有固定的变量。比如因子$f\_4(A)$和$f\_5(A)$对应$P(j|a)$和$P(m|a)$的表达式只取决于$A$的值因为$J$和$M$在这个查询中都是固定的。它们都是两个元素的向量：
$$f\_4(A) = \begin{pmatrix}P(j|a)\\\\P(j|\neg a)\end{pmatrix} = \begin{pmatrix}0.90\\\\0.05\end{pmatrix}$$
$$f\_5(A) = \begin{pmatrix}P(m|a)\\\\P(m|\neg a)\end{pmatrix} = \begin{pmatrix}0.70\\\\0.01\end{pmatrix}$$
$f\_3(A,B,E)$是一个$2\times 2\times 2$的矩阵。用因子表达的话，查询的表达式变成了：
$$P(B|j,m) = \alpha f\_1(B)\times \sum\_ef\_2(E)\times \sum\_af\_3(A,B,E)\times f\_4(A)\times f\_5(A)$$
其中$\times$不是普通的矩阵乘法，而是对应元素相乘(pointwise product)。整个表达式的计算过程可以看成从右到左变量相加的过程，将现有的因子消去产生新的因子，最后只剩下一个因子的过程。具体的步骤如下：
首先先利用$f\_3,f\_4,f\_5$把变量$A$消掉，产生一个新的$2\times 2$的只含有变量$B$和$E$的新因子$f\_6(B,E)$：
\begin{align\*}
f\_6(B,E) &= \sum\_af\_3(A,B,E)\times f\_4(A) \times f\_5(A)\\\\
&= (f\_3(a,B,E)\times f\_4(a) \times f\_5(a)) + (f\_3(\neg a,B,E)\times f\_4(\neg a)\times f\_5(\neg a)
\end{align\*}
这样目标变成了：
$$P(B|j,m) = \alpha f\_1(B)\times \sum\_ef\_2(E)\times \sum\_af\_6(B,E)$$
利用$f\_2,f\_6$消去$E$：
\begin{align\*}
f\_7(B) &= \sum\_ef\_2(E)\times \sum\_af\_6(B,E)\\\\
& = f\_2(e)\times f\_6(B,e) + f\_2(\neg e)\times f\_6(B,\neg e) 
\end{align\*}
将表达式化成：
$$P(B|j,m) = \alpha f\_1(B)\times f\_7(B)$$
显然，根据这个表达式就可以计算出我们想要的结果了。上面的过程可以总结成两步，第一步是point-wise的因子乘法，第二步是利用因子的乘法进行消元。
#### 因子运算(Operations on factors)
两个因子$f\_1$和$f\_2$进行point-wise乘法运算产生新的因子(factor)$f$的变量是$f\_1$和$f\_2$变量的并，新的因子中的元素的值是$f\_1$和$f\_2$中对应项的积。假设两个因子有公共变量$Y\_1,\cdots,Y\_k$，那么就有：
$$f(X\_1,\cdots,X\_j,Y\_1,\cdots,Y\_k,Z\_1,\cdots,Z\_l)=f\_1(X\_1,\cdots,X\_j,Y\_1,\cdots,Y\_k)f\_2(Y\_1,\cdots,Y\_k,Z\_1,\cdots,Z\_l).$$
如果所有的变量都是二值化的，那么$f\_1$和$f\_2$各有$2^{j+l} $和$2^{l+k} $项，$f$有$2^{j+l+k} $项。比如，$f\_1(A,B),f\_2(B,C)$，那么point-wise乘法产生的$f\_3(A,B,C)=f\_1\times f\_2$有$8$项，如Figure 14.10所示。
![figure 14.10]()
根据图中给出的值，消去$f\_3(A,B,C)$中的$A$：
\begin{align\*}
f(B,C) &= \sum\_af\_3(A,B,C)\\\\
&= f\_3(a,B,C) + f\_3(\neg a,B,C)\\\\ 
&= \begin{pmatrix} 0.06&0.24\\\\0.42&0.28\end{pmatrix} + \begin{pmatrix}0.18&0.72\\\\0.06&0.04\end{pmatrix}\\\\
&= \begin{pmatrix}0.24&0.96\\\\048&0.32\end{pmatrix}
\end{align\*}
产生新的因子用的是pointwise乘法，消元用的是累乘。给定pointwise乘法和消元函数，消元算法就变得很简单，一个消元算法如Figure 14.11所示。
![figure 14.11]()

#### 变量顺序和变量相关性(Variable ordering and variable relevance)
Figure 14.11中的算法包含一个没有给出具体实现的排序函数Order()对要消去的变量进行排序，每一种排序选择都会产生一组有效的算法，但是不同的消元顺序会产生不同的中间因子。一般情况下，消元法的时间和空间复杂度是由算法产生的最大因子决定的，这个最大因子是由消元的顺序和贝叶斯网络的结构决定的，选取最优的消元顺序是很困难的，但是有一些小的技巧：总是消去让新产生的因子最小的变量。
另一个属性是：每一个不是查询变量或者证据变量的祖先变量都和这次查询无关，在实现消元算法的时候可以把这些变量都去掉。（具体的示例可以看第十四章，在$528$页）。

### 精确推理的复杂度(The complexity of exact inference)
贝叶斯网络的精确推理跟网络的结构有很大的关系。
Figure 14.2中警报贝叶斯网络中的复杂度是线性的。该网络中任意两个节点只有一条路径，这种网络称为单连接的(singly-connected)或者多树(polytrees)，这种结构有一个很好的属性就是：多树结构中精确推理的时间，空间复杂度对于网络大小来说都是线性关系，这里网络大小指的是CPT项的个数。如果每一个节点的父节点都是一个有界的常数，那么复杂度和节点数之间也是线性关系。
对于多连接(multiply connected)的网络，如Figure 14.12(a)所示，最坏情况下，即使每一个节点的父节点个数都是有界常数，消元法的时间和空间复杂度也都是指数级别的。因为贝叶斯网络的推理也是NP难问题。
![figure 14.12]()

### 聚类算法(clustering algorithms)
用消元法来计算单个的后验概率是简单而高效的，但是如果要计算网络中所有变量的后验概率是很低效的。例如：在单连接的网络中，每一个查询都是$O(n)$，总共有$O(n)$个查询，所以总共的代价是$O(n^2 )$。使用聚类算法(clustering algorithms)，代价可以降到$O(n)$，因此贝叶斯网络中的聚类算法已经被广泛商用。（这里不明白为什么？）。
聚类算法的基本思想是将网络中的一些节点连接成聚点(cluster nodes)，最后形成一个多树(polytree)结构。例如Figure 14.12(a)中的多连接网络可以转换成Figure 14.12(b)所示的多树，Sprinkler和Rain节点形成了SPrinkler+Rain聚点，这两个布尔变量被一个大节点(meganode)取代，这个大节点有四个可能的取值：$tt,tf,ft,ff$。一旦一个多树形式的网络生成了以后，就需要特殊的推理算法进行推理了，因为普通的推理算法不能处理共享变量的大节点，有了这样一个特殊的算法，后验概率的时间复杂度就是线性于聚类网络的大小。但是，NP问题并没有消失，如果消元需要指数级别的时间和空间复杂度，聚类网络中的CPT也是指数级别大小。

## 贝叶斯网络的估计推理(Approximate inference in bayesian networks)
因为多连接网络中的推理是不可行的，所以用估计推理取代精确推理是很有用的。这一节会介绍随机采样算法，也叫蒙特卡洛算法(Monte Carlo)，它的精确度取决于生成的样本数量。我们的目的是采样用于计算后验概率。这里给出了两类算法，直接采样(direct sampling)和马尔科夫链采样(Markov chain sampling)。变分法(variational methods)和循环传播(loopy propagation)将会在本章的最后进行介绍。
### 直接采样(Direct sampling methods)
任何采样算法都是通过一个已知的先验概率分布生成样本。比如一个公平的硬币，服从一个先验分布$P(coin) = <0.5,0.5 > $，从这个分布中采样就像抛硬币。
一个最简单的从贝叶斯网络中进行随机采样的方法就是：从没有证据和它相关的网络中生成事件，即按照拓扑顺序对每一个变量进行采样。如Figure 14.13所示的算法，每一个变量的采样都取决于前之前已经采样过了的父节点变量的值。按照Figure 14.13中的算法对Figure 14.12(a)中的网络进行采样，假设一个采样顺序是[Cloudy,Sprinkler,Rain,WetGrass]：
1. 从$P(Cloudy)=<0.5,0.5>$中采样，采样值是true；
2. 从$P(Sprinkler|Cloudy=true) = <0.1,0.9>$中采样，采样值是false； 
3. 从$P(Rain|Cloudy=true)=<0.8,0.2>$中采样，采样值是true；
4. 从$P(WetGrass|Sprinkler=false,Rain=true)=<0.9,0.1>$中采样，采样值是true；

这个例子中，PRIOR-SAMPLE算法返回事件[true,false,true,true]。可以看出来，PRIOR-SAMPLE算法根据贝叶斯网络指定的先验联合分布生成样本。假设$S\_{PS}(x\_1,\cdot,x\_n)$是PRIOR-SAMPLE算法生成的一个样本事件，从采样过程中我们可以得出：
$$S\_{PS}(x\_1,\cdots,x\_n) = \prod\_{i=1}^n P(x\_i|parents(X\_i))$$
即每一步采样都只取决于父节点的值。这个式子和贝叶斯网络的联合概率分布是一样的，所以，我们可以得到：
$$S\_{PS} = P(x\_1,\cdots,x\_n).$$
通过采样让这个联合分布的求解很简单。
![figure 14.13]()
事实上在任何采样算法中，结果都是通过对产生的样本进行计数得到的。假设生成了$N$个样本，$N\_{PS}(x\_1,\cdots,x\_n)$是样本集中的一个具体事件$(x\_1,\cdots,x\_n)$发生的次数。我们希望这个值比上样本总数取极限和采样概率$S\_{PS}$是一样的，即：
$$ lim\_{N\rightarrow \infty}\frac{N\_{PS}(x\_1,\cdots,x\_n)}{N} = S\_{PS}(x\_1,\cdots,x\_n) = P(x\_1,\cdots,x\_n).$$
例如之前利用PRIOR-SAMPLE算法产生的事件[true,false,true,true]，这个事件的采样概率是：
$$S\_{PS}(true,false,true,true) = 0.5 \times 0.9 \times 0.8 \times 0.9 = 0.324.$$
即当$N$取极限时，我们希望有$32.4\%$的样本都是这个事件。(这里为什么要用采样进行计算呢，我的想法是因为实际情况中，采样概率$S\_{PS}$是很难计算的，就通过不断的采样，计算出某个样本出现的概率。)
我们用$\approx$表示估计概率(estimated probability)在样本数量$N$取极限时和真实概率一样的估计，这叫一致(consistent)估计。比如，对于任意的含有隐变量的事件(partially spefified event)，$x\_1,\cdots,x\_m,m\le n$，会产生一个一致估计：
$$P(x\_1,\cdots,x\_m)\approx N\_{PS}(x\_1,\cdots,x\_m)/N.$$
这个事件的概率可以看成所有满足观测变量条件的样本事件（隐变量所有值都可以取）比上所有样本事件的比值。比如在Spinkler网络中，生成$1000$个样本，其中有$511$个样本的Rain=true，那么rain的估计概率就是$\hat{P}(Rain=true) = 0.511.$

#### 贝叶斯网络的拒绝采样(Rejection sampling in Bayesian networks)
##### 算法
![figure 14.14]()
拒绝采样(rejection sampling)利用容易采样的分布来生成难采样分布的样本，计算后验概率$P(X|\mathbf{e})$，算法流程如Figure 14.14所示，首先根据贝叶斯网络的先验分布生成样本，接下来拒绝(reject)那些和证据变量不匹配的结果，最后在剩下的样本中统计每个$X=x$出现的概率，估计$\hat{P}(X|\mathbf{e}).$
用$\hat{P}(X|\mathbf{e})$表示估计概率分布，利用拒绝采样算法的定义计算：
$$\hat{P}(X|\mathbf{e}) = \alpha N\_{PS}(X,\mathbf{e}) = \frac{N\_{PS}(X,\mathbf{e})}{N\_{PS}(\mathbf{e})}.$$
而根据$P(x\_1,\cdots,x\_m)\approx N\_{PS}(x\_1,\cdots,x\_m)/N$，就有：
$$\hat{P}(X|\mathbf{e}) = \alpha N\_{PS}(X,\mathbf{e}) = \frac{N\_{PS}(X,\mathbf{e})}{N\_{PS}(\mathbf{e})} =  \frac {P(X,\mathbf{e})}{P(\mathbf{e})} = P(X|\mathbf{e}).$$
所以，拒绝采样产生了真实概率的一个一致估计(consistent estimate)，但是这个一致估计和无偏估计还不一样。

##### 示例
举一个例子来说明，假设我们要估计概率$P(Rain|Sprinkler=true)$，生成了$100$个样本，其中$73$个是$Sprinkler=false$，$27$是$Sprinkler=true$，这$27$个中有$8$个$Rain=true$，有$19$个$Rain=false$，因此：
$$P(Rain|Sprinkler=true)\approx NORMALIZE \lt\lt 8,19>> = <0.296,0.704>.$$
正确答案是$<0.3,0.7>$，可以看出来，估计值和真实值差的不多。生成的样本越多，估计值就会和正确值越接近，概率的估计误差和$1/\sqrt{n}$成比例，$n$是用来估计概率的样本数量。

##### 不足 
拒绝采样最大的问题是它拒绝了很多样本，随着证据变量的增加，和证据$\mathbf{e}$一致的样本指数速度减少，所以这个方法对于复杂的问题是不可行的。拒绝假设和现实生活中条件概率是很像的，比如估计观测到晚上天空是红的，第二天下雨的概率$P(Rain|RedSkyAtNight=ture)$，这个条件概率的估计就是根据日常生活的观察实现的。但是如果天空很少是红的，就需要很长时间才能估计它的值，这就是拒绝假设的缺点。

#### 可能性加权(Likelihood weighting)
##### 算法
可能性加权(Likelihood weighting)只产生和证据$\mathbf{e}$一致的事件，因此避免了拒绝采样的低效。它是统计学中重要性采样的一个例子，专门为贝叶斯推理设计的。
如Figure 14.15所示，加权似然固定证据变量$\mathbf{E}$的值，只对非证据变量进行采样，这就保证了每一个事件都是和证据一致的。但是，不是所有的事件权重都是一样的。给定每一个证据变量的父节点，它的可能性(likelihood)是证据变量的条件概率的乘积，每一个事件都根据证据的可能性进行加权。

##### 示例
对于Figure 14.12(a)中的例子，计算后验概率$P(Rain|Cloudy=true,WetGrass=true)$，采样顺序是Cloudy,Sprinkler,Rain,WetGrass。过程如下，首先，权重$w$设为$1$，一个事件生成过程如下：
1. Cloudy是一个证据变量，它的值是true,因此，令：
$$w\leftarrow w\times P(cloudy=true) = 0.5.$$
2. Sprinkler是隐变量，所以从$P(Sprinkler|Cloudy=true)=<0.1,0.9>$中采样，假设采样结果是false； 
3. Rain是隐变量，从$P(Rain|Cloudy=true)=<0.8,0.2>$中采样，假设采样结果是true； 
4. WetGrass是证据变量，值是true,令：
$$w\leftarrow w\times P(WetGrass=true|Sprinkler=false,Rain=true) = 0.45.$$ 

所以WEIGHTED-SAMPLE算法生成事件[true,false,true,true]，相应的权重是$0.45$。
##### 原理
用$S\_{WS}$表示WEIGHTED-SAMPLE算法中事件的采样概率，证据变量$\mathbf{E}$的取值$\mathbf{e}$是固定的，用$\mathbf{Z}$表示非证据变量，包括隐变量$\mathbf{Y}$和查询变量$\mathbf{X}$。给定变量$\mathbf{Z}$的父节点，算法对变量$\mathbf{Z}$进行采样：
$$S\_{WS}(\mathbf{z},\mathbf{e}) = \prod\_{i=1}^l P(z\_i|parents(Z\_i)).$$
其中$Parents(Z\_i)$可能同时包含证据变量和非证据变量。
和先验分布$P(\mathbf{z})$不同的是，每一个变量$Z\_i$的取值会受到$Z\_i$的祖先(ancestor)变量的影响。比如，对Sprinkler进行采样的时候，算法会受到它的父节点中的证据变量Cloudy=true的影响，而先验分布不会。另一方面，$S\_{WS}$比后验分布$P(\mathbf{z}|\mathbf{e})$受证据的影响更小，因为对$Z\_i$的采样忽略了$Z\_i$的非祖先(non-ancestor)变量中的证据。比如，对Sprinkler和Rain进行采样的时候，算法忽略了子节点中的证据变量WetGrass=true，事实上这个证据已经排除了(rule out)Sprinkler=false和Rain=false的情况，但是WEIGHTED-SAMPLE还会产生很多这样的样本事件。
理想情况下，我们想要一个采样分布和真实的后验概率$P(\mathbf{z}|\mathbf{e})$相等，不幸的是不存在这样的多项式时间的算法。如果有这样的算法的话，我们可以用多项式数量的样本以任意精度逼近想要求的概率值。
可能性权重$w$弥补了实际的分布和我们想要的分布之间的差距。一个由$\mathbf{z}$和$\mathbf{e}$组成的样本$\mathbf{x}$的权重是给定了父节点的证据变量的可能性乘积：
$$w(\mathbf{z},\mathbf{e}) = \prod\_{i=1}^m P(e\_i|parents(E\_i)).$$
将上面的两个式子乘起来，可以得到一个样本的加权概率(weighted probability)是：
$$S\_{WS}(\mathbf{z},\mathbf{e})w(\mathbf{z},\mathbf{e}) = \prod\_{i=1}^l P(z\_i|parents(Z\_i))\prod\_{i=1}^m P(e\_i|parents(E\_i)) = P(\mathbf{z},\mathbf{e}).$$
可能性加权估计是一致估计。对于任意的$x$，估计的后验概率按下式计算：
\begin{align\*}
\hat{P}(x|\mathbf{e}) &= \alpha \sum\_{\mathbf{y}} N\_{WS}(x,\mathbf{y},\mathbf{e})w(x,\mathbf{y},\mathbf{e})\\\\
&\approx \alpha'\sum\_{\mathbf{y}}S\_{WS}(x,\mathbf{y},\mathbf{e})w(x,\mathbf{y},\mathbf{e})\\\\
&=\alpha'\sum\_{\mathbf{y}}P(x,\mathbf{y},\mathbf{e})\\\\
&=\alpha'\sum\_{\mathbf{y}}P(x,\mathbf{y},\mathbf{e})\\\\
&=P(x|\mathbf{e})
\end{align\*}
算法中真实实现的是第一行，即统计出用WEIGHTED-SAMPLE产生的样本$(x,\mathbf{y},\mathbf{e})$数量$N\_{WS}$，以及对应的权重$w(x,\mathbf{y},\mathbf{e})$，后面的都是理论推导，当$N$取极限的时候$lim\_{N\rightarrow \infty}\frac{N\_{WS}(x\_1,\cdots,x\_n)}{N} = S\_{WS}(x\_1,\cdots,x\_n)$，后面的都是为了证明算法是一致估计。

##### 不足
可能性加权算法使用了所有生成的样本，它比拒绝假设算法更高效。然而，随着证据变量的增加，算法性能会退化(degradation)，这是因为很多样本的权重都会很小，因此加权估计可能会受一小部分权重很大的样本的影响(dominated)。如果证据变量在非证据变量的后边，这个问题会加剧，因为它们的父节点或者祖先节点没有证据变量来指导样本的生成。这就意味着生成的样本和证据变量支撑的真实情况可能差距很大(bear little resemblance)。

### 马尔科夫链仿真推理(Inference by Markov chain simulation)
[马尔科夫链蒙特卡洛(Markov chain Monte Carlo,MCMC)](https://mxxhcm.github.io/2019/08/01/Monte-Carlo-Markov-Chain/)算法和拒绝采样以及可能性加权很不一样。那两个方法每次都从头开始生成样本，而MCMC算法在之前的样本上做一些随机的变化。可以将MCMC算法看成指定了每一个变量值的特殊当前状态(current state)，通过对当前状态(current state)做任意的改变生成下一个状态(next state)。这一节要介绍的一种MCMC算法是吉布森采样(Gibbs sampling)。

#### 贝叶斯网络中的吉布森采样(Gibbs sampling in Bayesian networks)
##### 算法
贝叶斯网络中的吉布森采样从任意一个状态开始，其中证据变量的取值固定为观测值，通过随机选取非证据变量$X\_i$的值生成下一个状态。变量$X\_i$的采样取决于变量$X\_i$的马尔科夫毯的当前值。算法在状态空间（所有非证据变量的全部可能取值空间）中随机采样，每次采样都保持证据变量不变，一次改变一个非证据变量的值。完整的算法如Figure 14.16所示。
![figure 14.16]()

##### 示例
Figure 14.12(a)中的查询(query)$P(Rain|Sprinkler=true, WetGrass=true)$，证据变量Spinkler和WetGrass取它们的观测值不变，非证据变量Cloudy和Rain随机初始化，假设取的是true和false。那么初始状态就是[true,true,false,true]，接下来对非证据变量进行重复的随机采样。
比如第一次对Cloudy采样（也可以对Rain采样），给定它的马尔科夫毯变量，然后从$P(Cloudy|Sprinkler=true,Rain=false)$中进行采样，假设采样结果是false，新的状态就是[false,true,false,true]。接下来随机可以对Rain采样（也可以对Cloudy采样），给定Rain的马尔科夫毯变量的取值，从$P(Rain|Cloudy=false,Sprinkler=true,WetGrass=true)$中进行采样，假设采样值是true,那么新的状态是[true,true,false,false]。接下来可以一直进行采样。。最终利用生成的样本计算出相应的概率。

#### 为什么吉布森采样有用(Why Gibbs sampling works)
接下来给出为什么吉布森采样计算后验概率是一致估计。基本的解释是直截了当的：采样过程建立了一个动态平衡，每个状态花费的时间长期来说和它的后验概率是成比例的。
具体的，不想看了。。。就随缘吧

## 参考文献
1.《Aritifical Intergence》