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
### 简介
将network的输出转换成具体的action。常用的有
- Argmax：用于使用Q网络的方法，生成离散action。
- Policy-based：网络输出logits或者normalizaed distribution，从这个distribution中采样。

Action Selector被Agent使用，常用的有：
- ArgmaxActionSelector
- EpsilonGreedyActionSector
- ProbabilityActionSelector

### 基类
#### ActionSelector
``` python
class ActionSelector:
    def __call__(self, scores)
```
### 子类
#### ArgmaxActionSelector
``` python
class ArgmaxActionSelector(ActionSelector):
    def __call__(self, scores)
```

#### EpsilonGreedyActionSector
``` python 
class EpsilonGreedyActionSector(ActionSelector):
    def __init__(self, epsilon=0.05, selector=None)
    def __call(self, scores)
```

#### ProbabilityActionSelector
``` python
class ProbabilityActionSelector(ActionSelector):
    def __call__(self, probs)
```

#### 对比三个ActionSelector
两个GreedySelector：Argmax和EpsilonGreedy，输入都需要是q值，输出是action。
而Probability需要的输入是概率，输出是动作。

### 其他
#### EpsilonTraker
用来记录epsilon的变化。

## Agent
将observation转换为actions，常见的三种方法如下：
- Q-function：预测当前observation下所有可能采取的action的$Q$值，选择$\arg \max Q(s)$作为action。
- Policy-based：预测$\pi(s)$的概率分布，从分布中采样。
- Continuous Contrl：预测连续控制参数$\mu(s)$，直接输出action。

### 基类
#### BaseAgent
``` python 
class BaseAgent:
    # 1.
    def __initial_state(self)
    # 2.
    def __call__(self, states, agent_states)
        """
        :param states: env states list 
        :param agent_states: agent state list
        :return: actions tuple, agent_states
        """
```

### 子类
#### DQNAgent
``` python
class DQNAgent(BaseAgent):
    # 1.
    def __init__(self, dqn_model, action_selector, device="cpu", preprocessor=default_states_preprocessor)
    # 2
    def __call__(self, states, agent_states=None)
```
#### PolicyAgent
输入的model产生离散动作的policy distribution，Policy distribution可以是logtis或者normalized distribution。
PolicyAgent调用probability action selector对这个distribution进行采样 。PolicyAgent其实就是将model和action selector组装在了一起。
``` python
class PolicyAgent(BaseAgent):
    # 1.
    def __init__(self, model, action_selector=actions.ProbabilityActionSelector(), device="cpu", apply_softmax=False, preprocessor=default_states_preprocessor)
    # 2.
    @torch.no_grad()
    def __call__(self, states, agent_states=None)
```

#### ActorCriticAgent
``` python
class ActorCriticAgent
    # 1.
    def __init__(self, model, action_selector=actions.ProbabilityActionSelector(), device="cpu", apply_softmax=False, preprocessor=default_states_preprocessor)
    # 2.
    def __call__(self, states, agent_states=None)

```

### 其他
#### default_states_preprocessor
``` python
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
``` python
class TargetNet
    # 1.
    def __init__(self, model)
    # 2.
    def sync(self)
    # 3.
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
- ExperienceReplayBuffer: ：DQN中几乎不会使用刚刚获得的experience samples，因为他们是高度相关的，让训练很不稳定。Buffer用来存放experience pieces，从buffer中采样进行训练，因为buffer容量有限，老样本会被从replay buffer中删掉
- PrioReplayBufferNaive: Complexity of sampling is O(n)
- PrioritizedReplayBuffer: O(log(n)) sampling complexity.

### 基类
#### ExperienceSource
```  python
class ExperienceSource 
    """
    简单的n-step source for single or multiple envs
    每一个experience都有n个Experience entries的list
    """
    # 1.
    def __init__(self, env, agent, steps_count=2, steps_delta=1, vectorized=False)
        """
        env: 环境或者list of环境
        agent: 将observation 转换为actions
        steps_count: 每一个experience chain的计数
        steps_delta: experience items之间相隔多少个steps
        vectorized: bool,OpenAI vectorized
        """
    # 2.
    def __iter__(self):
    """
    重写for循环的iter方法
    """
    # 3.返回rewards，然后重置
    def pop_total_rewards(self)
    # 4.返回rewards和steps，然后重置
    def pop_rewards_steps(self)
```

