---
title: python time
date: 2019-04-13 14:52:30
tags:
 - python
categories: python
---

## time（time library,datetime library,panda.Timestamp()）
import time
### 获得当前时间
``` python
time.time()        #获得当前timestamp
```

### time.localtime(timestamp)
得到一个struct_time
time.struct_time(tm_year=2018....)

### 将struct time转换成string
> Convert a tuple or struct_time representing a time as returned by gmtime() or localtime() to a string as specified by the format argument.If t is not provided,the current time as returned by localtime() is used.

```python
import time
time.strftime(format,t)        #将一个struct_time表示为一个格式化字符串
time.strftime("%Y-%m-%d %H:%M:%S",time.localtime())
```

### 将一个string类型的事件转换成struct time
> Parse a string representing a time accroding to a format.The return value is a struct_time as returned by gmtime() or localtime()

```python
import time
time.strptime("a string representing a time","a format")    #将某个format表示的time转化为一个struct_time()
time.strptime("2014-02-01 00:00:00","%Y-%m-%d %H:%M:%S")
```

### time.mktime()
将时间t转换成timestamp


