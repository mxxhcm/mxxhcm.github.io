---
title: '''dict_values'' object does not support indexing'
date: 2019-03-13 10:40:03
tags:
 - python3
categories: Error
---


## python3 dict
python3 中调用字典对象的一些函数，返回值是view objects。如果要转换为list的话，需要使用list()强制转换。
而python2的返回值直接就是list。
> The objects returned by dict.keys(), dict.values() and dict.items() are view objects. They provide a dynamic view on the dictionary’s entries, which means that when the dictionary changes, the view reflects these changes.

### 代码示例
```python
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
```
如果使用python3执行以上代码，输出结果如下所示：
> class 'dict_values'
dict_values([10, 20])

> class 'dict_items'
dict_items([('a', 10), ('b', 20)])

> class 'dict_keys'
dict_keys(['a', 'b'])

如果使用python2执行以上代码，输出结果如下所示：
> type 'list'                                                                           
[10, 20]                                                                                
                                                                                        
> type 'list'                                                                           
[('a', 10), ('b', 20)]                                                                  

> type 'list'
['a', 'b']

 
## 参考文献
1.https://www.cnblogs.com/timxgb/p/8905290.html
2.https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects
3.https://stackoverflow.com/questions/43663206/typeerror-unsupported-operand-types-for-dict-values-and-int
