---
title: python defaultdict
date: 2019-04-25 10:24:36
tags:
- python
categories: python
---

## 使用defaultdict创建字典的值默认类型 
### 使用defaultdict创建值类型为dict的字典
如下示例
``` python
from collections import defaultdict

ddd = defaultdict(dict)
print(ddd)

m = ddd['a']
m['step'] = 1
m['exp'] = 3
print(type(m))
print(ddd)

m = ddd['b']
m['step'] = 1
m['exp'] = 3
print(ddd)
```
上述代码创建了一个dict，dict的value类型还是一个dict
> defaultdict(class 'dict'&gt , {})
&lt class 'dict'&gt 
defaultdict(&lt class 'dict'&gt , {'a': {'step': 1, 'exp': 3}})
defaultdict(&lt class 'dict'&gt , {'a': {'step': 1, 'exp': 3}, 'b': {'step': 1, 'exp': 3}})


### 使用defaultdict创建值类型为list的dict
如下示例
``` python
from collections import defaultdict

ddl = defaultdict(list)
print(ddl)
m = ddl['a']
print(type(m))
m.append(3)
m.append('hhhh')
print(ddl)
```
上述代码创建了一个dict，dict的value类型是一个list，输出如下
> defaultdict(&lt class 'list'&gt , {})
&lt class 'list'&gt 
defaultdict(&lt class 'list'&gt , {'a': [3, 'hhhh']})

### 代码
点击获得[完整代码](https://github.com/mxxhcm/myown_code/blob/master/tools/python/defaultdict_test.py)

## 参考文献
1.http://www.cnblogs.com/dancesir/p/8142775.html

