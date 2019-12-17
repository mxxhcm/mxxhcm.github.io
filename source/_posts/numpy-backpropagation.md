---
title: pytohn numpy implements simple backpropagation
date: 2019-06-08 23:10:17
tags:
 - python
 - 机器学习
categories: 机器学习
---

## numpy 代码示例
``` python
import numpy as np

# N is batch size; D_in is input dimension;
# H is hidden dimension; D_out is output dimension.
N, D_in, H, D_out = 64, 1000, 100, 10

# Create random input and output data
x = np.random.randn(N, D_in)
y = np.random.randn(N, D_out)

# Randomly initialize weights
w1 = np.random.randn(D_in, H)
w2 = np.random.randn(H, D_out)

learning_rate = 1e-6
for t in range(500):
    # Forward pass: compute predicted y
    h = x.dot(w1)
    h_relu = np.maximum(h, 0)
    y_pred = h_relu.dot(w2)

    # Compute and print loss
    loss = np.square(y_pred - y).sum()
    print(t, loss)

    # Backprop to compute gradients of w1 and w2 with respect to loss
    grad_y_pred = 2.0 * (y_pred - y)
    grad_w2 = h_relu.T.dot(grad_y_pred)
    grad_h_relu = grad_y_pred.dot(w2.T)
    grad_h = grad_h_relu.copy()
    grad_h[h < 0] = 0
    grad_w1 = x.T.dot(grad_h)

    # Update weights
    w1 -= learning_rate * grad_w1
    w2 -= learning_rate * grad_w2
```

## 推理
![bp1](bp.jpg)
![bp2](bp2.jpg)

## 参考文献
1.http://pytorch.org/tutorials/beginner/pytorch_with_examples.html
2.https://datascience.stackexchange.com/questions/27506/back-propagation-in-cnn
3.https://medium.com/@14prakash/back-propagation-is-very-simple-who-made-it-complicated-97b794c97e5c
4.https://medium.com/@pavisj/convolutions-and-backpropagations-46026a8f5d2c
5.https://becominghuman.ai/back-propagation-in-convolutional-neural-networks-intuition-and-code-714ef1c38199
6.http://www.robots.ox.ac.uk/~vgg/practicals/cnn/
7.https://www.researchgate.net/post/How_backpropagation_works_for_learning_filters_in_CNN
8.https://github.com/ivallesp/awesome-optimizers
