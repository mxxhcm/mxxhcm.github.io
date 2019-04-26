---
title: ubuntu 驱动安装
date: 2019-04-26 21:03:02
tags:
 - ubuntu
 - 显卡驱动
categories: 工具
maxhjax: true
---

## 步骤
卸载原有驱动
~$:sudo apt purge nvidia\*
禁用nouveau
~$:sudo vim /etc/modprobe.d/blacklist.conf
在文件最后添加
blacklist nouveau
更新内核
~$:sudo update-initramfs -u
使用如下命令，如果没有输出，即已经关闭了nouveau
~$:lsmod | grep nouveau 
关闭X service
~$:sudo service lightdm stop
接下来执行如下语句即可：
``` shell
sudo apt-get install build-essential pkg-config xserver-xorg-dev linux-headers-`uname -r`
sudo apt-get install mesa-common-dev
sudo apt-get install freeglut3-dev
sudo chmod a+x NVIDIA-Linux-x86_64-375.66.run
sudo sh NVIDIA-Linux-x86_64-375.66.run -no-opengl-files
sudo reboot
```

## 参考文献
1.http://gwang-cv.github.io/2017/07/26/Faster-RCNN+Ubuntu16.04+Titan%20XP+CUDA8.0+cudnn5.0/
