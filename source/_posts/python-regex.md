---
title: python regex
date: 2019-04-13 14:50:41
tags:
 - 正则表达式
 - python
categories: python
---


## regex（re库）

import re

### 如何创建正则表达式
.    match any character
\*    match 0 or more repetitons
\+    match 1 or more repetitions
?    match 0 or 1 repetition
^    matching the start of the string
$    matching the end os the string
+,\*.?    都是贪婪匹配，如果加一个?为非贪婪匹配
+?,\*?,??    为非贪婪匹配
{}    表示重复多少次a{3,5}
[]    匹配方括号内的,如[1-9]
()  match whatever regular expression is inside the parentheses,and indicated the start and end of a group.

### 模块

str = "https://abc https://dcdf https://httpfn https://hello"

#### re.compile(patern, flags=0)
将一个正则表达式语句编译成一个正则表达式对象
> complie a regular expression pattern into a regular expression object,which can be used for matching using its match() and search()

``` python
import regex as re
prog = re.compile(pattern)
results = prog.match(string)
# is equivalent to 
results = re.match(pattern, string)
```
flags can be re.DEBUG, re.I,re.IGNORECASE, re.L, re.LOCALE, re.M, re.MULTIINE, re.s, re.DOTALL, re.U, re.UNICODE, re.X, re.VERBOSE

#### re.search(pattern, string, flags=0)
在给定的string任意位置进行查找
locat a match anywhere in string

#### re.match(pattern, string, flags=0) or re.fullmatch(pattern,string,flags=0)
在给定的string开始位置进行查找
> match at the beginning of string, or if the whole string match the regular expression pattern.

``` python
p3 = "https.+? "
results = re.match(p3,str)
```
> <_sre.SRE_Match object; span=(0, 12), match='https://abc '>

#### re.split(pattern, string, maxflit=0, flags=0)
按照patten对string进行分割

``` python
import regex as re
str = "https://abc https://dcdf https://httpfn https://hello"
p1 = " "
results = re.split(p1,str)
```
> ['https://abc', 'https://dcdf', 'https://httpfn', 'https://hello']

#### re.findall(pattern,string,flags=0)
查找字符string所有匹配pattern的字符

``` python
import regex as re
str = "https://abc https://dcdf https://httpfn https://hello"
p2 = "https.+? "    # pay attention to space here
results = re.findall(p2,str)
```
> ['https://abc ', 'https://dcdf ', 'https://httpfn ']    # pay attention to the last ,since the end of str is \n

#### re.sub(pattern,repl,string,count=0,flags=0)

#### re.subn(pattern,repl,string,count=0,flags=0)

#### ...


### 正则表达式对象(regular express object)
#### class re.RegexObject
只有re.compile()函数会产生一个正则表达式对象
> only re.compile() will create a direct regular express object,
it's a special class which design for re.compile().

#### search(string[,pos[,endpos]])

#### match(string[,pos[,endpos]])
#### split(string,maxsplit=0)
#### findall(string[,pos[,endpos]])
#### sub()
#### flags
#### groups
#### groupindex
#### pattern

### 匹配对象(match objects)

#### class re.MatchObject
匹配是否成功
> match objects have a boolean value of True.

```
match = re.search(pattern,string)
if match:
   processs(match)
```

#### group([group1,..])
group的话pattern需要多个()

#### groups([default])
返回一个元组
> return a tuple containing all the subgroups of the match.

``` python
re.match(r"(\d+)\.(\d+)","24.1632")
m.groups()
```
> ('24','1632')

show default
``` python
m = re.match(r"(\d+)\.?(\d+)?", "24")
m.groups()      # Second group defaults to None.
```
> ('24', None)

change default to 0
``` python
m.groups('0')  # Now, the second group defaults to '0'.
('24', '0')
```

### regex examples


