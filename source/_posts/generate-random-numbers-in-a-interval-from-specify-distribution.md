---
title: generate random numbers in a interval from specify distribution
date: 2019-10-26 10:28:22
tags:
 - 概率论
categories: 概率论
---

## 问题
如何利用一个分布（高斯分布），生成指定区间内的随机数？
比如在[-100,100]区间内，利用生成分布产生200个随机数，这个其实是叫截断正态分布。
思路：
我的想法是先使用正态分布随机生成一个数，然后判断这个数是否是满足区间内，如果不满足，就生成下一个。

## 代码实现
### C
``` c
```

### python
``` python
```

### matlab
``` matlab
```

### R
``` R
rand_interval = function(min, max, number)
{
  sd = (as.numeric(max)-as.numeric(min))/3
  count = 0
  res = 1:number
  while(TRUE)
  {
    x = rnorm(1, mean=0,sd=sd)
    if(x >= min && x <= max)
    {
      count = count+1
      res[count] = x
    }
  }
  return (res)
}

min = -100
max = 100
number = 20
x1 = rand_interval(min, max, number)
x2 = rand_interval(min, max, number)
x1
x2
```

## 参考文献
1.https://www.ilovematlab.cn/thread-302139-1-1.html
2.https://stats.stackexchange.com/questions/113230/generate-random-numbers-following-a-distribution-within-an-interval
