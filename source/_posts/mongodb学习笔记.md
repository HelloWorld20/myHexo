---
title: mongodb笔记
date: 2020-01-17 11:12:52
tags: [mongodb,数据库,学习笔记,后端]
---
只是笔记

<!-- more -->

# 命令

- db 显示当前数据库
- show 一般试show dbs。显示所有数据库（没用数据不显示）
- use 切换数据库

# db命令

跟在`db.`之后的命令

db.dropDatabase();
db.createCollection(name, options)
db.COLLECTION_NAME.drop();

# collection命令

跟在`db.[colection名].`之后的，假设有个叫test的collection

- find
- findOne
- insert 插入一条记录。insert({name: 'wei',age: 22})
- insertMany 插入多条记录。insert([{name: 'wei',age: 22},{name: 'wei',age: 23}])
- save 通过`_id`匹配。如匹配到，则更新，如无匹配，则同`update`
- update 条件更新
- remove 条件删除
- limit 限制条数 
- skip 跳过条数
- sort 排序
- ensureIndex 创建索引，加快检索速度。客户端上看不出区别
- aggregate 聚合

## find

基本

db.test.find({name: 'wei'})

嵌套查询
如有记录如下：

	{
		"_id" : ObjectId("5e12d974519b7839323573b8"),
		"item" : "planner",
		"qty" : 75.0,
		"size" : {
			"h" : 22.85,
			"w" : 30.0,
			"uom" : "cm"
		},
		"status" : "D"
	}

可以嵌套查询

	db.test.find({'size.h: 22.85})

投影，可以指定返回的字段

	db.test.find({name: 'wei'}, {age: 1, _id: 0})	// 只返回age字段，不返回_id、name字段
	

## update

基础更新
会用第二个参数提单匹配到的第一个记录

	db.test.update({name: 'wei'}, {name: 'wang'})

如果只想更新一个字段且不想写其他字段

	db.test.update({ name: 'wei' }, { $set: { name: 'wang'  } }

默认只更新一条记录，如果想更新所有匹配的记录

	db.test.update({ name: 'wei' }, { $set: { name: 'wang'  }, {multi: true} }

## save

通过`_id`字段匹配，如果匹配不到，则相当于命令`insert`，如果匹配到，则替换所有。

# collection命令语法

collection命令可以接收的语法

- $and
- $or
- $set
- 

关于条件 `$and`。以下两个方式相等。第一个相当于简写
```javascript
db.test.find({
	name: 'wei',
	age: 22
})

db.test.find({
	$and: [{ name: 'wei' }, { age: 22 }]
})
```

# aggregate

https://juejin.im/post/5d40482ce51d4561c41fb79e

## 聚合管道

管道操作符
- $group 将collection中的document分组，可用于统计结果
- $match 过滤数据，只输出符合结果的文档
- $project 修改输入文档的结构(例如重命名，增加、删除字段，创建结算结果等)
- $sort 将结果进行排序后输出
- $limit 限制管道输出的结果个数
- $skip 跳过制定数量的结果，并且返回剩下的结果
- $unwind 将数组类型的字段进行拆分

表达式操作符

- $sum
- $avg
- $min
- $max
- $push
- $first
- $last


### $group

语法: ` { $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }`

顾名思义，group就是，分组的意思。以“liuzhou_market"商品详情数据库为例

	db.getCollection('goods').aggregate([
			{
				$group: {
					_id: 'category',
					total: { $sum: '$totalNum' },
					avarage: { $avg: '$totalNum' }
				}
			}
		])

输出的结果是：

	{
		"_id" : "category",
		"total" : 230,
		"avarage" : 10.0
	}

必须传入的参数 `_id`。指定要分组的key。
其他的则是输出key。可以接收的指令如下。

- $sum：计算总和。

- $avg：计算平均值。

- $min：根据分组，获取集合中所有文档对应值得最小值。

- $max：根据分组，获取集合中所有文档对应值得最大值。

- $push：将指定的表达式的值添加到一个数组中。

- $addToSet：将表达式的值添加到一个集合中（无重复值）。

- $first：返回每组第一个文档，如果有排序，按照排序，如果没有按照默认的存储的顺序的第一个文档。

- $last：返回每组最后一个文档，如果有排序，按照排序，如果没有按照默认的存储的顺序的最后个文档。

### $project

类似`find()`指令的第二个参数，用于指定哪个字段显示、哪个不显示

## mapReducer

## 单一的聚合命令

MongoDB还提供了，db.collection.estimatedDocumentCount（），db.collection.count（）和db.collection.distinct（） 所有这些单一的聚合命令。可以提高效率

### distinct

	db.test.distict("name")

输出

	["wei", "wang"]

会从所有记录中，返回记录的`name`字段，去重后，组合在数组里返回。

### count

顾名思义。返回统计数据。