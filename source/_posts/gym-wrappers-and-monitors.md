---
title: gym wrappers and monitors
date: 2019-10-09 15:24:33
tags:
 - gym
 - python
 - 强化学习
categories: 强化学习
---

这一篇文章主要介绍wrappers和monitors。

## 什么是Wrappers
对已有的environments的功能进行扩展。比如保留过去的N个observation或者对输入进行cropped。Gym提供了这样的借口，叫做Wrapper class。
Wrapper继承自Env class，构造函数只接收一个参数，要被"wrapped"的Env实例。为了添加新的逻辑，需要重写step()或者reset()方法，但是需要调用父类的original方法。为了更细粒度的满足要求，gym还提供了三个Wrapper的subclass，ObservationWrapper，RewardWrapper和ActionWrapper分别只对observation，reward和action进行重定义。他们之间的关系如下所示：
![gym_wrapper](gym_wrapper.png)
Env是一个abstract class，具体的environments如Breakout继承了Env class，实现了step()，reset()等abstract function。Wrapper继承了env class，对step(), reset()等方法进行了重载。ActionWrapper对Wrapper进行了重载，对step和reset进行了重载。
Env 
- abstract step(self, action)
- abstract reset(self)

Breakout(Env)
- overwrite step(self, action)
- overwrite reset(self)

Wrapper(Env)
- \_\_init\_\_(self, env): self.env = env # instance of gym environment
- overwrite step(self, action): self.env.step(action) # 调用的是传入参数env的step函数
- overwrite reset(self) # 调用的是传入参数env的reset函数

ActionWrapper(Wrapper)
- overwrite step(self, action): self.env.step(action) # 调用的是self.env的step函数
- overwrite reset(self) # 调用的是self.env的step函数
- abstract action(self, action)
- abstract reverse_action(self, action)

MyownActionWrapper(ActionWrapper)
- overwrite action(self, action)

## Wrapper示例
``` python
import gym
import random
import time


class RandomActionWrapper(gym.ActionWrapper):
    def __init__(self, environment, epsilon=-1.1):
        super(RandomActionWrapper, self).__init__(environment)
        self.epsilon = epsilon

    def action(self, action):
        if random.random() < self.epsilon:
            print("Random")
            return self.env.action_space.sample()
            # return self.environment.action_space.sample()
        else:
            # print("Not random")
            pass
        return action

    def reverse_action(self, action):
        pass


if __name__ == "__main__":
    environment = gym.make("CartPole-v0")
    env = RandomActionWrapper(environment)
    
    obs = env.reset()
    episode = 0
    total_steps = 0
    total_reward = 0
    while True:
        action = env.action_space.sample()
        obs, reward, done, info = env.step(action)
        print(reward)
        total_reward += reward
        total_steps += 1
        env.render()
        time.sleep(0.1)
        if done:
            print("Episode %d done in %d steps, total reward %.1f" %(episode, total_steps, total_reward))
            time.sleep(0.1)
            env.reset()
            episode += 1
            total_reward = 0

```

## 什么是Monitors
使用gym.wrapper.Monior记录当前agent的执行动作。它的使用方法很简单，如下所示：
``` python
env = gym.make("CartPole-v0")
env = gym.wrappers.Monitor(env, "path")
```
只需要在env的外面套上一个Monitor即可，它会自动记录玩游戏的过程。

## Monitors示例
``` python
import gym
import time

if __name__ == "__main__":
    env = gym.make("CartPole-v0")
    env = gym.wrappers.Monitor(env, "recording")

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
            if episode > 5:
                break
            episode += 1
            total_reward = 0

```

## 参考文献
1.https://hub.packtpub.com/openai-gym-environments-wrappers-and-monitors-tutorial/
2.https://www.packtpub.com/big-data-and-business-intelligence/deep-reinforcement-learning-hands
