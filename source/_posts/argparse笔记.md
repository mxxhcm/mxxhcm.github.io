---
title: argparse笔记
date: 2019-03-18 15:15:41
tags:
 - python
 - argparse
categories: python
---

## 简单的例子

### 创建一个parser

``` python
parser = argparse.ArgumentParser(description='Process Intergers')
```

### 添加参数
``` python
parser.add_argument(,,)
```

### 解析参数
``` python
arglist = parser.parse_args()
```
###
完整代码如下
```python
import argparse

def parse_args():
    parser = argparse.ArgumentParser("input parameters")
    parser.add_argument("--batch_size", type=int, default=32)
    parser.add_argument("--episodes", type=int, default=1)
    parser.add_argument("--lr", type=float, default=0.01)
    parser.add_argument("--momentum", type=float, default=0.9)
    args_list = parser.parse_args()
    return args_list

	
def main(args_list):
	print(args_list.batch_size)


if __name__ == "__main__":
    args_list = parse_args()
    main(args_list)
```



## ArgumentParser objects
> The ArgumentParser object will hold all the information necessary to parse the command line into python data types

``` python
class argparse.ArgumentParser(prog=None,usage=None,description=None,epilog=None,parents=[],formatter_class=argparse.HelpFormatter,prefix_chars='-',fromfile_prefix_chars=None,argument_default=None,conflict_handler='error',add_help=True)
```

### 创建一个名为test.py的程序如下
``` python
import argparse
parser = argparse.ArgumentParser()
args = parser.parse_args()
```
~#:python test.py -h
> usage: test.py [-h]

> optional arguments:
  -h, --help  show this help message and exit

### prog参数
设置显示程序的名称
#### 直接使用默认显示的程序名
~#:python test.py -h
> usage: test.py [-h]

> optional arguments:
  -h, --help  show this help message and exit

#### 使用prog参数进行设置
修改test.py的程序如下
``` python
import argparse
parser = argparse.ArgumentParser(prog="mytest")
args = parser.parse_args()
```
~#:python test.py -h
> usage: mytest [-h]

> optional arguments:
  -h, --help  show this help message and exit

usage后的名称变为我们prog参数指定的名称

### usage
#### 使用默认的usage
#### 使用指定的usage

### description
#### 使用默认的description
#### 使用指定的description


### epilog
#### 使用默认的epilog
#### 使用指定的epilog

### parents

### formatter_class

### prefix_chars
指定其他的prefix，默认的是-，比如可以指定可选参数的前缀为+

### fromfile_prefix_chars

### argument_default

### conflict_handler
将conflict_handler设置为resolve就可以防止override原来older arguments
``` python
import argparse
parser = argparse.ArgumentParser(conflict_handler='resolve')
parser.add_argument('--foo','-f',help="old help")
parser.add_argument('-f',help="new_help")
parser.print_help()
```

### add_help
``` python
import argparse
parser = argparse.ArgumentParser(add_help=False)
parser.print_help()
```
输出
> usage: [-h]

> optional arguments:
  -h, --help  show this help message and exit
将add_help设置为false

``` python
import argparse
parser = argparse.ArgumentParser(add_help=False)
parser.print_help()
```
输出
> usage: 

## The add_argument() method
``` python
ArgumentParser.add_argument(name or flags...[,action],[,nargs],[,const],[,default],[,type],[,choices],[,required],[,help],[,metavar],[,dest])
```
### 例子
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f','-foo','-a', defaults=, type=, help=)
parser.add_argument('hello')
parser.add_argument('hi')
args = parser.parse_args(['Hello','-f','123','Hi'])
print(args)
```

### name or flags

#### 添加可选参数
parser.add_argument('-f', '--foo', '-fooo')

#### 添加必选参数
parser.add_argument('bar')

#### 调用parse_args()
当parse_args()函数被调用的时候，可选参数会被-prefix所识别，剩下的参数会被分配给必选参数的位置。如下代码中，'3'对应的就是'hello'的参数，'this is hi'对应的就是'hi'的参数，而'123'是'-f'的参数。

``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f','-foo','-a')
parser.add_argument('hello', type=int)
parser.add_argument('hi')
args = parser.parse_args(['3','-f','123','this is hi'])
print(args)
```
输出
> Namespace(f='123', hello='Hello', hi='Hi')

