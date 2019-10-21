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
``` txt
# Markdown-it config
## Docs: https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki
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
```

然后重新生成部署即可。
测试：
:smile:
:laughing:
:nose:

## 测试
<table>
   <tr>
      <td></td>
   </tr>
   <tr>
      <td>Coding</td>
   </tr>
   <tr>
      <td>Content</td>
   </tr>
   <tr>
      <td>描述性提炼性质的研究</td>
   </tr>
   <tr>
      <td>第一部分：</td>
   </tr>
   <tr>
      <td>文献综述</td>
   </tr>
   <tr>
      <td>（对话）</td>
   </tr>
   <tr>
      <td>SPL</td>
   </tr>
   <tr>
      <td>本文的文献综述贯穿在行文的过程中</td>
   </tr>
   <tr>
      <td>（1）关于分家的原因：</td>
   </tr>
   <tr>
      <td>敌军：①弗里德曼兄弟之间的利害冲突②许烺光夫妻纽带强于父子之间的纽带→概括为家庭内摩擦</td>
   </tr>
   <tr>
      <td>作者（部分认同敌军基础上提出自己的观点）：分家逐渐演化成一种“文化现象”</td>
   </tr>
   <tr>
      <td>（2）分家中的“继”与“合”</td>
   </tr>
   <tr>
      <td>敌军：①孔迈隆以家产正式分才算分家的定义②分灶</td>
   </tr>
   <tr>
      <td>评论：①经济上的考虑多于社会上的考虑②认为分家是家庭的破裂以及兄弟没有继承一个完整的家庭</td>
   </tr>
   <tr>
      <td>作者：分家中也有垂直关系的“继承”、横纵一体的“合”</td>
   </tr>
   <tr>
      <td></td>
   </tr>
   <tr>
      <td>CPL</td>
   </tr>
   <tr>
      <td>①对现有文献的理解和看法②作者在哪个细分领域展开研究</td>
   </tr>
   <tr>
      <td>理论基础</td>
   </tr>
   <tr>
      <td>RAT</td>
   </tr>
   <tr>
      <td>上面两个部分的完善是为这个部分做准备</td>
   </tr>
   <tr>
      <td>第二部分：</td>
   </tr>
   <tr>
      <td>机制和结构</td>
   </tr>
   <tr>
      <td>F（x）</td>
   </tr>
   <tr>
      <td>结构： </td>
   </tr>
   <tr>
      <td>（1）概念界定：分家的基本内容</td>
   </tr>
   <tr>
      <td>什么是分家？分家时财产按照“股”分割；分家的原因</td>
   </tr>
   <tr>
      <td>（2）分家带来的影响（案例分析）：分家带来了社会流动</td>
   </tr>
   <tr>
      <td>  借用说“分家三年显高低”、“富不过三代”、“父子一条心，黄土变成金”三句俚语来说明分家对社会变化的影响</td>
   </tr>
   <tr>
      <td>（3）分家的中“继”与“合”</td>
   </tr>
   <tr>
      <td>  继：赡养老人、继宗祧（tiao 1声），对应儒的孝、父子一体观念</td>
   </tr>
   <tr>
      <td>  合：生产生活上的合作</td>
   </tr>
   <tr>
      <td>（4）结语</td>
   </tr>
   <tr>
      <td>机制：对应儒的孝、父子一体观念；生产生活上的合作</td>
   </tr>
   <tr>
      <td>Argument</td>
   </tr>
   <tr>
      <td>CA</td>
   </tr>
   <tr>
      <td>分中有继也有合</td>
   </tr>
   <tr>
      <td>第三部分：</td>
   </tr>
   <tr>
      <td>问题和发展</td>
   </tr>
   <tr>
      <td>    （1） 文献综述找敌军可以借鉴，以及文献评述</td>
   </tr>
   <tr>
      <td>    （2） 文献综述可以加一些友军</td>
   </tr>
   <tr>
      <td>    （3） 文章可能写的太早了，不太符合现在的文章写作规范。不太理解第二部分“分家对社会发展的影响”对整篇文章有什么关系？？是不是没有必要占这么大篇幅</td>
   </tr>
   <tr>
      <td>    （4） 内容方面：</td>
   </tr>
   <tr>
      <td>随着时间的演变，他们的合越来越局限于小，男女双方的直系亲属，直系的兄弟关系和姻亲关系，大家族的联系越来越少；大家族即使祖坟放在一起，也难以通过祭祀的手段联系起来，慢慢农村也形成原子化的家庭单位，对于同村的人来说，地缘关系、邻里关系是比血缘关系更重要的存在</td>
   </tr>
   <tr>
      <td></td>
   </tr>
</table>

## 参考文献
1.https://daringfireball.net/projects/markdown/syntax
2.https://www.webfx.com/tools/emoji-cheat-sheet/
3.https://guides.github.com/features/mastering-markdown/
4.https://github.com/mxxhcm/use_vim_as_ide#8.4
5.https://chaxiaoniu.oschina.io/2017/07/10/HexoAddEmoji/