#### ExperienceReplayBuffer
``` python
class ExperienceReplayBuffer
    #
    def __init__(self, experience_source, buffer_size)

    #
    def __len__(self)

    #
    def __iter__(self)

    # 从experience中随机采样一个batch_size大小的样本
    def sample(self, batch_size)

    # 添加一个sample，类内函数
    def _add(self, sample)

    # 从experience_source中获得samples_numbers个样本，将其添加到buffer
    def populate(self, samples_numbers)
```

#### BatchPreprocessor
``` python
class BatchPreprocessor
```

### 子类
#### ExperienceSourceFirstLast
``` python
# Q(st, at) = rt+1 + \gamma r_t+2 + ... \gamma^t+n-1 r_t+n + Q(s t+n, s t+n)
class ExperienceSourceFirstLast(ExperienceSource):
    #
    def __init__(self, env, agent, gamma, steps_count=1, steps_delta=1, vectorized=False)
    #
    def __iter(self)
```

#### PrioritizedReplayBuffer
``` python
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
``` python
class QLearningPreprocessor(BatchPreprocessor)
```

### 其他
#### ExperienceSourceRollouts
```  python
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
``` python
class ExperienceReplayNaive
```

## 代码解析
### ExperienceSource
```  python
class ExperienceSource 
    """
    简单的n-step source for single or multiple envs
    每一个experience都有n个Experience entries的list
    """
    # 1.
    def __init__(self, env, agent, steps_count=2, steps_delta=1, vectorized=False):
        """
        env: 环境或者list of环境
        agent: 将observation 转换为actions
        steps_count: 每一个experience chain的计数
        steps_delta: experience items之间相隔多少个steps
        vectorized: bool,OpenAI vectorized
        """
        assert isinstance(env, (gym.Env, list, tuple))
        assert isinstance(agent, BaseAgent)
        assert isinstance(steps_count, int)
        assert steps_count >= 1
        assert isinstance(vectorized, bool)
        # self.pool存放envs list，类型是list
        if isinstance(env, (list, tuple)):
            self.pool = env
        else:
            self.pool = [env]
        self.agent = agent
        self.steps_count = steps_count
        self.steps_delta = steps_delta
        self.total_rewards = []
        self.total_steps = []
        self.vectorized = vectorized

    # 2.
    def __iter__(self):
        # states记录所有env的所有obs
        # agent_states记录
        # histories记录当前的steps_count个experience
        # cur_rewards记录当前episode的rewards
        # cur_steps记录当前episode的steps
        states, agent_states, histories, cur_rewards, cur_steps = [], [], [], [], []

        # env个数， 记录每个env observation的shape，如果向量化，[>=1, >=1, ...]，如果不向量化[1, 1, 1,...]，不管是否向量化，长度都等于envs个数的list，
        env_lens = [] 

        # 对每一个env进行操作，生成需要记录变量的维度
        for env in self.pool:
            obs = env.reset()
            # if the environment is vectorized, all it's output is lists of results.
            # 生成state的维度
            # 如果向量化obs，states的维度大于等于env数，否则states的维度和env数量一样
            if self.vectorized:
                obs_len = len(obs)
                states.extend(obs)
            else:
                obs_len = 1
                states.append(obs)

            # [env1_obs_len, env_2_obs_len, ..., envn_obs_len]
            env_lens.append(obs_len)

            # 生成histories, cur_rewards，cur_steps，agent_states的维度
            for _ in range(obs_len):
                # 记录所有env中所有的obs
                histories.append(deque(maxlen=self.steps_count))
                cur_rewards.append(0.0)
                cur_steps.append(0)
                agent_states.append(self.agent.initial_state())

        # 总的迭代次数，用于steps_delta
        iter_idx = 0
        # ================================分界线===========================
        # 上面是iteration的初始化，生成各类信息(states, agent_states, histories)的初始值。
        # 下面开始iteration
        while True:
            # 1.根据states生成len(states)个action
            # states >= len(envs_len) 个数
            actions = [None] * len(states)
            states_input = []
            states_indices = []
            for idx, state in enumerate(states):
                if state is None:
                    actions[idx] = self.pool[0].action_space.sample()  # assume that all envs are from the same family
                else:
                    states_input.append(state)
                    states_indices.append(idx)
            if states_input:
                states_actions, new_agent_states = self.agent(states_input, agent_states)
                for idx, action in enumerate(states_actions):
                    g_idx = states_indices[idx]
                    actions[g_idx] = action
                    agent_states[g_idx] = new_agent_states[idx]
            # 根据env_lens将actions进行合并，长度和envs number一样
            grouped_actions = _group_list(actions, env_lens)

            # 2.step执行action
            # global_of是全局offset，相当于每个env obs初始位置，ofs是每个env的len(action_n)个维度。
            global_ofs = 0

            # 分别对每个env执行相应的action_n个动作
            for env_idx, (env, action_n) in enumerate(zip(self.pool, grouped_actions)):
                if self.vectorized:
                    next_state_n, r_n, is_done_n, _ = env.step(action_n)
                else:
                    next_state, r, is_done, _ = env.step(action_n[0])
                    next_state_n, r_n, is_done_n = [next_state], [r], [is_done]

                # ofs是每个env的局部offset
                for ofs, (action, next_state, r, is_done) in enumerate(zip(action_n, next_state_n, r_n, is_done_n)):
                    idx = global_ofs + ofs
                    state = states[idx]
                    history = histories[idx]

                    cur_rewards[idx] += r
                    cur_steps[idx] += 1
                    if state is not None:
                        # 记录state, action, reward, done，没有记录next_state，使用完之后，更新state[idx] = next_state
                        history.append(Experience(state=state, action=action, reward=r, done=is_done))
                    # 每一个env在enumerate时都会算一次
                    if len(history) == self.steps_count and iter_idx % self.steps_delta == 0:
                        yield tuple(history)
                    # 更新下一步，states[idx] = next_state
                    states[idx] = next_state
                    if is_done:
                        # in case of very short episode (shorter than our steps count), send gathered history
                        if 0 < len(history) < self.steps_count:
                            yield tuple(history)
                        # generate tail of history
                        while len(history) > 1:
                            history.popleft()
                            yield tuple(history)
                        self.total_rewards.append(cur_rewards[idx])
                        self.total_steps.append(cur_steps[idx])
                        cur_rewards[idx] = 0.0
                        cur_steps[idx] = 0
                        # vectorized envs are reset automatically
                        states[idx] = env.reset() if not self.vectorized else None
                        agent_states[idx] = self.agent.initial_state()
                        history.clear()
                global_ofs += len(action_n)
            iter_idx += 1
    """
    重写for循环的iter方法
    """
    # 3. 重置total_rewards
    def pop_total_rewards(self)
    # 4. 重置total rewards和total steps
    def pop_rewards_steps(self)
```

