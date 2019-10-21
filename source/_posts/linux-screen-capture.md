---
title: linux screen capture
date: 2019-06-22 21:48:27
tags:
 - linux
 - scrot
 - screenshot
categories: linux
---

## 常用的截屏工具
- screenshot
- scrot

screentshot是系统自带的截屏工具，可以部分截取，直接按win键搜索可运行。
scrot需要安装，这个工具需要从终端运行，截图默认保存在当前目录。

## screenshot

## scrot
scrot [options] [filename]
### 参数介绍
scrot [-vusbcm]
    不加参数，直接截取整个screen
    -v  查看scort版本
    -u  截取当前鼠标焦点所在window
    -s  执行命令之后，点击鼠标截取相对应的window。
    -b  包含当前window的边界
    --delay [NUM]   延长NUM秒之后截图
    -c  设置delay的时候显示数字延迟
    --quliaty [NUM] 指定screenshot image的质量，从$1-100$ 
    --thumb [NUM]   缩略图，是原始大小的百分之多少。。。

## 参考文献
1.https://www.howtoforge.com/tutorial/how-to-take-screenshots-in-linux-with-scrot/
2.http://awesomescreenshot.com/
