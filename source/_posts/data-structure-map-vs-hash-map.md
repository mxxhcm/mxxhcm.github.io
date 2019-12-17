---
title: data structure map vs hash_map
date: 2019-11-03 21:28:13
tags:
 - 数据结构
categories: 数据结构
mathjax: true
---

map vs hash_map(unordered_map)
- 数据结构，
map使用平衡二叉树，通常是红黑树；hash_map使用哈希函表。
- 查找时间
map是$O(\log n)$；hash_map是$O(1)$（没有冲突的情况下），最坏情况下是$O(n)$。

C++中的hash_map叫做unordered_map。


## 参考文献
1.https://stackoverflow.com/questions/2189189/map-vs-hash-map-in-c/2189206
2.https://stackoverflow.com/questions/5139859/what-the-difference-between-map-and-hashmap-in-stl/5139888

