---
title: tensorflow multinomial代码及例子
date: 2019-05-08 17:37:45
tags:
- tensorflow
- python
categories: tensorflow
---

## tf.multinomial
多项分布，采样。
如下[示例](https://github.com/mxxhcm/myown_code/blob/master/tf/some_ops/tf_multinominal.py)
``` python
import tensorflow as tf

# tf.multinomial(logits, num_samples, seed=None, name=None)
# logits 是一个二维张量，指定概率，num_samples是采样个数
sess = tf.Session()
sample = tf.multinomial([[5.0, 5.0, 5.0], [5.0, 4, 3]], 10) # 注意logits必须是float
for _ in range(5):
  print(sess.run(sample))
```
输出结果如下:
> [[2 1 2 1 0 2 1 1 1 0]
 [1 0 0 1 0 1 0 1 0 0]]
[[2 2 0 2 2 0 2 0 1 2]
 [1 0 0 2 0 1 0 1 1 0]]
[[0 0 0 2 0 0 1 2 0 1]
 [0 0 0 1 0 1 0 0 0 0]]
[[2 1 0 1 1 1 0 0 2 0]
 [1 0 0 2 0 0 0 0 0 1]]
[[1 0 1 0 0 1 2 2 0 0]
 [1 0 0 0 0 1 1 1 2 0]]


