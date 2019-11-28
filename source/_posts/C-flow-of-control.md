---
title: C++ flow of control
date: 2019-11-10 12:23:54
tags:
 - for
 - while
 - if 
 - else
 - switch
 - try
 - catch
 - C/C++
categories: C/C++
---

## 范围for语句 
语法格式：
``` c
for(declaration: expression)
    statement
```

`expression`表示的必须是一个序列，比如花括号括起来的初始化值列表，数组，或者`vector`,`string`等类型的对象，这些类型的共同特点是拥有能返回迭代器的`begin`和`end`成员。
`declaration`定义一个变量，序列中的每个元素都能转换成该变量的类型，最简单的方法就是使用`auto`语句。
每次迭代都会重新定义循环控制变量，并将其初始化成序列中的下一个值，之后才会执行`statement`。
范围`for`语句的定义来源于和它等价的传统`for`语句。在范围`for`语句中，相当于预存了`end()`的值，如果改变序列的话，`end()`的值可能会变得无效了。

## try block和异常处理
异常处理机制为程序中异常检测和异常处理这两部分的写作提供支持。在C++中，异常处理包括：
- `throw`表达式，异常检测部分使用`throw`表达式来表示它遇到了无法处理的问题。我们说`throw`引发了异常。
- `try`语句块，异常处理部分使用`try`语句块处理异常。`try`语句块以关键字`try`开始，以一个或者多个`catch`子句结束。`try`语句块中代码抛出的异常通常会被某个子句处理。因为`catch`子句处理异常，它们也被称为异常处理代码。
- `一套异常类，用于在`throw`和`catch`语句之间传递异常的具体信息。

### `try`语句块和`throw`表达式 
如下代码所示，`try`语句块中是程序的正常逻辑，和其他任何块一样。`try`语句块的变量在块外部无法访问，即使实在`catch`块内。
`try`语句块对应一个`catch`子句，这个子句负责处理`runtime_error`的异常。如果`try`语句块的代码中跑出了`runtime_error`的异常，接下来执行`catch`块中的语句。`catch`块中的内容进行异常处理，这里是输出了`err.what`的返回信息。`runtime_error`是标准库异常类型的一种，所有的标准库异常类都定义了`what`的成员函数，这些函数不需要参数，返回值是C风格字符串。
``` c
while(cin >> item1 >> item2)
{

    try{
        
        if(item1.isbn()!=  item2.isbn())
            throw runtime_error("data must refer to same ISBN.\n");
    }
    catch (runtime_error err)
    {
        cout << err.what() < "\n Try again ?" << endl;
        char c ;
        cin >> c;
    if(!cin || c = 'n')
        break;

    }
}
```

### 标准异常
C++标准库定义了一组类，用于报告函数库遇到的问题。这些异常类也可以用在用户编写的程序中，它们分别定义在4个头文件中：
- `exception`头文件中定义了最通用的异常类`exception`，它只报告异常类的发生，不做任何处理。
- `stdexcept`头文件定义了集中常用的异常类，如下所示：
    - exception
    - runtime_error
    - range_error
    - overflow_error
    - underflow_error
    - logic_error
    - domain_error
    - invalid_argument
    - length_error
    - out_of_range
我们只能以默认初始化
- `new`头文件定义了`bad_alloc`异常类型。
- `type_info`定义了`bad_cast`异常类型。

标准库只定义了几种运算，包括创建或者拷贝异常类型的对象，以及为异常类型的对象赋值。
我们只能以默认初始化的方式初始化`exception`, `bad_alloc`和`bad_cast`对象，不允许为这些对象提供初始值。
其他异常类型的行为恰好相反，应该使用`string`对象或者C风格字符串进行初始化，但是不允许使用默认初始化的方式。创建此类对象时，必须提供初始值。

## 参考文献
1.《C++ Primer》第五版
