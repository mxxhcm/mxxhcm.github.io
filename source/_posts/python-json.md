---
title: python json
date: 2019-06-06 15:58:02
tags:
- python
categories: python
---

## 什么是json
是一种文件格式
json object和python的字典差不多。
json在python中可以以字符串形式读入。

## python读取json

### json.loads()
json.loads()从内存中读取。
```
import json

person = '{"name": "Bob", "languages": ["English", "Fench"]}'
person_dict = json.loads(person)

# Output: {'name': 'Bob', 'languages': ['English', 'Fench']}
print( person_dict)

# Output: ['English', 'French']
print(person_dict['languages'])
```

### json.load()
json.load()从文件对象中读取。
假设有名为person.json的json文件
``` text
{
"name": "Bob", 
"languages": ["English", "Fench"]
}
```
直接从文件对象中读取
``` python
import json

with open('person.json') as f:
  data = json.load(f)

# Output: {'name': 'Bob', 'languages': ['English', 'Fench']}
print(data)
```

## python写json文件
### json.dumps()
json.dumps()将字典转化为JSON字符串
``` python

import json

person_dict = {'name': 'Bob',
'age': 12,
'children': None
}
person_json = json.dumps(person_dict)

print(person_json)
```

### json.dump()
json.dump()直接将字典写入文件对象
``` python
import json

person_dict = {"name": "Bob",
"languages": ["English", "Fench"],
"married": True,
"age": 32
}

with open('person.txt', 'w') as json_file:
  json.dump(person_dict, json_file)
```

## pretty JSON
``` python
import json

person_string = '{"name": "Bob", "languages": "English", "numbers": [2, 1.6, null]}'

# Getting dictionary
person_dict = json.loads(person_string)

# Pretty Printing JSON string back
print(json.dumps(person_dict, indent = 4, sort_keys=True))
```

## 参考文献
1.https://www.programiz.com/python-programming/json
