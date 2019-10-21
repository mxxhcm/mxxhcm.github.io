---
title: 'python special method'
date: 2019-04-13 14:41:38
tags:
 - python
categories: python
---

## 结论
print(object)就是调用了类对象object的__repr__()函数
如下代码
``` python
class Tem:
  def __init__(self):
     pass

  def __repr__(self):
     return "tem class"
```

声明类对象 
>>>Tem tem
下面两行代码的功能是一样的。
>>>print(tem)
>>>print(repr(tem))

## 基本的自定义方法
### object.__new__
### object.__init__
### object.__repr__和object.__str__
``` python
class Tem(object):
  pass

class TemStr(object):
  def __str__(object):
    return 'foo'

class TemRepr(object):
  def __repr__(object):
    return 'foo'

class TemStrRepr(object):
  def __repr__(object):
    return 'foo'
  def __str__(object):
    return 'foo_str'
if __name__ == "__main__": 
   tem = Tem() 
   print(str(tem)) 
   print(repr(tem)) 
   tem_str = TemStr() 
   print(str(tem_str)) 
   print(repr(tem_str)) 
   tem_repr = TemRepr() 
   print(str(tem_repr)) 
   print(repr(tem_repr)) 
   tem_str_repr = TemStrRepr() 
   print(str(tem_str_repr)) 
   print(repr(tem_str_repr)) 
```
单独重载__repr__，__str__也会调用__repr__，
但是单独重载__str__,__repr__不会调用它。
__repr__面向的是程序员，而__str__面向的是普通用户。它们都用来返回一个字符串，这个字符串可以是任何字符串，我觉得这个函数的目的就是将对象转化为字符串。

   
### object.__bytes__

## 自定义属性方法

### object.__getattr__(self, name)

### object.__setattr__(self, name)

## 比较
### object.__eq__(self, others)
### object.__lt__(self, others)
### object.__le__(self, others)
### object.__ne__(self, others) 
### object.__gt__(self, others)
### object.__ge__(self, others)

## 特殊属性
### object.__dict__
### instance.__class__