### action
#### store,the default action
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo')
args = parser.parse_args(['--foo','1'])
print(args)
```
输出
> Namespace(foo='1')

#### store_const
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', action='store_const', const=42)
args = parser.parse_args(['--foo')
print(args)
```
输出
> Namespace(foo=42)

#### store_true and store_false
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', action='store_true')
parser.add_argument('--bar', action='store_false')
parser.add_argument('--baz', action='store_false')
args = parser.parse_args('--foo --bar'.split())
print(args)
```
输出
> Namespace(bar=False, baz=True, foo=True)

这里为什么是这样呢，因为默认存储的都是True，当你调用--bar,--foo参数时，会执行action操作，会把action指定的动作执行

#### d.append
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', action='append')
args = parser.parse_args('--foo 1 --foo 2 --foo 3'.split())
print(args)
```
输出
> Namespace(foo=['1', '2', '3'])

#### append_const
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--str', action='append_const',const=str)
parser.add_argument('--int', action='append_const',const=int)
args = parser.parse_args('--str --int'.split())
print(args)
```
输出
> Namespace(int=[<class 'int'>], str=[<class 'str'>])

#### count
统计一个keyword argument出现了多少次
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--co', '-c',action='count')
args = parser.parse_args(['-ccc'])
print(args)
```
输出
> Namespace(co=3)

#### help
``` python
import argparse
parser = argparse.ArgumentParser()
args = parser.parse_args('--help'.split())
print(args)
```
输出，如果是交互式环境的话，会退出python
> usage: [-h]

> optional arguments:
  -h, --help  show this help message and exit

#### version
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--version', action='version',version='version 3')
args = parser.parse_args('--version'.split())
print(args)
```
输出,如果是交互式环境的话，会退出python
> version 3

### nargs 指定参数个数
#### N
如果是可选参数的话，或者不指定这个参数，或者必须指定N个参数
如果是必选参数的话，必须指定N个参数，不能多也不能少，也不能为0个
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',nargs=3)
parser.add_argument('bar',nargs=4)
args = parser.parse_args('bar 3 4 5'.split())
print(args)
```
输出
> Namespace(bar=['bar', '3', '4', '5'], foo=None)

#### ?
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',nargs='?',const='c',default='d')
parser.add_argument('bar',nargs='?',default='d')
args = parser.parse_args('3'.split())
print(args)
args = parser.parse_args('3 --foo'.split())
print(args)
```
输出
> Namespace(bar='3', foo='d')
Namespace(bar='3', foo='c')

如果显式指定可选参数，但是不给它参数，那么如果有const的话，就会显示const的值，否则就会显示None

#### *
nargs设置为\*的话，不能直接用const=''来设置const参数，需要使用其他方式
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',nargs='*',default='d')
parser.add_argument('bar',nargs='*',default='d')
args = parser.parse_args('3 --foo 3 4'.split())
print(args)
```
输出
> Namespace(bar=['3'], foo=['3', '4'])

#### +
nargs设置为+，参数个数必须大于等于1
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',nargs='+',default='d')
parser.add_argument('bar',nargs='+',default='d')
args = parser.parse_args('3 3'.split())
print(args)
args = parser.parse_args('--foo 3'.split())
print(args)
```
输出
> Namespace(bar=['3'], foo='d')
Namespace(bar=['3'], foo=['3'])

### const
#### action=''store_const" or action="append_const"
the examples are in the action 
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', action='store_const', const=42)
args = parser.parse_args(['--foo')
print(args)
```
输出
> Namespace(foo=42)

