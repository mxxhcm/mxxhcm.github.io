---
title: gym介绍
date: 2019-04-12 16:54:44
tags:
 - gym
 - 强化学习
categories: gym
mathjax: true
---

## 简介
强化学习中最主要的两类对象是“智能体”和“environment”，以及和这两类对象相关的一些概念：“reward”、“return”、“state”、“action”、“value”、“policy”、“predict”、“control”等，他们之间有着以下的关系：
1. environment会对智能体采取的action做出回应。当智能体执行一个行为时，它需要根据environment本身的动力学来更新environment，也包括更新智能体状态，同时给以智能体一个反馈信息：即时奖励(immediate reward)。
2. 对于智能体来说，它并不知道整个environment的所有信息，只能通过观测(observation)来获得所需要的信息，它能观测到的信息取决于问题的设置；同样因为智能体需要通过action与environment进行交互，智能体能采取哪些action，也要由智能体和environment协商好。因此environment要确定智能体的观测空间和action空间。
3. 智能体还需要有一个决策功能，该功能根据当前observation来判断下一时刻该采取什么action，也就是决策过程。
4. 智能体能执行一个确定的action。（这个刚开始还没想明白，智能体执行什么action干嘛，一般我们写代码不都是env.step(action)，后来才想到是action本身就是智能体自己执行的，只不过代码是这么写，因为environment需要根据这个action，去更新智能体的状态以及environment的状态。）
5. 智能体应该能从与environment的交互中学到知识，进而在与environment交互时尽可能多的获取reward，最终达到最大化累积奖励(accumate reward)的目的。
6. environment应该给智能体设置一个（些）终止条件，即当智能体处在这个状态或这些状态之一时，交互结束，即产生一个完整的Episode。随后重新开始一个Episode或者退出交互。

## 自己实现一个environment
如果用代码表示上述关系，可以定义为如下式子：
``` python
class Environment(object):
  self.aget_state #
  self.states # 所有可能的状态集合
  self.observation_space # 智能体的observation space
  self.action_space # 智能体体的action space

  # 给出智能体的immediate reward
  def reward(self):
    pass

  # 根据智能体的动作，更新环境
  def step(self, action):
    pass

  # 当前回合是否结束
  def is_episode_end():
    pass

  # 生成智能体的obs
  def obs_for_agent():
    pass

class Agent(object):
   self.env = env # 智能体依附于某一个环境
   self.obs # 智能体的obs
   self.reward  # 智能体获得的immediate reward

   # 根据当前的obs生成action
   def policy(self, obs):
      self.action

   # 智能体观测到obs和reward
   def observe(self):
     self.obs = 
     self.reward = 

```

## Action space
Agent执行的actions可以是discrete，也可以是continuous，或者是discrete和continuous相结合的。Discrete actions是agent能够做的一系列操作，比如在地图中的上下左右操作，每一个action都是互斥的，他们不能同时发生。
一个continous action有一个值，比如说，可以是方向盘的一个具体角度，从-720到720；也可以是油门上施加的力的程度，从0到1。

## Observation space
Observations是environment在每一个timestep返回的信息。它可以是几个数字，也可以是从cameras获得的多通道rgb图像。和action space一样，observation space可以是discrete，也可以是continuous。比如像灯泡的状态，有亮和不亮。

## gym
gym库在设计environment和智能体的交互时基本上也是按照这几条关系来实现自己的规范和接口的。gym库的核心在文件core.py里，这里定义了两个最基本的类Env和Space。
Env类是所有environment类的基类，Space类是所有space类的基类，action space和observation都是基于Space类实现的。

## Spaces
Space是一个抽象类，其中包含以下函数，以下几个全是abstract函数，需要在子类中实现
- __init__(self, shape=None, dtype=None) 函数初始化shape和dtype以及初始化numpy随机数RandomState()对象。
- sample(self) 函数进行采样，实际上是调用了numpy的随机函数。
- contains(self, x) 函数判断某个对象x是否是这个space中的一个member。
- seed(self, seed) 设置numpy随机数种子，这里使用的是RandomState对象，生成随机数，种子一定的情况下，采样的过程是一定的。
- to_jsonable(self, sample_n)
- from_jsonable(self, sample_n)

