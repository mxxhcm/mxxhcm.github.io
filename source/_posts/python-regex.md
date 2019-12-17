---
title: python regex
date: 2019-04-13 14:50:41
tags:
 - python
categories: python
---

## regex examples
``` python
import regex as re

# 找出每一行的数字
string = """a9apple1234
2banana5678
a3coconut9012"""
pattern = "[0-9]+"

# search
result = re.search(pattern, string)
print(type(result))
print(result[0])
print(result.group(0))

# match
# 即使设置了MULTILINE模式，也只会匹配string的开头而不是每一行的开头
result = re.match(pattern, string, re.S| re.M)  
print(type(result))
# print(result[0])
# print(result.group(0))

# findall
result = re.findall(pattern, string)
print(type(result))
print(result)
```

## 语法
.   匹配除了newline的任意character，如果要匹配newline，需要添加re.DOTALL flag
\*  重复至少$0$次 
\+  重复至少$1$次
?   重复$0$次或者$1$次 
{}  重复多少次，如a{3,5}表示重复$3-5$次
[]  匹配方括号内的内容,如[1-9]表示匹配$1-9$中任意一个
^   matching the start of the string
$   matching the end os the string
+,\*.?    都是贪婪匹配，如果加一个?为非贪婪匹配
+?,\*?,??    为非贪婪匹配
()  匹配括号内的正则表达式，表示一个group的开始和结束
|   或
\number
\b  匹配empty string
\B   
\d  匹配数字
\D  匹配非数字
\s  匹配空白符[ \t\n\r\f\v]
\S  匹配非空白符
\w  匹配unicode
\W
\A
\Z

## 模块
- re.compile(patern, flags=0)
- re.match(pattern, string, flags=0)
- re.fullmatch(pattern,string,flags=0)
- re.search(pattern, string, flags=0)
- re.split(pattern, string, maxflit=0, flags=0)
- re.findall(pattern,string,flags=0)
- re.sub(pattern,repl,string,count=0,flags=0)
- re.subn(pattern,repl,string,count=0,flags=0)

### flags
> flags can be re.DEBUG, re.I, re.IGNORECASE, re.L, re.LOCALE, re.M, re.MULTILINE, re.S, re.DOTALL, re.U, re.UNICODE, re.X, re.VERBOSE

- re.I(re.IGNORECASE) 忽略大小写
- re.L(re.LOCALE) 
- re.M(re.MULTILINE) 多行模式，设置以后.匹配newline。指定re.S时，'^'匹配string的开始和each line的开始(紧跟着each newline); '$'匹配string的结束和each line的结束($在newline之前，immediately preceding each newline)。如果不指定的话, '^'只匹配string的开始,'$'只匹配string的结束和immediately before the newline (if any) at the end of the string，对应inline flag (?m).
- re.S(re.DOTALL)  
- re.U(re.UNICODE)
- re.X(re.VERBOSE)
- re.DEBUG

``` python
import regex as re

print(re.I)
print(re.IGNORECASE)
print(re.L)
print(re.LOCALE)
print(re.M)
print(re.MULTILINE)
print(re.S)
print(re.DOTALL)
print(re.U)
print(re.UNICODE)
print(re.X)
print(re.VERBOSE)
print(re.DEBUG)

print(re.M is re.MULTILINE)
print(re.I is re.IGNORECASE)
```

re.M例子
``` python
text = """First line.
Second line.
Third line."""

pattern = "^.*$"  # 匹配从开始到结束的任何字符
# 默认情况下， . 不匹配newlines，所以默认情况下不会有任何匹配结果，因为$之前有newline，而.不能匹配
# re.search(pattern, text) is None  # Nothing matches!
print(re.search(pattern, text))

# 如果设置MULTILINE模式, $匹配每一行的结尾，这个时候第一行就满足要求了，设置MULTILINE模式后，$匹配string的结尾和每一行的结尾（each newline之前)
print(re.search(pattern, text, re.M).group())
# First line.

# 如果同时设置MULTILINE和DOTALL模式, .能够匹配newlines，所以第一行和第二行的newline都匹配了，在贪婪模式下，就匹配了整个字符串。
print(re.search(pattern, text, re.M | re.S).group())
# First line.
# Second line.
# Third line.
```
### re.compile(patern, flags=0)
将一个正则表达式语句编译成一个正则表达式对象，可以调用正则表达式的match()和search()函数进行matching。
> complie a regular expression pattern into a regular expression object,which can be used for matching using its match() and search()

