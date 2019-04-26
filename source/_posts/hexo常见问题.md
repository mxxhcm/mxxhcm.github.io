---
title: hexo常见问题
date: 2019-04-26 20:36:32
tags:
 - hexo 
categories: 工具
mathjax: true
---

## 问题$1$
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

解决方法
卸载pandoc，
~$:npm un hexo-renderer-pandoc --save

## 问题$2$


## 参考文献
1.https://hexo-guide.readthedocs.io/zh_CN/latest/theme/[NexT]%E9%85%8D%E7%BD%AEMathJax.html
