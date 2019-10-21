---
title: pandas笔记
date: 2019-03-18 15:15:54
tags:
 - pandas
 - python
categories: python
---

## pd.read_***()
### pd.read_csv()
``` python
import pandas
pandas.read_csv(filepath_or_buffer, sep=', ', delimiter=None, header='infer', names=None, index_col=None, usecols=None, squeeze=False, prefix=None, mangle_dupe_cols=True, dtype=None, engine=None, converters=None, true_values=None, false_values=None, skipinitialspace=False, skiprows=None, nrows=None, na_values=None, keep_default_na=True, na_filter=True, verbose=False, skip_blank_lines=True, parse_dates=False, infer_datetime_format=False, keep_date_col=False, date_parser=None, dayfirst=False, iterator=False, chunksize=None, compression='infer', thousands=None, decimal=b'.', lineterminator=None, quotechar='"', quoting=0, escapechar=None, comment=None, encoding=None, dialect=None, tupleize_cols=None, error_bad_lines=True, warn_bad_lines=True, skipfooter=0, skip_footer=0, doublequote=True, delim_whitespace=False, as_recarray=None, compact_ints=None, use_unsigned=None, low_memory=True, buffer_lines=None, memory_map=False, float_precision=None)
```

filepath_or_buffer: 文件路径，或者一个字符串，url等等
sep: str,分隔符，默认是','
delimiter: str,定界符，如果指定该参数，sep参数失效
delimiter_whitespace: boolean,指定是否吧空格作为分界符如果指定该参数，则delimiter失效
header: int or list of ints,指定列名字，默认是header=0,表示把第一行当做列名，如果header=[0,3,4],表示吧第0,3,4行都当做列名，真正的数据从第二行开始，如果没有列名，指定header=None
index_col: int or sequence or False,指定哪几列作为index，index_col=[0,1],表示用前两列的值作为一个index，去访问后面几列的值。
prefix: str,如果header为None的话，可以指定列名。
parse_dates: boolean or list of ints or names,or list of lists, or dict 如果是True，解析index，如果是list of ints，把每一个int代表的列都分别当做一个日期解析，如果是list of lists，将list中的list作为一个日期解析，如果是字典的话，将dict中key作为一个新的列名，value为这个新的列的值。
keep_date_col: boolean,如果parser_dates中是将多个列合并为一个日期的话，是否保留原始列
date_parser: function,用来解析parse_dates中给出的日期列，是自己写的函数，函数参数个数和一个日期的列数相同。

chunksize: 如果文件太大的话，分块读入
``` python
data = pd.read_csv("input.csv",chunksize=1000)
for  i  in  data:
  ...
```

## DataFrame
### 声明一个DataFrame
data = pandas.DataFrame(numpy.arange(16).reshape(4,4),index=list('abcd'),columns=('wxyz')
    w  x  y  z
a  0  1  2  3
b  4  5  6  7
c  8  9  10  11
d  12  13  14  15
index 是index列的值
columns 是列名


### 访问某一列
``` python
data = pandas.DataFrame(numpy.arange(16).reshape(4,4),index=list('abcd'),columns=('wxyz')
data['w']
data.w
```

### 写入某一列
只能先访问列 再访问行
data['w'] = []   # =左右两边shape必须一样
data['w'][0]  #某一列的第0行

### groupby
``` python
data = pandas.DataFrame(np.arange(16).reshape(4,4),index=list('abcd'),columns=('wxyz'))
for key,value in data.groupby("w"):  # group by 列名什么的，就是说某一列的值一样分一组
  value = value.values  # value是一个numpy数组
  value_list = value.tolist()  #将numpy数组转换为一个list
  for single_list in value_list:
     single_list = str(single_list)
     ...
```
