---
title: markdown帮助
date: 2019-03-09 19:53:32
tags:
 - markdown
categories: 工具
---

## 引用
### 代码引用
``` python
import tensorflow as tf
```

### 文字引用
> 实际是人类进步的阶梯。　－－高尔基


## 表格
|  name | age | gender|
| :-: | :-: | :-: |
|Alice|11 | female|  
| Bob| 82 | male |

## 表情
### 安装过程
第一步，卸载hexo默认的hexo-renderer-marked markdown渲染器
~$:npm un hexo-renderer-marked --save
第二步，安装支持emoji的markdown渲染器
~$:npm i hexo-renderer-markdown-it --save
第三步，修改博客根目录下的\_config.yml文件，添加下列内容：
> #Markdown-it config
#Docs: https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: true
    linkify: true
    typographer: true
    quotes: '“”‘’'
  plugins:
    - markdown-it-abbr
    - markdown-it-footnote
    - markdown-it-ins
    - markdown-it-sub
    - markdown-it-sup
    - markdown-it-emoji  ## add emoji
  anchors:
    level: 2
    collisionSuffix: 'v'
    # If `true`, creates an anchor tag with a permalink besides the heading.
    permalink: false  
    permalinkClass: header-anchor
    # The symbol used to make the permalink
    permalinkSymbol: ¶

然后重新生成部署即可。
测试：
:smile:
:laughing:
:nose:

## 参考文献
1.https://daringfireball.net/projects/markdown/syntax
2.https://www.webfx.com/tools/emoji-cheat-sheet/
3.https://guides.github.com/features/mastering-markdown/
4.https://github.com/mxxhcm/use_vim_as_ide#8.4
5.https://chaxiaoniu.oschina.io/2017/07/10/HexoAddEmoji/
