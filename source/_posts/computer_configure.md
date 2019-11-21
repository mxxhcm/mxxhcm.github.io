---
title: 电脑配置
date: 2019-05-01 12:31:18
tags:
 - 硬件
categories: 工具
---

## SSD
### 物理接口
常见的物理接口，就是和主板相连的接口形状有SATA和M.2(NGFF)。
M.2接口也叫NGFF，有两种接口模式，socket2和socket3。socket2对应的接口是bkey，对应的是传输模式为SATA，对传输模式为SATA。而socket3对应的接口是mkey，走的是PCIE。

### 总线(bus)方式（协议通道）
目前市面上SSD的总线有两种类型PCI-E和SATA。
PCIE是用来取代SATA的新总线接口。PCIE总线的上层协议可以是NVME，也可以是ACHI。比如著名的sm951，既有NVME协议的也有ACHI协议的版本。[4]

### 上层协议（逻辑设备接口标准）
SSD的传输协议有NVME, IDE和AHCI。NVME是最新的高性能和优化协议，是用来取代AHCI的，NVME支持PCI-E，但是支持PCI-E的SSD不一定支持NVME协议。
NVME需要硬盘和主板M.2插槽都支持。
SATA采用AHCI协议，也支持IDE协议，是为寻道旋转磁盘而不是闪存设计的。

### 总结
M.2是物理接口形式，SATA可以指的是接口，也可以指的是总线方式。M.2接口也可以走SATA总线，本质上还是sata硬盘，只不过用的是m.2的接口，只有走PCIE总线的使用Nvme协议的m.2的固态硬盘才是真正跟stata硬盘有区别的。[2]
如下图所示，是所有接口，
![ssd](ssd.jpg)
上图来源见参考文献[3]。

## CPU
CPU后缀名字介绍

### 笔记本后缀
Y超低压处理器
U代表低电压
M代表标压
H高电压不可拆卸
X代表高性能
Q代表4核心至高性能处理器

### 台式机后缀
X至高性能处理器
E嵌入式工程级处理器
S低电压处理器
K不锁倍频处理器
T超低电压处理器
P屏蔽集显处理器


## GPU
显卡的话，好像也没啥要说的了。。。

## 写在最后
好吧，看了很多电脑，神舟现在缩水很厉害，把蓝天的p系列模具的散热管去掉了很多。买了gx9之后，还是有点后悔，看上了蓝天准系统，但是太贵了，总共要13000了，i7-8700+ rtx2070，自己暂时也完全发挥不了它的性能。就先这样子把。以后有钱了再说～。

## 参考文献
1.https://www.jianshu.com/p/6db2a47fdf60
2.https://www.zhihu.com/question/52811023/answer/132388287
3.https://www.zhihu.com/question/52811023/answer/527580986
4.https://www.zhihu.com/question/52811023/answer/132430870
