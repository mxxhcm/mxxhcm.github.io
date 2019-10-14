---
title: docker
date: 2019-10-14 15:42:19
tags:
 - 工具
 - docker
categories: 工具
---

## 什么是docker
Docker是一个开源的应用容器引擎，开发者可以打包应用及依赖包到一个可移植的容器中，然后发布到其他机器上，与平台，硬件等无关。容器是完全使用沙箱机制，相互之间不会有任何接口。

## Docker构成
Docker包含三个基本概念，镜像，容器和仓库。
Docker image(镜像)是用于创建Docker容器的模板，其中包含了应用程序所需的各项资源。
Docker container(容器)是独立运行的一个或者一组应用，容器和镜像的关系相当于面向对象中的对象和类。
Docker registry(仓库)用来存放镜像，可以理解为代码控制中的代码仓库，默认的docker仓库是Docker Hub。

以及三个组件：Docker client，Docker host和Docker Index
Docker client(客户端)，用户通过Docker client通过命令行或者其他工具使用Docker API与Docker host(Docker daemon)通信。
Docker host(主机)，包含docker镜像并以容器形式运行镜像的主机，或者说运行Docker daemon和容器的主机。
DockerMachine是一个Docker安装的命令行工具。

## Ubuntu Docker安装
1. 获取最新版本的docker
``` shell
wget -qO- https://get.docker.com/ | sh
# 从https://get.docker.com下载安装脚本，通过管道执行
```
报错：
``` txt
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/images/json: dial unix /var/run/docker.sock: connect: permission denied
```
是因为docker daemon使用socker at unix，需要root权限才可以访问，解决方案有以下两种：
1.使用sudo
2.docker daemon启动时，会给docker用户组读写unix socket的权限，所以只需要将当前用户加入docker用户组即可：
``` shell
sudo groudadd docker    # 添加docker用户组
cat /etc/group | grep 'docker'  # 查看docker group中的成员
sudo gpasswd -a $USER docker    # 将当前用户加入到docker用户组中
newgrp docker   # 更新docker用户组
cat /etc/gropu | grep 'docker'  # 查看docker group中是否有当前用户
```
2. 启动docker后台服务
``` shell
sudo service docker start
```
3. 测试
``` shell
docker run hello-world
```
4. 更换docker镜像
修改/etc/docker/daemon.json文件，在其中加入：
``` txt 
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

## Docker命令
### 容器生命周期管理
- run
- start/stop/restart
- kill
- rm
- pause/unpause
- create
- exec

### 容器操作
- ps
- inspect
- top
- attach
- events
- logs
- wait
- export
- port

### 容器rootfs命令
- commit
- cp
- diff

### 镜像仓库
- login
- pull
- push
- search

### 本地镜像管理
- images
- rmi
- tag
- build
- history
- save
- load
- import

### info/version
- info
- version


## Dockerfile
在Dockerfile中，每一条指令都会创建一个镜像层，增加整体镜像的大小。

## 在Docker上运行应用程序
### 从Docker仓库pull镜像
#### 过程
首先安装docker和编辑器，然后拉取docker镜像打包的依赖资源。
1. pull一个镜像
2. 使用容器运行这个镜像

Docker client利用pull命令从Docker registry拉取Docker Image到Docker host，然后使用run命令在Docker host上运行一个Docker container运行拉取的Docker image。

#### 命令
Docker pull命令的格式：
docker pull \[选项\] \[Docker Registry地址\] \<仓库名\>:\<标签名\>
默认的Docker registry地址是DockerHub。仓库名：这里的仓库名是两段式名称，既 / ，“/”前面一般是用户名。对于 Docker Hub，如果不给出用户名，则默认为 library ，也就是官方镜像。
示例
``` shell
docker pull ubuntu:14.04 
# 完整的仓库名和标签名是libary/ubuntu:14.04
```

### Push镜像到Docker仓库
或者反过来，Docker client通过build命令在Docker host创建一个子集的Docker image，然后Docker client使用push命令将Docker iamge推送到Docker Registry。这个Docker Image就可以被自己或者其他人pull了。


## 参考文献
1.https://zhuanlan.zhihu.com/p/66857204
2.https://www.cnblogs.com/chenxuming/p/9682571.html
3.https://www.cnblogs.com/wushuaishuai/p/9984228.html
4.https://www.cnblogs.com/informatics/p/8276172.html