#### like -f or --foo and nargs='?'
the examples are the same as examples in the nargs='?'
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',nargs='?',const='c',default='d')
parser.add_argument('bar',nargs='?',default='d')
args = parser.parse_args('3'.split())
print(args)
args = parser.parse_args('3 --foo'.split())
print(args)
```
输出
> Namespace(bar='3', foo='d')
Namespace(bar='3', foo='c')
如果显式指定可选参数，但是不给它参数，那么如果有const的话，就会显示const的值，否则就会显示None

### default
default对于可选参数来说，是有用的，当可选参数没有在command line中显示出来时被使用，但是对于必选参数来说，只有nargs=?或者\*才能起作用。
#### 对于可选参数
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo',default=43)
args = parser.parse_args([])
print(args)
args = parser.parse_args('--foo 3'.split())
print(args)
```
输出
> Namespace(foo='43')
Namespace(foo='3')

#### 对于必选参数
对于nargs=‘+’是会出错
```
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('bar',nargs='+',default='d')
args = parser.parse_args([])
print(args)
```
输出
> usage: [-h] bar [bar ...]
: error: the following arguments are required: bar

对于nargs=‘\*’或者nargs='?'就行了
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('bar',nargs='?',default='d')
args = parser.parse_args([])
print(args)
```
输出
> Namespace(bar='d')

### type
将输入的字符串参数转换为你想要的参数类型
对于文件类型来说，这个文件必须在当前目录存在。
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--door',type=int)
parser.add_argument('filename',type=file)
parser.parse_args(['--door','3','hello.txt'])
```
输出
> Namespace(door=3)
这里的door就是int类型的

### choices
输入的参数必须在choices这个范围中，否则就会报错
``` python
import argparse
parser = argparse.ArgumentParse()
parser.add_argument('--door',type=int,choices=range(1,9))
parser.parse_args(['--door','3'])
```
输出
> Namespace(door=3)

### required
如果将required设置为True的话，那么这个可选参数必须要设置的
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f', '--foo-bar', '--foo',required=True)
```

### help
help可以设置某个参数的简要介绍。
使用help=argparse.SUPRESS可以在help界面中不显示这个参数的介绍
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f', '--foo-bar', '--foo',help='fool you ')
parser.add_argument('-xs', '--y',help=argparse.SUPPRESS)
parser.print_help()
```
输出
> usage: [-h] [-f FOO_BAR]

> optional arguments:
  -h, --help            show this help message and exit
  -f FOO_BAR, --foo-bar FOO_BAR, --foo FOO_BAR
                        fool you

### dest
dest就是在help输出时显示的optional和positional参数后跟的名字（没有指定metavar时）
如下,dest就是FOO
-foo FOO

#### positional argument
dest is normally supplied as the first argument to add_argument()

#### 可选参数
对于optional argument选择，--参数最长的一个作为dest，如果没有最长的，选择第一个出现的，如果没有--参数名，选择-参数。
``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f', '--foo-bar', '--foo')
parser.add_argument('-xs', '--y')
args = parser.parse_args('-f 1 -xs 2'.split())
print(args)
args = parser.parse_args('--foo 1 --y 2'.split())
print(args)
parser.print_help()
```
输出
> Namespace(foo_bar='1', y='2')
Namespace(foo_bar='1', y='2')
usage: [-h] [-f FOO_BAR] [-xs Y]

> optional arguments:
  -h, --help            show this help message and exit
  -f FOO_BAR, --foo-bar FOO_BAR, --foo FOO_BAR
  -xs Y, --y Y

### metavar
如果指定metavar变量名的话，那么help输出的postional和positional参数后跟的名字就是metavar的名字而不是dest的名字

``` python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-f', '--foo-bar', '--foo',metavar="FOO")
parser.add_argument('-xs', '--y',metavar='XY')
args = parser.parse_args('-f 1 -xs 2'.split())
print(args)
args = parser.parse_args('--foo 1 --y 2'.split())
print(args)
parser.print_help()
```
输出
> Namespace(foo_bar='1', y='2')
Namespace(foo_bar='1', y='2')
usage: [-h] [-f FOO] [-xs XY]

> optional arguments:
  -h, --help            show this help message and exit
  -f FOO, --foo-bar FOO, --foo FOO
  -xs XY, --y XY

