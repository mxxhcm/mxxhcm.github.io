---
title: MongoDB笔记
date: 2019-03-18 15:06:56
tags: 
 - 数据库
categories: 数据库
mathjax: false
---

## 数据库的安装

自行下载安装包并安装

## 数据库的运行和连接,以及以下简单的使用

### 一些简单的操作
``` shell
# 连接mongodb数据库
mongo (database_name) # 如果不输入数据库名会默认连接到mongodb自带的一个数据库test，如果指定了数据库名就会连接到该数据库

#切换数据库
use database_name;

#查看所有的数据库
show databases;

#查看所有的collection
show collections;
```

### 使用python代码中连接到数据库
``` python
import pymongo

connection = pymongo.MongoClient('localhost', 27017)

db = connection.test  # db指向test数据库
collection = db.mycollection

items = collection.find()
print(items['key'])
```

### C++ 访问mongodb
安装参考[1]。


## CRUD操作
### id的构成 12 bytes hex
4+3+2+3
timestamp + mac address + pid + counter
timestamp是unix timestamp，mac address 是 mongd运行的网卡mac address，pid是process id，

### create document

#### create one document(insertOne)
``` mongodb
db.collection_one.insertOne({"key_one":"value","key_two":"value"})
```

#### create many documents（有order,insertMany）
``` mongodb
db.collection_one.insertMany(
[
{"key_one":"value","key_two":"value"},
{"key_one":"value","key_two":"value"},
{"key_one":"value","key_two":"value"}
])
```

#### create many documents（无order,insertMany）
``` mongodb
db.collection_one.insertMany(
db.collection_one.insertMany(
[
{"key_one":"value","key_two":"value"},
{"key_one":"value","key_two":"value"},
{"key_one":"value","key_two":"value"}
] , {"ordered":false})
```

#### upsert
第一个参数是一个filter选择合适的 document，第二个参数是一个更新操作for the documents were selected，第三个参数是 that if there is no matching result,if the value of upsert is true,then insert a new document,else do nothing.
``` mongodb
db.collection_one.insertMany(
db.movieDetails.updateOne({ name:"mxxhcm"}, { \$set:{lover:"mahuihui"} } , {upsert : true})
```

#### 有无order的区别
有order的话遇到inset错误就会停下来，没有order的话在插入document的时候，遇到错误会跳过该条语句执行下一条语句。

### read documents(query documents)
link:
https://docs.mongodb.com/manual/reference/operator/query/

#### 查找document
查找collection_one这个collection中所有的document
``` monogodb
db.collection_one.find()                
```

查找collection_one这个collection中满足{}中条件的collection，{}中的条件需要满足anded
```
db.collection_one.find({})
```

pretty()表示以规范的格式展现出来查询结果
``` mongodb
db.collection_one.find().pretty()
```

findOne表示只展示出第一条结果
``` mongodb
db.collection_one.findOne()
```
满足{}中条件的第一条结果
``` mongodb
db.collection_one.findOne({})     
```

#### 对document进行计数
``` mongodb
db.collection_one.count()
```

#### 设置查找的条件(equality match)
##### a.scalar equality match
``` mongodb
db.collection_one.find({"key":"value","key","value"})
```

##### b.nested documents equality match
``` mongodb
db.collection_one.find({"key.key2.key3":"value"})
```

##### c.equality matches on arrays
###### entire array value match
``` mongodb
db.collection_one.find({key:[value1,value2]})
```

###### any array element fileds match a specfic value
``` mongodb
db.collection_one.find({key:"value2"})
```

###### a specfiec element fields match a specfic value
``` mongodb
db.collection_one.find({key.0:"value1"})
```

#### （4）cursor

#### （5）projection
by default,mongodb return all fields in all matching documents for query.
Projection are supplied as the second argument
db.collection_one.find({"key1":"value","key2","value"},{"key1":1,"key2":1,"key3":0,"key4":0}).pretty()

#### （6）comparison operation
\$eq
\$gt
\$gte
\$lt
\$lte
\$ne
\$in
\$nin
##### a.在某个范围内
``` mongodb
db.movieDetails.find({ runtime : { \$gt: 70,  \$lte:100 } }).pretty()
```

##### b.不等于(\$ne)
``` mongodb
db.movieDetails.find({ rated : { \$ne:"unrated" } }).pretty()
```

##### c.在(\$in)
``` mongodb
db.movieDetails.find({rated : { \$in : ["G","PG","PG-13"] }  }).pretty()
```

#### （7）element operator
##### a.存在某个filed(\$exists)
``` mongodb
db.movieDetail.find( { filed_name : { \$exists: true|false } } ).pretty()
```

