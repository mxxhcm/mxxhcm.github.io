---
title: github push使用ssh
date: 2019-04-23 21:21:08
tags:
- github
categories: 工具
---

## github push时使用ssh,不用输入密码
``` shell
git remote -v  # 查看远程连接的方式
git remote rm origin # 删除https的连接方式，如果是ssh的方式，就不需要了
# 从github复制ssh地址
git remote add origin git@github.com:mxxhcm/**.git # 添加ssh连接方式
git push --set-upstream origin master
git remote -v  # 再次查看远程连接的方式
```

## 参考文献