从Space基类派生出几个常用的Space子类，其中最主要的是Discrete类和Box类，其余的还有MultiBinary类，MultiDiscrete类，Tuple类等，每个子类重新实现了__repr__和__eq__以及几乎所有Space类中的函数。可以看以下类图：
![class_diagram](class_diagram.png)
最常见的Discrete和Box类，Discrete对应于一维离散空间，Box对应于多维连续空间。它们既可以应用在action space中，也可以用在state space，可以根据具体场景选择。

### Discrete
#### 说明
Discrete声明的时候需要给定一个整数，然后整个类的取值在${0, 1, \cdots, n-1}$之间。然后使用sample()函数采样，实际调用的是numpy的randint()进行采样，得到一个整数值。
#### 示例
``` python
from gym import spaces

# 1.Discrete
# 取值是{0, 1, ..., n - 1}
print("==================")
dis = spaces.Discrete(8)
print(dis.shape)
print(dis.n)
print(dis)
dis.seed(4)
for _ in range(5):
    print(dis.sample())
```
输出结果是：
> ==================
() # shape是None
8  # n为8
Discrete(8) # repr()函数的值
2
6
7
5
1

### Box
#### 说明
而Box类应用于连续空间，有两种初始化方式，一种是给出最小值，最大值和shape，另一种是直接给出最小值矩阵和最大值矩阵。然后使用sample()函数采样，实际上调用的是numpy的uniform()函数。

#### 代码示例
``` python
from gym import spaces
# 2.Box
#
print("==================")
# def __init__(self, low=None, high=None, shape=None, dtype=None):
"""
Two kinds of valid input:
    Box(low=-1.0, high=1.0, shape=(3,4)) # low and high are scalars, and shape is provided
    Box(low=np.array([-1.0,-2.0]), high=np.array([2.0,4.0])) # low and high are arrays of the same shape
"""
box = spaces.Box(low=3.0, high=4, shape=(2,2))
print(box) 
box.seed(4)
for _ in range(2):
    print(box.sample())

```
输出结果是：
> ==================
Box(2, 2) # repr()函数的值
[[3.9670298 3.5472322]
 [3.9726844 3.714816 ]]
[[3.6977289 3.2160895]
 [3.9762745 3.0062304]]

### Tuple
#### 说明
当某些场景的state既有discrete也有continuous时，这就可以使用tuple了。举个例子来说，开车时，有三个连续控制组件：方向盘，刹车，油门等，它们可以用一个Box表示；此外还有一些离散的控制组件如转向灯（关闭，左转，右转等），喇叭（开，关）等。这就可以用tuple表示：
``` python
Tuple(spaces=(Box(low=-1.0, high=1.0, shape=(3,)), Discrete(n=3), Discrete(n=2)))
```

### 示例
这里给出一个应用场景，例如要描述一个$4\times 4$的网格世界，它一共有16个状态，每一个状态只需要用一个数字来描述即可，这样可以用Discrete(16)对象来表示这个问题的state space。
对于经典的小车爬山的问题，小车的state是用两个变量来描述，一个是小车对应目标旗杆的水平距离，另一个是小车的速度，因此environment要描述小车的state需要2个连续的变量。由于小车的state对智能体是完全可见的，因此小车的state space即是小车的observation space，此时不能用Discrete来表示，要用Box类，Box空间定义了多维空间，每一个维度用一个最小值和最大值来约束。同时小车作为智能体可以执行的action有3个：左侧加速、不加速、右侧加速。因此action space可以用Discrete来描述。最终，该environment类的观测空间和行为空间描述如下：
``` python
class Env(obejct):
  self.min_position = -1.2
  self.max_position = 0.6
  self.max_speed = 0.07
  self.goal_position = 0.5 
  self.low = np.array([self.min_position, -self.max_speed])
  self.high = np.array([self.max_position, self.max_speed])
  self.action_space = spaces.Discrete(3)  # action space,是离散的
  self.observation_space = spaces.Box(self.low, self.high) # 状态空间是连续的
```

## Env
### 组成
OpenAI官方在gym.core.Env类中给出了如下的说明
The main OpenAI Gym class. It encapsulates an environment with arbitrary behind-the-scenes dynamics. An environment can be partially or fully observed.

#### 常用属性（三个）
- action_space: 环境中允许的actions的介绍
- observation_space: 指定了环境给出的observation
- reward_range: A tuple corresponding to the min and max possible rewards 

Note: a default reward range set to [-inf,+inf] already exists. Set it if you want a narrower range.

