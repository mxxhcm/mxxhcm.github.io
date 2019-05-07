---
title: linux cpu信息
date: 2019-05-07 16:30:27
tags:
 - linux
categories: linux
---

## 查看cpu核数和cpu信息
~$:lscpu
~$:cat /proc/info

# 总核数 = 物理CPU个数 X 每颗物理CPU的核数
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

## 查看物理CPU个数
~$:cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

## 查看每个物理CPU中core的个数(即核数)
~$:cat /proc/cpuinfo| grep "cpu cores"| uniq

## 查看逻辑CPU的个数
~$:cat /proc/cpuinfo| grep "processor"| wc -l

## 参考文献
1.鸟哥的Linux私房菜
