---
title: python 类和函数的属性
date: 2019-04-14 14:49:41
tags:
 - python
categories: python
---

## 函数和类的默认属性
这里主要介绍类和函数的一些属性。
__dict__用来描述对象的属性。对于类来说，它内部的变量就是它的数量，注意，不是它的member variable，但是对于函数来说不是。对于类来说，而对于类对象来说，输出的是整个类的属性，而__dict__输出的是self.variable的内容。

python中的函数有很多特殊的属性（包括自定义的函数和库函数）
- __doc__  输出用户定义的关于函数的说明
- __name__ 输出函数名字
- __module__ 输出函数所在模块的名字
- __dict__ 输出函数中的字典

示例：
``` python
def myfunc():
   'this func is to test the __doc__'
   myfunc.func_attr = "attr"
   print("hhhh")
 
myfunc.func_attr1 = "first1"
myfunc.func_attr2 = "first2"
 
if __name__ == "__main__":
  print(myfunc.__doc__)
  print(myfunc.__name__)
  print(myfunc.__module__)
  print(myfunc.__dict__)
```
输出：
> this func is to test the __doc__
myfunc
__main__
{'func_attr1': 'first1', 'func_attr2': 'first2'}

类也有很多特殊的属性（包括自定义的类和库中的类）
- __doc__  输出用户定义的类的说明
- __module__ 输出类所在模块的名字
- __dict__ 输出类中的字典

示例：
``` python
class MyClass:
  """This is my class __doc__"""
  class_name = "cllll"
  def __init__(self, test=None):
     self.test = test
  pass


if __name__ == "__main__":
  print(MyClass.__dict__)
  print(MyClass.__doc__)
  print(MyClass.__module__)
```
输出：
> {'__module__': '__main__', '__doc__': 'This is my class __doc__', 'class_name': 'cllll', '__init__': \<function MyClass.__init__ at 0x7f1349d44510\>, '__dict__': \<attribute '__dict__' of 'MyClass' objects\>, '__weakref__': \<attribute '__weakref__' of 'MyClass' objects\>}
This is my class __doc__
__main__

类的对象的属性
- __doc__  输出用户定义的类的说明
- __module__ 输出类对象所在模块的名字
- __dict__ 输出类对象中的字典

示例
``` python
  1 class MyClass:
  2   """This is my class __doc__"""
  3   class_name = "cllll"
  4   def __init__(self, test=None):
  5      self.test = test
  6   pass
  7 
  8 if __name__ == "__main__":
  9 
 10   cl = MyClass()
 11   print(cl.__dict__)
 12   print(cl.__doc__)
 13   print(cl.__module__)
```
输出
> {'test': None}
This is my class __doc__
__main__

