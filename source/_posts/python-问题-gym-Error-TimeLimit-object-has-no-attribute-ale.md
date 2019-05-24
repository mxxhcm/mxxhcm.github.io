---
title: gym Error! TimeLimit' object has no attribute 'ale'
date: 2019-03-27 11:00:16
tags:
 - python
 - gym
 - tensorflow
 - 常见问题
 - 机器学习
categories: python
---

## 问题描述
运行github clone 下来的[DQN-tensorflow](https://github.com/devsisters/DQN-tensorflow)，报错:
> AttributeError: 'TimeLimit' object has no attribute 'ale'.
是因为gym版本原因，在gym 0.7版本中，可以使用env.ale.lives()访问ale属性，但是0.8版本以及以上，就没有了该属性，可以在系列函数中添加如下修改：
``` python
def __init__(self, config):
    self.step_info = None

def _step(self, action):
    self._screen, self.reward, self.terminal, self.step_info = self.env.step(action)

def lives(self):
    if self.step_info is None:
        return 0
    else:
        return self.step_info['ale.lives']
```

## ale属性是什么
我看官方文档也没有看清楚，但是我觉得就是生命值是否没有了
> info (dict): diagnostic information useful for debugging. It can sometimes be useful for learning (for example, it might contain the raw probabilities behind the environment’s last state change). However, official evaluations of your agent are not allowed to use this for learning.

``` python
import gym
env = gym.make('CartPole-v0')
for i_episode in range(20):
    observation = env.reset()
    for t in range(100):
        env.render()
        print(observation)
        action = env.action_space.sample()
        observation, reward, done, info = env.step(action)
        if done:
            print("Episode finished after {} timesteps".format(t+1))
            break
env.close()
```


## 参考文献
1.https://github.com/devsisters/DQN-tensorflow/issues/29
2.https://gym.openai.com/docs
3.https://github.com/openai/baselines/issues/42
