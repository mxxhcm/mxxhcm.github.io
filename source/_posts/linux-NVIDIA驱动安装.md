---
title: ubuntu NVIDIA 驱动安装
date: 2019-04-26 21:03:02
tags:
 - ubuntu
 - linux
 - 显卡驱动
categories: 工具
maxhjax: true
---


## 方法1.命令行安装
### 步骤
卸载原有驱动
~\\$:sudo apt purge nvidia\*
禁用nouveau
~\\$:sudo vim /etc/modprobe.d/blacklist.conf
在文件最后添加
blacklist nouveau
更新内核
~\\$:sudo update-initramfs -u
使用如下命令，如果没有输出，即已经关闭了nouveau
~\\$:lsmod | grep nouveau 
关闭X service
~\\$:sudo service lightdm stop
接下来执行如下语句即可：
``` shell
sudo apt-get install build-essential pkg-config xserver-xorg-dev linux-headers-`uname -r` sudo apt-get install mesa-common-dev
sudo apt-get install freeglut3-dev
sudo chmod a+x NVIDIA-Linux-x86_64-375.66.run
sudo sh NVIDIA-Linux-x86_64-375.66.run -no-opengl-files
sudo reboot
```


## 方法2.图形界面


## 方法3.自动安装
### 添加源
~$:sudo add-apt-repository ppa:graphicsw-drivers/ppa
~$:sudo apt update

### 自动安装
~$:sudo ubuntu-drivers devices
~$:sudo ubuntu-drivers autoinstall

### 更新grud
~$:sudo vim /etc/default/grub
将"splash"改为"splash acpi_osi=linux"
~$:sudo update-grub


## 安装cuda 9.0
到NVIDIA官网下载cuda 9.0的runfile，然后执行
~$:sudo sh cuda\*.run
### 测试报错
> Error: unsupported compiler: 7.4.0. Use --override to override this check.

安装gcc低版本
~$:sudo apt install gcc-6

从CUDA 4.1版本开始，支持gcc 4.5。gcc 4.6和4.7不受支持。
从CUDA 5.0版本开始，支持gcc 4.6。gcc 4.7不受支持。
从CUDA 6.0版本开始，支持gcc 4.7。
从CUDA 7.0版本开始，支持gcc 4.8，在Ubuntu 14.04和Fedora 21上支持4.9。
从CUDA 7.5版开始，支持gcc 4.8，在Ubuntu 14.04和Fedora 21上支持4.9。
从CUDA 8版本开始，Ubuntu 16.06和Fedora 23支持gcc 5.3。
从CUDA 9版本开始，Ubuntu 16.04，Ubuntu 17.04和Fedora 25支持gcc 6。
使用update-alternativess修改默认gcc版本
~$:sudo update--install /usr/bin/g++ g++ /usr/bin/g++-6 50
~$:sudo update--install /usr/bin/gcc gcc /usr/bin/gcc-6 50

然后继续安装：
~$:sudo sh cuda\*.run

cuda安装在/usr/local/cuda-9.0 目录下
卸载的话进入/usr/loca/cuda-9.0/bin 找到uninstall_cuda_9.0.pl运行卸载。

### import tensorflow 报错
> ImportError: libcublas.so.9.0: cannot open shared object file: No such file or directory
Failed to load the native TensorFlow runtime.

配置cuda环境变量
在bashrc文件中加入
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
执行
~$:source ~/.bashrc

继续报错
然后我才发现我没有装cudnn，按照参考文献[1]安装cudnn即可。
解压cudnn
~$:tar -xvf cudnn-x.x-linuz-x64-vx.x.tar.gz
然后执行
``` shell
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```
即可


## 版本对应
### 显卡
RTX 20系列显卡，需要使用cuda 10
### pytorch
而pytorch目前不支持cuda 10.1，所以只能使用cuda 10.0。
### tensorflow
te

## 参考文献
1.http://gwang-cv.github.io/2017/07/26/Faster-RCNN+Ubuntu16.04+Titan%20XP+CUDA8.0+cudnn5.0/
2.https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#axzz4qYJp45J2


