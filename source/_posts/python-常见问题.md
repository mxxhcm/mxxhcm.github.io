---
title: python 常见问题（不定期更新）
date: 2019-03-13 10:40:03
tags:
 - python
 - gym
 - 常见问题
categories: python 
---

## 问题1-'dict_values' object does not support indexing'
参考文献[1,2,3]
### 报错
``` txt
'dict_values' object does not support indexing'
```

### 原因
The objects returned by dict.keys(), dict.values() and dict.items() are view objects. They provide a dynamic view on the dictionary’s entries, which means that when the dictionary changes, the view reflects these changes.
python3 中调用字典对象的一些函数，返回值是view objects。如果要转换为list的话，需要使用list()强制转换。
而python2的返回值直接就是list。

### 代码示例
```python
m_dict = {'a': 10, 'b': 20}
values = m_dict.values()
print(type(values))
print(values)
print("\n")

items = m_dict.items()
print(type(items))
print(items)
print("\n")

keys = m_dict.keys()
print(type(keys))
print(keys)
print("\n")
```
如果使用python3执行以上代码，输出结果如下所示：
> class 'dict_values'
dict_values([10, 20])
class 'dict_items'
dict_items([('a', 10), ('b', 20)])
class 'dict_keys'
dict_keys(['a', 'b'])

如果使用python2执行以上代码，输出结果如下所示：
> type 'list'
[10, 20]
type 'list'
[('a', 10), ('b', 20)]
type 'list'
['a', 'b']

## 问题2-'TimeLimit' object has no attribute 'ale'
参考文献[4,5,6]
### 问题描述
运行github clone 下来的[DQN-tensorflow](https://github.com/devsisters/DQN-tensorflow)，报错:
> AttributeError: 'TimeLimit' object has no attribute 'ale'.

### 原因
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

### ale属性是什么
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

## 问题3-cannot import name \*\*\*
参考文献[7]
### 报错
``` txt
cannot import name tqdm
```

### 问题原因
谷歌了半天，没有发现原因，然后百度了一下，发现了原因，看来还是自己太菜了。。
因为自己起的文件名就叫tqdm，然后就和库中的tqdm冲突了，这也太蠢了吧。。。

## 问题4-linux下python执行shell脚本输出重定向
[详细介绍](https://mxxhcm.github.io/2019/06/03/linux-python调用shell脚本并将输出重定向到文件/)
 
## 参考文献
1.https://www.cnblogs.com/timxgb/p/8905290.html
2.https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects
3.https://stackoverflow.com/questions/43663206/typeerror-unsupported-operand-types-for-dict-values-and-int
4.https://github.com/devsisters/DQN-tensorflow/issues/29
5.https://gym.openai.com/docs
6.https://github.com/openai/baselines/issues/42
7.https://blog.csdn.net/m0_37561765/article/details/78714603
