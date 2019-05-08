---
title: tensorflow app代码及例子
date: 2019-05-08 17:35:39
tags:
 - python
 - tensorflow
categories: tensorflow
---

## tf.app.flags
``` python
flags.py
import tensorflow as tf

flags = tf.app.flags
flags.DEFINE_string('model', 'mxx', 'Type of model')
flags.DEFINE_boolean('gpu','True', 'use gpu?')
FLAGS = flags.FLAGS

def main(_):
    for k,v in FLAGS.flag_values_dict().items():
        print(k, v)

if __name__ == "__main__":
    tf.app.run(main)
```
传递参数的方法有两种，一种是命令行~$:python flags.py --model hhhh ，一种是pycharm中传递参数。