``` python
str = "https://abc https://dcdf https://httpfn https://hello"
import regex as re

prog = re.compile(pattern)
results = prog.match(string)
# 上面两行等价于下面一行

results = re.match(pattern, string)
```

### re.match(pattern, string, flags=0) or re.fullmatch(pattern,string,flags=0)
在给定的string开始位置进行查找，返回一个match object。**即使设置了MULTILINE mode, re.match()也只会在string的开始而不是each line的每一行开始匹配。**

### re.search(pattern, string, flags=0)
在给定的string任意位置进行查找，返回一个match object。
> locat a match anywhere in string

### search() vs. match()
re.macth()在string的开头查找，而re.search在string的任意位置查找，他们都返回match object对象。如果不匹配，返回None。
```
import re

match1 = re.match("cd", "abcdef")     # match
match2 = re.search("cd", "abcdef")    # search
print(match1)
print(match2)
print(match2.group(0))
# None
# <regex.Match object; span=(2, 4), match='cd'>
# cd

with open("content.txt", "r") as f:
    s = f.read()
match3 = re.match("cd", s)     # match
match4 = re.search("cd", s)
print(match3)
print(match4)
# None
# <regex.Match object; span=(4, 6), match='cd'>
```

### re.findall(pattern,string,flags=0)
查找字符string所有匹配pattern的字符
``` python
import regex as re
str = "https://abc https://dcdf https://httpfn https://hello"
p2 = "https.+? "    # pay attention to space here
results = re.findall(p2,str)
```
> ['https://abc ', 'https://dcdf ', 'https://httpfn ']    # pay attention to the last ,since the end of str is \n

### re.split(pattern, string, maxflit=0, flags=0)
按照patten对string进行分割
``` python
import regex as re
str = "https://abc https://dcdf https://httpfn https://hello"
p1 = " "
results = re.split(p1,str)
```
> ['https://abc', 'https://dcdf', 'https://httpfn', 'https://hello']

### re.sub(pattern,repl,string,count=0,flags=0)
### re.subn(pattern,repl,string,count=0,flags=0)
### ...


## 正则表达式对象(regular express object)
class re.RegexObject
只有re.compile()函数会产生正则表达式对象，正则
> only re.compile() will create a direct regular express object,
it's a special class which design for re.compile().
正则表达式对象支持下列方法和属性
- match(string[,pos[,endpos]])
- search(string[,pos[,endpos]])
- findall(string[,pos[,endpos]])
- split(string,maxsplit=0)
- sub()
- flags
- groups
- groupindex
- pattern

### match(string[,pos[,endpos]])
### search(string[,pos[,endpos]])
### findall(string[,pos[,endpos]])
### split(string,maxsplit=0)
### sub()
### flags
### groups
### groupindex
### pattern


## 匹配对象(match objects)
class re.MatchObject
匹配是否成功
> match objects have a boolean value of True.

```
match = re.search(pattern, string)
if match:
   processs(match)
```

MatchObject支持以下方法和属性
- group([group1,..])
- groups([default=None])
- groupdict(default=None)
- start([group])
- end([group])
- span([group])
- pos
- endpos
- lstindex
- lastgroup
- re
- string

### group([group1,..])
group的话pattern需要多个()

### groups([default])
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

## 参考文献
1.https://stackoverflow.com/questions/180986/what-is-the-difference-between-re-search-and-re-match
2.https://devdocs.io/python~3.7/library/re
3.https://mail.python.org/pipermail/python-list/2014-July/674576.html