#### 常用方法（五个）
- step
- reset
- render
- close
- seed

The methods are accessed publicly as "step", "reset", etc.. The non-underscored versions are wrapper methods to which we may add functionality over time.

智能体主要通过环境的几个方法进行交互，用户如果要编写自己的环境的话，需要实现seed, reset, step, close, render等函数。

### step函数执行一个时间步的更新。       
#### 说明
Accepts an action and returns a tuple (observation, reward, done, info).
输入参数：
  action (object):智能体执行的动作
返回值：
- observation (object): 环境的observation，一个numpy数组，
- reward (float) : 采取输入的action之后环境给出的reward
- done (boolean): 当前episode是否结束，一个bool变量
- extra_info (dict): 调试信息，一般情况下会忽略，是一个dict，可能是当前agent还有多少条命。

每一次调用step()，都会执行以下操作：
1. 告诉env在接下来的一个timestep中采取什么action
2. 获得一个新的observation
3. 获得一个新的reward
4. 获得当前episode是否结束
5. 获得其他额外信息。

### reset函数重置
不接收输入参数，重置环境并返回初始的observation。
Returns: observation (object): 将环境重置为初始状态，返回环境的初始observation

### reder函数绘制
Renders the environment.

### close函数回收garbge
在使用完之后调用close函数清理内存

### seed函数设置环境的随机数种子
使用seed函数设置随机数种子，使得结果可以复现。

## 创建一个environment
在使用Env类的时候，一种是使用gym中自带的已经注册了的类，另一种是使用自己编写的类。
### gym中自带的envs
gym中有很多很多个自带的environments。拿gym 0.9.3来说，总共有777个环境（包含同一种env的不同变种），116个unique env，他们可以分为以下几类：
- 经典的控制问题：这类问题很简单，但是可以用来检查模型的实现。
- Atari 2600, 63 unique游戏
- Board games
- Box2D
- MuJoCo
- Parameter tunning
- Toy text
- PyGame
- Doom

具体的可以查看[https://gym.openai.com/envs/#classic_control](https://gym.openai.com/envs/#classic_control)。

#### 代码实例
第一种的话，使用如下语句注册：
``` python
import gym
env = gym.make("CartPole-v0")

print(type(env.action_space))
# Discrete
print(env.action_space.n)
# 2, action的取值就是0和1，0向左推，1向右推
print(env.action_space.shape)
# ()
print(type(env.observation_space))
# Box
print(env.observation_space.shape)
# (4,)

s_0 = env.reset()
print(s_0)
s_1, r_1, done_1, info_1 = env.step(0)
print(s_1)
```

#### RandomAgent示例
``` python
import gym
import time

if __name__ == "__main__":
    env = gym.make("CartPole-v0")
    total_reward = 0.0
    total_steps = 0
    obs = env.reset()
    episode = 1
    while True:
        action = env.action_space.sample()
        obs, reward, done, _ = env.step(action)
        total_reward += reward
        total_steps += 1
        env.render()
        if done:
            print("Episode %d done in %d steps, total reward %.2f" %(episode, total_steps, total_reward))
            time.sleep(1)
            env.reset()
            episode += 1
            total_reward = 0

```


### 自己声明envs
另一种自己编写的环境类是和普通的python 类对象声明一样。

## Some issues
1.>gym.error.DeprecatedEnv: Env PongDeterministic-v4 not found (valid versions include ['PongDeterministic-v3', 'PongDeterministic-v0'])
gym版本太老了，升级一下就行[2]。这个是gym$0.7.0$遇到的问题。
2.>UserWarning: WARN: <class 'envs.AtariRescale42x42'> doesn't implement 'observation' method. Maybe it implements deprecated '\_observation' method.
这个是gym版本太新了，apis进行了重命名。这个是gym$0.12.0$遇到的问题。
上面两个问题都是在测试github上的一个![A3C](https://github.com/ikostrikov/pytorch-a3c/)代码遇到的。最后装了$0.9$版本的gym就没有警告了。（测试了一下，装$0.10$版本的也不行）

## 参考文献
1.https://github.com/openai/gym
2.https://github.com/ikostrikov/pytorch-a3c/issues/36
3.https://github.com/openai/roboschool/issues/169
4.https://www.packtpub.com/big-data-and-business-intelligence/deep-reinforcement-learning-hands