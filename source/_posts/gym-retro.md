---
title: gym retro
date: 2019-09-15 20:28:55
tags:
 - gym-retro
 - 强化学习
 - gym
categories: 强化学习
---

## Gym Retro
### 什么是Gym Retro？
将不同平台的video games都转换成gym environments。可以使用统一的gym接口进行管理。

### 包含哪些平台
- Atari
- NEC
- Nintndo
- Sega

### 包含哪些ROMs
- the 128 sine-dot by Anthrox
- Sega Tween by Ben Ryves
- Happy 10! by Blind IO
- 512-Colour Test Demo by Chris Covell
- Dekadrive by Dekadence
- Automaton by Derek Ledbetter
- Fire by dox
- FamiCON intro by dr88
- Airstriker by Electrokinesis
- Lost Marbles by Vantage

## 示例
### 安装gym retro
``` shell
pip3 install gym-retro
```

### 创建Gym Env
下面代码创建一个gym环境
``` python
import retro
env = retro.make(game='Airstriker-Genesis')
```
retro中的environment是从gym.Env类继承而来的。

### 默认ROM
Airstriker-Genesis的ROM是默认包含在gym-retro之中的，其他的一些ROMs需要手动添加。

### 所有的games
``` python
import retro
retro.data.list_games()
```
上述代码会列出所有的游戏，包含那些默认没有集成的ROMS中的。

### 手动添加ROMs
``` shell
python3 -m retor.import /path/to/your/ROMs/directory
```
上述代码将存放在某个路径下的ROMs拷贝到Gym Retro的集成目录中去。

## 参考文献
1.https://retro.readthedocs.io/en/latest/
2.https://arxiv.org/pdf/1804.03720.pdf

