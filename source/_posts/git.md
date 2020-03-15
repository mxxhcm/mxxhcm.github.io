---
title: linux git
date: 2019-05-07 16:56:44
tags:
 - git
 - 工具
 - linux
categories: 工具
---

## linux安装github
### 安装
~$:sudo apt-get install git git-core git-gui

## 添加ssh公钥到github
~$:cd ~/.ssh
查看是否有ssh keys，没有的话执行下一步	
~$:ssh-keygen -t rsa -C "github 邮箱"
在github上登陆自己的账号，找到Settings->SSH Keys -> ADD SSH Key 将id_rsa.pub文件中的字符串复制进去，不含空格和回车。

## 配置本地文件
~$:git config  --gloabl user.name "github用户名"
~$:git config  --gloabl user.email "github邮箱"

## 测试
在github上新建repository 名为 testgit
~$:cd mkdir /home/mxx/github/testgit
~$:cd github/testgit
~$:git init
~$:touch Readme
~$:git add Readme
~$:git commit -m 'add readme file'

还可以直接克隆一个已有的仓库，
~$:git clone https://github.com/github用户名/github仓库.git

## push使用ssh，不用输入密码
``` shell
git remote -v  # 查看远程连接的方式
git remote rm origin # 删除https的连接方式，如果是ssh的方式，就不需要了
# 从github复制ssh地址
git remote add origin git@github.com:mxxhcm/**.git # 添加ssh连接方式
git push --set-upstream origin master
git remote -v  # 再次查看远程连接的方式
```
## git的区域
git的区域分为工作区，git一个项目后，能看到的目录，不包含.git目录。
.git所在目录是版本库，版本库包含叫做stage的暂存区。以及创建的各个branch。
git add 操作的是暂存区。
git commit是把暂存区的所有东西提交到当前的branch。

## 分支管理
~$:git branch dev 创建分支
~$:git checkout dev 切换分支
~$:git branch 查看当前分支
~$:git merge dev 将分支合并
~$:git brancd -d dev 删除分支

## 撤销
### git add加入缓存区后撤销
~$:git status # 查看add的文件
~$:git reset HEAD  # 撤销上一次add的内容
~$:git reset HEAD  xx.file # 撤销上一次add的某个文件

### git commit后撤销
~$:git log # 查看提交记录
找到相应commit的id，这个commit id不是刚才提交记录的id，而是他之前的那一个
commit xxxxxx
~$:git reset commit_id # 回退到git add之前，所有代码保留
~$:git reset --hard commit_id # 回退到上一次commit后，所有之后的代码都删掉

### git reset --hard错误回退后撤销
使用git reflog
~$:git reflog # 找到所有的commit id，然后
~$:git reset commit_id
~$:git reset --hard commit_id

### push之后撤销
使用git revert 
~$:git revert HEAD # 撤销前一次的commit
~$:git revert HEAD^ # 撤销前前一次的commit
~$:git rever commit-id # 撤销commit-id对应的版本

git revert会提交一个新的版本，将回退当做新的一个push，之前的内容都会保留。

## git pull和git fetch区别
本地的一个git项目中，一般都有本地仓库和远程仓库。一般写完代码，git add添加到本地缓冲区，git commit提交到本地仓库。这个时候假设本地仓库和远程仓库是一致的。
git pull 相当于git fetch和git merge。git merge将某个分支合并到master分支上。有可能会引发冲突。
然后假设有人更新了远程仓库的代码，这个时候可以使用git pull或者git fetch进行更新。
使用git fetch的时候，本地仓库中代码的commit id不变，而本地跟踪的远程仓库的commit id是会变的和远程仓库一致的。

## 参考文献
1.https://blog.csdn.net/kongbaidepao/article/details/52253774
2.https://www.cnblogs.com/ruiyang-/p/10764711.html
