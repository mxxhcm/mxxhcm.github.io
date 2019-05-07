---
title: hexo常见问题
date: 2019-04-26 20:36:32
tags:
 - hexo 
categories: 工具
mathjax: true 
---

## 问题1
Error: pandoc exited with code 7: pandoc: Unknown extension: smart
> INFO  Start processing
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: pandoc exited with code 7: pandoc: Unknown extension: smart
    at ChildProcess.<anonymous> (/home/mxxmhh/github/blog/node_modules/hexo-renderer-pandoc/index.js:94:20)
    at emitTwo (events.js:126:13)
    at ChildProcess.emit (events.js:214:7)
    at maybeClose (internal/child_process.js:925:16)
    at Socket.stream.socket.on (internal/child_process.js:346:11)
    at emitOne (events.js:116:13)
    at Socket.emit (events.js:211:7) 
    at Pipe._handle.close [as _onclose] (net.js:567:12) 

### 解决方法
卸载pandoc
~\$:npm un hexo-renderer-pandoc --save

## 问题2
部分公式无法解析。
是因为markdown和mathjax的解析有一些冲突，按照参考文献$1$中进行修改即可，原因见[2]。
修改node_modules/kramed/lib/rules/inline.js文件，将第11行替换成"escape: /^\\([`*\[\]()#$+\-.!_>])/"，将第19行替换成"em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/"。（不用加双引号）

## 问题3
Ubuntu 16.04直接使用命令安装nodejs，版本太老，需要使用源代码安装
~$:sudo apt install nodejs npm
上述命令可以在Ubuntu 18.04直接使用。

## 参考文献
1.https://hexo-guide.readthedocs.io/zh_CN/latest/theme/[NexT]%E9%85%8D%E7%BD%AEMathJax.html
2.https://shomy.top/2016/10/22/hexo-markdown-mathjax/
