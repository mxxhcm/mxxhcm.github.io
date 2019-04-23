---
title: python2和python3中的dict
date: 2019-04-13 14:46:26
tags:
 - python2
 - python3
categories: python
---

## python2和python3的dict

### 将object转换为dict
vars([object]) -> dictionary
	
### python2 dict
``` python2
m_dict = {'a': 10, 'b': 20}

values = m_dict.values()
print(type(values))
print(values)
print("\n")

items = m_dict.items()
print(type(items))
print(items)
print("\n")

keys = m_dict.keys()
print(type(keys))
print(keys)
print("\n")

l_values = list(values)
print(type(l_values))
print(l_values)

输出：
```

### python3 dict
``` python3
m_dict = {'a': 10, 'b': 20}

values = m_dict.values()
print(type(values))
print(values) print("\n")

items = m_dict.items()
print(type(items))
print(items)
print("\n")

keys = m_dict.keys()
print(type(keys))
print(keys)
print("\n")

l_values = list(values)
print(type(l_values))
print(l_values)

```
输出：

