---
title: python ptan
date: 2019-10-12 20:36:40
tags:
 - python
 - pytorch
categories: python
---

## PyTorch Agent Net library
### 简介
Ptan是一个简化RL的库，它主要目标是实现两个问题的平衡：
1. 导入库函数，只需要一行命令，就像OpenAI的baselines一样
2. 从头开始实现

我们既不想一行命令直接调包，也不想从头开始实现一切。

### 模块
- Agent：
- ActionSelector
- ExperienceSource
- ExperienceSourceBuffer
- others

## Action Selector
将network的输出转换成具体的action。常用的有
- Argmax：用于使用Q网络的方法，生成离散action。
- Policy-based：网络输出logits或者normalizaed distribution，从这个distribution中采样。

Action Selector被Agent使用，常用的有：
- ArgmaxActionSelector
- EpsilonGreedyActionSector
- ProbabilityActionSelector

### 基类
```
class ActionSelector:
    def __call__(self, scores)
```
### 子类
#### ArgmaxActionSelector
```
class ArgmaxActionSelector(ActionSelector):
    def __call__(self, scores)
```

#### EpsilonGreedyActionSector
``` 
class EpsilonGreedyActionSector(ActionSelector):
    def __init__(self, epsilon=0.05, selector=None)
    def __call(self, scores)
```

#### ProbabilityActionSelector
```
class ProbabilityActionSelector(ActionSelector):
    def __call__(self, probs)
```

#### 对比三个ActionSelector
两个GreedySelector：Argmax和EpsilonGreedy，输入都需要是q值，输出是action。
而Probability需要的输入是概率，输出是动作。

### 其他
EpsilonTraker，用来记录epsilon的变化。

## Agent
将observation转换为actions，常见的三种方法如下：
- Q-function：预测当前observation下所有可能采取的action的$Q$值，选择$\arg \max Q(s)$作为action。
- Policy-based：预测$\pi(s)$的概率分布，从分布中采样。
- Continuous Contrl：预测连续控制参数$\mu(s)$，直接输出action。

### 基类
``` 
class BaseAgent:
    def __initial_state(self)
    def __call__(self, states, agent_states)
        """
        :param states: env states list 
        :param agent_states: agent state list
        :return: actions tuple, agent_states
        """
```

### 子类
#### DQNAgent
```
class DQNAgent(BaseAgent):
    def __init__(self, dqn_model, action_selector, device="cpu", preprocessor=default_states_preprocessor)
    def __call__(self, states, agent_states=None)
```
#### PolicyAgent
输入的model产生离散动作的policy distribution，Policy distribution可以是logtis或者normalized distribution。
PolicyAgent调用probability对这个distribution进行采样 。PolicyAgent其实就是将model和action selector组装在了一起。
```
class PolicyAgent(BaseAgent):
    def __init__(self, model, action_selector=actions.ProbabilityActionSelector(), device="cpu", apply_softmax=False, preprocessor=default_states_preprocessor)
    @torch.no_grad()
    def __call__(self, states, agent_states=None)
```

#### ActorCriticAgent
```
class ActorCriticAgent
    def __init__(self, model, action_selector=actions.ProbabilityActionSelector(), device="cpu", apply_softmax=False, preprocessor=default_states_preprocessor)
    def __call__(self, states, agent_states=None)

```

### 其他
#### default_states_preprocessor
```
def default_states_preprocessor(states):
    """
    Convert list of states into the form suitable for model. By default we assume Variable
    :param states: list of numpy arrays with states
    :return: Variable
    """
    if len(states) == 1:
        np_states = np.expand_dims(states[0], 0)
    else:
        np_states = np.array([np.array(s, copy=False) for s in states], copy=False)
    return torch.tensor(np_states)
```

#### TargetNet
``` 
class TargetNet
    def __init__(self, model)
    def sync(self)
    def alpha_sync(self, alpha)
```

## Experience Source
Agent不断的和env进行交互产生一系列的trajectories，Experience可以将这些交互存储起来，重复利用。Experience的主要作用有：
1. 支持batch，利用GPU的并行计算提高训练效率
2. 可以对transitions或者trajectory进行预处理。比如n-step DQN。
3. ???

常见的ExperienceSource有：
- ExperienceSource
- ExperienceSourceFirstLast
- ExperienceSourceRollouts

### 基类
#### ExperienceSource
``` 
class ExperienceSource 
    """
    简单的n-step source for single or multiple envs
    每一个experience都有n个Experience entries的list
    """
    #
    def __init__(self, env, agent, steps_count=2, stepd_delta=1, vectorized=False)
    """
    """
    #
    def __iter__(self):
    """
    重写for循环的iter方法
    """
    #
    def pop_total_rewards(self)
    #
    def pop_rewards_steps(self)
```

#### ExperienceReplayBuffer
```
class ExperienceReplayBuffer
```

#### BatchPreprocessor
```
class BatchPreprocessor
```

### 子类
#### ExperienceSourceFirstLast
``` 
class ExperienceSourceFirstLast(ExperienceSource):
    #
    def __init__(self, env, agent, gamma, steps_count=1, steps_delta=1, vectorized=False)
    #
    def __iter(self)
```

#### PrioritizedReplayBuffer
```
class PrioritizedReplayBuffer(ExperienceReplayBuffer)
    # 1.
    def __init__(self, experience_source, buffer_size, alpha)
    # 2.
    def _add(self, *args, **kwargs)
    # 3.
    def _sample_proprotional(self, batch_size)
    # 4.
    def sample(self, batch_size, beta)
    # 5.
    def update_priorities(self, idxes, priorities)
```

#### QLearningPreprocessor
```
class QLearningPreprocessor(BatchPreprocessor)
```

### 其他
#### ExperienceSourceRollouts
``` 
class ExperienceSourceRollouts:
    #
    def __init__(self, env, agent, gamma, setps_count=5)
    # 
    def __iter__(self)
    #
    def pop_total_rewards(self)
    # 
    def pop_rewards_steps(self)
```

#### ExperienceSourceBuffer
```
class ExperienceSourceBuffer
```

#### ExperienceReplayNaive
```
class ExperienceReplayNaive
```

## 参考文献
1.https://github.com/Shmuma/ptan/blob/master/docs/intro.ipynb