##### b.某个字段的类型(\$type)
``` mongodb
db.movieDetail.find( { filed_name : { \$type :"string"} }).pretty()
```

#### （8）logical operator
\$or 
\$and 
\$not 
\$nor 
##### a.逻辑或(\$or) 
\$or需要数组作为参数
``` mongodb
db.movieDetails.find( { \$or: [ { field_one : {\$type : "string"} } , {field_two : {\$exist: "name" } } ] } ).pretty()
```

##### b.逻辑与(\$and)
\$and操作支持我们在同一个filed指定多个约束条件
``` mongodb
db.movieDetails.find({ \$and: [ {field_one: {\$ne :null} } , { field_one: {\$gt:60, \$lte: 100} } ] }).pretty()
```

#### （9）regex operator
``` mongodb
db.movieDetails.find({ "awards.text": { \$regex: /^Won\s/}  }).pretty()
```

#### （10）array operator
\$all
\$size
\$elementMatch
##### a.all
``` mongodb
db.movieDetails.find({genres : {\$all :["comedy","crime","drama"]} }).pretty()
db.movieDetails.find({genres :  ["comedy","crime","drama"]  }).pretty()
```
上面两个式子是有区别的，第一个式子会匹配genres中包含"comedy","crime","drama"的document
而第二个只会匹配genres为"comedy","crime","drama"的document。
##### b.size
``` mongodb
db.movieDetails.find({country : {\$size : 3} }).pretty()
```

##### c.elementMatch
``` mongodb
db.movieDetails.find({ boxOffice: { country: "UK", revenue: { \$gt: 15 } } })
```

### 9.update documents
link:
https://docs.mongodb.com/manual/reference/operator/update/

#### （0）some update operator
``` mongodb
db.movieDetails.updateOne( { name : "mxxhcm" } , { \$inc : { age: 1} })
```

#### （1）updateOne
##### a.update for scalar fields
\$set
``` mongodb
db.movieDetails.updateOne( { name : "mxxhcm" } , { \$set : { age: 19} })
```
\$unset
``` mongodb
db.movieDetails.updateOne( { name : "mxxhcm" } , { \$unset : { age: 19} })
```
\$inc
age后是在原来的age上加的数值
``` mongodb
db.movieDetails.updateOne( { name : "mxxhcm" } , { \$set : { age: 19} })
```
updateOne has two arguments, the first one is a selector,the second argument is how we want to update the document.
##### b.update for array fields
\$push
``` mongodb
db.movieDetails.updateOne({name:"mxxhcm"} , {\$push: { reviews: { key1:value,key2:value...}  }  } )
db.movieDetails.updateOne({name:"mxxhcm"} , {\$push: { reviews:
                                                                                                { \$each: [{ key1:value,key2:value...} ,                                                                                                                        {key1:value,key2:value...} ]  }  
                                                                                              }   } )
db.movieDetails.updateOne({name:"mxxhcm"} , {\$push: { reviews:
                                                                                                { \$each: [{ key1:value,key2:value...} ,                                                                                                                        {key1:value,key2:value...} ] ,                                                                                                             \$slice:3 }  
                                                                                              }   } )
db.movieDetails.updateOne({name:"mxxhcm"} , {\$push: { reviews:
                                                                                                { \$each: [{ key1:value,key2:value...} ,                                                                                                                        {key1:value,key2:value...} ] ,                                                                                                             \$position:0,  
                                                                                                  \$slice:3 }  
                                                                                              }   } )
```

#### （2）updateMany
the same as updateOne

#### （3）replaceOne
``` mongodb
db.movieDetail.replcaeOne({},{})
```
the first argument is a filter,the second argument is the thing that replace what the filter choose,it can be a document,or a variable point to a document.

### 10. using mongdb by pymongo
见代码
#### （1）sort，skip，limit
sort > skip > limit
``` mongodb
cursor.sort('student_id',pymongo.ASCENDING).skip(4).limit(3)
in python file:
cursor.sort(  [ ('student_id',pymongo.ASCENDING) , ('score',pymongo.DESCENDING) ] ).skip(4).limit(3)
in mongo shell:
cursor.sort(  [ {'student_id':1}, {'score',-1)} ] ).skip(4).limit(3)
```
####（2）find,find_one,cursors
####（3）project
####（4）regex
####（5）insert
####（6）update
####（7）
There is a intervening between find and update,so maybe you find and update is not the same one.


## 参考文献
1.http://mongocxx.org/mongocxx-v3/installation/
