---
title: linux git
date: 2019-05-07 16:56:44
tags:
 - github
 - linux
categories: git
---

## linux安装github
### 安装
~$:sudo apt-get install git git-core git-gui
ssh -T git@gitgub.com
如果看到
``` txt
Warning: Permanently added ‘github.com,204.232.175.90’ (RSA) to the list of known hosts. 
Permission denied (publickey).
```
继续

### ssh秘钥
~$:cd ~/.ssh
查看是否有ssh keys，没有的话执行下一步	
~$:ssh-keygen -t rsa -C "github 邮箱"
在github上登陆自己的账号，找到Settings->SSH Keys -> ADD SSH Key 将id_rsa.pub文件中的字符串复制进去，不含空格和回车。
~$:ssh -T git$github.com

### 本地配置
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

## 远程库
~$:git remote add origin git@github.com:mxxhcm/testgit.git
~$:git remote add origin https://github.com/github用户名/github仓库.git
~$:git push -u origin master 把本地的master库和远程的master库连接起来，以后推送或者拉取就会简单
输入github账号和密码
　推送
~$:git push origin master
　拉取
~$:git pull	

还可以直接克隆一个已有的仓库，
~$:git clone https://github.com/github用户名/github仓库.git

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
找到相应commit的id
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


## 脚本地址

## 参考文献
1.https://blog.csdn.net/kongbaidepao/article/details/52253774