### ExperienceSourceFirstLast
```  python
class ExperienceSourceFirstLast(ExperienceSource):
        """
    def __init__(self, env, agent, gamma, steps_count=1, steps_delta=1, vectorized=False):
        assert isinstance(gamma, float)
        super(ExperienceSourceFirstLast, self).__init__(env, agent, steps_count+1, steps_delta, vectorized=vectorized)
        self.gamma = gamma
        self.steps = steps_count

    def __iter__(self):
        # 并不保留中间n步的experience，因为没必要，只留第一步和最后一步，中间计算rewards就行了
        for exp in super(ExperienceSourceFirstLast, self).__iter__():
            if exp[-1].done and len(exp) <= self.steps:
                last_state = None
                elems = exp
            else:
                last_state = exp[-1].state
                elems = exp[:-1]
            total_reward = 0.0
            # 计算中间的rewards
            for e in reversed(elems):
                total_reward *= self.gamma
                total_reward += e.reward
            yield ExperienceFirstLast(state=exp[0].state, action=exp[0].action,
                                      reward=total_reward, last_state=last_state)
```

### ExperienceReplayBuffer
``` python
class ExperienceReplayBuffer
```


## 参考文献
1.https://github.com/Shmuma/ptan/blob/master/docs/intro.ipynb
