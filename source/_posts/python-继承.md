---
title: python 继承
date: 2018-10-23 21:27:43
tags:
 - python
 - 面向对象
categories: python
---

## 继承
和C++一样，python也是面向对象的变成语言，也支持封装继承和多态。python同时支持单继承和多继承。
继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。
通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”，继承的过程，就是从一般到特殊的过程。在某些语言中，一个子类可以继承多个基类。但是一般情况下，一个子类只能有一个基类，要实现多重继承，可以通过多级继承来实现。
继承概念的实现方式主要有2类：实现继承、接口继承。　
- 实现继承是指使用基类的属性和方法而无需额外编码的能力。
- 接口继承是指仅使用属性和方法的名称、但是子类必须提供实现的能力(子类重构爹类方法)

## python继承定义
父类定义
``` python
class Person(object):
    def gender(self):
        return "Unknown"

```
子类定义
``` python
class Man(Person):
    def gender(self):
        return "male"
```

## python构造函数继承
父类构造函数
``` python
class Person(object):
    def __init__(self, name):
        self.name = name
```
子类构造函数
``` python
class Woman(Person):
    def __init__(self, name):
        super(Person, self).__init__(name)
```
或者
``` python
class Woman(Person):
    def __init(self, name):
        Person.__init__(self, name)
```

## python接口继承
父类
``` python
class Person(object):
    def __init__(self, name):
        self.name = name

    def gender(self):
        return NotImplentedError
```
子类
``` python
class Man(Person):
    def __init__(self, name):
        super(Man, self).__init__(name)
        self.name = name

    def gender(self):
        return "male"
```

## 代码示例
``` python
class Person(object):
    def __init__(self, name):
        self.name = name

    def work(self):
        return NotImplementedError


class Chief(Person):
    def __init__(self, name, salary):
        super(Chief, self).__init__(name)
        self.salary = salary

    def work(self):
        print("cook...")


class Student(Person):
    def __init__(self, name, grades, person):
        super(Student, self).__init__(name)
        self.grades = grades
        self.person = person

    def work(self):
        print("study...")

class A(Student):
    def __init__(self, name, grades, person):
        super(A, self).__init__(name, grades, person)
        self.type = "A"


class B(A):
    def __init__(self, name, grades, person):
        super(B, self).__init__(name, grades, person)
        self.type = "B"



if __name__=="__main__":
    chief = Chief("chief_a", 100)
    student = Student("student", 200, chief)
    a = A("student_a", 300, student)
    b = B("student_b", 400, a)


    print(b.person.name)
    print(b.person.person.name)
    print(b.person.person.person.name)
```


## 参考文献
1.https://www.cnblogs.com/bigberg/p/7182741.html
