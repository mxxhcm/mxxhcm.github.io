---
title: matlab basic
date: 2019-10-26 16:16:58
tags:
 - matlab
 - 概率论与统计
categories: matlab
---

## Matlab基础

### 矩阵大小
``` matlab
x = ones(20, 3);
size(x);
```

### 矩阵列拼接用,
``` matlab
x1 = ones(200, 1);
x2 = ones(200, 1);
x3 = ones(200, 1);
x4 = ones(200, 1);
x5 = ones(200, 3);
x = [x1(:, 1), x2(:, 1), x5(:, 2:3)]
```

### 矩阵行拼接用;
``` matlab
x1 = ones(1, 200);
x2 = ones(1, 200);
x3 = ones(1, 200);
x4 = ones(1, 200);
x5 = ones(3, 200);
x = [x1(1, :), x2(1, :), x5(2:3, :)]
```


### 指数运算
``` matlab
x = ones(10, 1);
x2 = x.^2;
```

##  Matlab进行回归分析

## 标准差怎么计算

## 参考文献
1.https://www.cnblogs.com/hezhiyao/p/6824083.html
2.http://blog.sina.com.cn/s/blog_03f96e310106lktb.html
3.https://bbs.pinggu.org/thread-3065519-1-1.html
4.http://mcm.dept.ccut.edu.cn/u_newsfiles/1283049677/20120508/20120508204298149814.pdf
5.https://zhidao.baidu.com/question/1819455048729653268.html
