title: mongodb一些简单操作笔记
author: Tany
tags:
  - mongo
categories:
  - mongo
date: 2021-05-22 17:28:00
---
学习 mongo 操作的简单笔记，记不住了就来看看。

<!-- more -->


### 插入文档

#### MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：

```
db.COLLECTION_NAME.insert(document)
或
db.COLLECTION_NAME.save(document)
```

#### 区别

- save()：如果 _id 主键存在则更新数据，如果不存在就插入数据。该方法新版本中已废弃，可以使用 db.collection.insertOne() 或 db.collection.replaceOne() 来代替。
- insert(): 若插入的数据主键已经存在，则会抛 org.springframework.dao.DuplicateKeyException 异常，提示主键重复，不保存当前数据。

##### db.collection.insertOne() 和 db.collection.insertMany()

- 语法：

```
单个:
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)
```

```
一个或多个:
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

- 参数说明：

1. document：要写入的文档。
2. writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
3. ordered：指定是否按顺序写入，默认 true，按顺序写入。



### 查询文档

#### db.collection.find(query, projection)

- query ：可选，使用查询操作符指定查询条件
- projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

##### 字段样例:

| _id                      | v     | c | abv                |
| ------------------------ | ----- | - | ------------------ |
| 5efed2b5282700007500040a | adsd  |   | (N/A)              |
| 5efed4dd2827000075000433 | (N/A) | c | (Array) 4 Elements |
| 5efed4dd2827000075000414 | (N/A) | c | (Array) 4 Elements |

- 查询样例:

```
db.test2.find({v:"adsd",abv: null})

// db.test2.findOne({v:"adsd",abv: null})  // 只返回第一个
```

- 返回样例:

| _id                      | v    | c |
| ------------------------ | ---- | - |
| 5efed2b5282700007500040a | adsd |   |

- ***结论***

1. null 字段不返回
2. 空 字段返回 空

#### 字段比较

| 操作       | 格式                                                                    | 范例                                    | RDBMS中的类似语句     |
| ---------- | ----------------------------------------------------------------------- | --------------------------------------- | --------------------- |
| 等于       | {`<key>`:`<value>`}                                                 | db.col.find({"by":"菜鸟教程"}).pretty() | where by = '菜鸟教程' |
| 小于       | {`<key>`:{$lt:<value>}}|	db.col.find({"likes":{$lt:50}}).pretty()   | where likes < 50                        |                       |
| 小于或等于 | {`<key>`:{$lte:<value>}}|	db.col.find({"likes":{$lte:50}}).pretty() | where likes <= 50                       |                       |
| 大于       | {`<key>`:{$gt:<value>}}|	db.col.find({"likes":{$gt:50}}).pretty()   | where likes > 50                        |                       |
| 大于或等于 | {`<key>`:{$gte:<value>}}|	db.col.find({"likes":{$gte:50}}).pretty() | where likes >= 50                       |                       |
| 不等于     | {`<key>`:{$ne:<value>}}|	db.col.find({"likes":{$ne:50}}).pretty()   | where likes != 50                       |                       |

#### and  or

- ***and***

```
db.col.find({key1:value1, key2:value2}).pretty()
```

- ***or*** 查询键 by 值为 菜鸟教程 或键 title 值为 MongoDB 教程 的文档。

```
db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```

- ***联合使用***  类似常规 SQL 语句为： 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'

```
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```

#### 分页处理

- 类似mysql的 limit offset
- 使用 limit(输出文档的行数) skip(跳过的文档行数)

#### 排序

- sort({KEY:1}) 升序
- sort({KEY:-1}) 降序

#### 索引

- 语法 : db.collection.createIndex(keys, options)
- 说明 : Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。
- 实例：
  `db.col.createIndex({"title":1})`
- ***多个字段创建索引(复合索引)***
- 实例：
  `db.col.createIndex({"title":1,"description":-1})`
- 可选参数列表:

| 参数               | 类型          | 描述                                                                                                                                       |
| ------------------ | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| background         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false。            |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为false.                                                                                 |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。                                                          |
| dropDups           | Boolean       | 3.0+版本已废弃。在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false.                                                   |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。                                                                               |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。                                                                             |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。                                                              |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语                                                                      |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language.                                                 |

- 后台创建索引实例
  `db.values.createIndex({open: 1, close: 1}, {background: true})`
  
### 更新文档

#### update() 和 save()

- update()语法：

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

- 参数说明：

1. query : update的查询条件，类似sql update查询内where后面的。
2. update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
3. upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
4. multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
5. writeConcern :可选，抛出异常的级别。

- 更新样例

```
db.test2.update(
{v:"vvv"},  // 条件 v = "vvv"
{$set:{v:"aaa",abv:"asd"}}, //set v="aaa" and abv= "asd"
    {
    upsert: true,  //不存在就添加
    multi:true      //更新多条
    }
)
```

- 省略 upsert ,multi ...

```
db.test2.update(
{v:"aaa"},
{$set:{v:"azxxc"}},

true,
true

)
```

#### sava() 通过传入的文档来替换已有文档，_id 主键存在就更新，不存在就插入。语法格式如下：

```
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

- 参数说明：

1. document : 文档数据。
2. writeConcern :可选，抛出异常的级别。

- 样例:
- 带 ObjectId

```
db.test2.save(
{"_id":ObjectId("5efee712e5ca213e2735928c"),
 v :"qwerr"
}
)
```

- 不带 ObjectId

```
db.test2.save(
{"_id":"5efee712e5ca213e2735928c",
 v :"qwerr"
}
)
```

- ***ObjectId*** 必填，否则第一次save会变成添加，第二次sava才能识别


#### 更多实例:

```
只更新第一条记录：
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
```

```
全部更新：
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
```

```
只添加第一条：
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
```

```
全部添加进去:
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
```

```
全部更新：
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
```

```
只更新第一条记录：
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );

```

### 删除文档

#### remove() 方法的基本语法格式如下所示：

- 2.6版本之前:

```
db.collection.remove(
   <query>,
   <justOne>
)
```

- 2.6 版本之后:

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

- 参数说明：

1. query :（可选）删除的文档的条件。
2. justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
3. writeConcern :（可选）抛出异常的级别。

- 操作样例:

```
db.test2.remove(
{v:"a"}
false
)
```

- 返回结果:

```
WriteResult({ "nRemoved" : 2, "writeConcernError" : [ ] })
```

#### remove() 方法已经过时了，现在官方推荐使用 deleteOne() 和 deleteMany() 方法

- 如删除集合下全部文档：
  `db.inventory.deleteMany({})`
- 删除 status 等于 A 的全部文档：
  `db.inventory.deleteMany({ status : "A" })`
- 删除 status 等于 D 的一个文档：
  `db.inventory.deleteOne( { status: "D" } )`

#### 知识点

- remove() 方法 并不会真正释放空间。需要继续执行 db.repairDatabase() 来回收磁盘空间。

```
db.repairDatabase()
或者
db.runCommand({ repairDatabase: 1 })
```