# MyMongoDBRoadOfGrowth

## 第一部分：MongoDB系统结构
### 1.1 NoSQL和MongoDB
### 1.2 MongoDB体系结构
### 1.3 MongoDB和RDBMS（关系型数据库）对比
|RDBMS|MongoDB|
|:---:|:---:|
|database|database|
|table|collection|
|row|document|
|column|filed|
|index|index|
|join|embedded document|
|primary key|primary key|
### 1.4 什么是BSON
BSON有三个特点：轻量性、可遍历性、高效性
### 1.5 BSON在MongoDB中的使用

## 第二部分：MongoDB命令
### 2.1 MongoDB的基本操作
```
查看数据库：
   1. show dbs;
   2. show databases;

切换数据库：
   use db_name;
   
创建集合：
   db.createCollection("collection");

查看集合：
   1. show tables;
   1. show colletions;
   
删除集合：
   db.collection.drop();
   
删除数据库：
   db.dropDatabase();
```

### 2.2 MongoDB集合数据操作（CRUD）
#### 2.2.1 数据添加
```
插入一条数据：
    1. db.collection.insert(doc);
    2. db.collection.insertOne(doc);

插入多条数据：
    1. db.collection.insert([doc1,doc2,...,docN]);
    2. db.collection.insertMany([doc1,doc2,...,docN]);
```

#### 2.2.2 数据查询

+ 比较查询

   db.collection.find({filter})
  
   |操作|条件格式|
   |:---:|:---:|
   |等于|{key:value}|
   |等于|{key:{$eq:value}}|
   |大于|{key:{$gt:value}}|
   |小于|{key:{$lt:value}}|
   |大于等于|{key:{$gte:value}}|
   |小于等于|{key:{$lte:value}}|
   |不等于|{key:{$ne:value}}|

+ 逻辑查询

   db.collection.find({filter})

   |操作|条件格式|
   |:---:|:---:|
   |and|{key1:value1,key2:value2,...,keyN:valueN}|
   |and|{$and:[{key1:value1},{key2:value2},...,{keyN:valueN}]}|
   |or|{$or:[{key1:value1},{key2:value2},...,{keyN:valueN}]}|
   |not|{key:{$not:{$比较操作:value}}}|

+ 分页查询
  ```
  db.collection.find({filter}).sort({sort_field:排序方式}).skip(lines).limit(show)
  
  filter：查询条件
  sort_field：排序字段
  排序方式：1正序，-1倒序
  lines：跳过行数
  limit：显示行数
  ```

#### 2.2.3 数据更新 调用UPDATE
+ 设置字段值：$set
+ 删除字段值：$unset
+ 对修改的值进行自增：$inc
注：如果不写更新操作符会导致数据丢失

db.collection.update(<query>, <update>, {
    upsert: <boolean>,
    multi: <boolean>,
    writeConcern: <document>
})

query: update的查询条件，类似sql update中的where
update：update的对象和一些更新的操作符，类似sql update中的set
upsert：可选，如果不存在update的记录是否插入，默认：false不插入
multi：可选，是否更新多条记录，默认：false只更新查询出来的第一条
writeConcern：可选，用来指定mongod对写操作的回执行为比如写的行为是否需要确认。

#### 2.2.4 数据删除
db.collection.remove(<query>,{
    justOne: <boolean>
    writeConcern: <document>
})

query: remove的查询条件，类似sql delete中的where
justOne：是否只删除一个document，默认：false删除全部
writeConcern：可选，用来指定mongod对写操作的回执行为比如写的行为是否需要确认。

### 2.3 MongoDB 聚合操作
#### 2.3.1 聚合操作简介
#### 2.3.2 MongoDB聚合操作分类
+ 单目聚合操作
+ 聚合管道
+ MapReduce 编程模型

#### 2.3.3 单目聚合操作
+ count：统计数据条数
+ distinct：数据去重

#### 2.3.4 聚合管道
db.collection.aggregate([aggregate_operation1,aggregate_operation2,...,aggregate_operationN])

+ 聚合操作

   |表达式|描述|
   |:---:|:---|
   |$sum|计算总和|
   |$avg|计算平均值|
   |$min|获取集合中所有文档对应值的最小值|
   |$max|获取集合中所有文档对应值的最大值|
   |$push|在结果文档中插入值到一个数组中|
   |$addToSet|在结果文档中插入值到一个数组中，但数据不重复|
   |$first|根据资源文档的排序获取第一个文档数据|
   |$last|根据资源文档的排序获取最后一个文档数据|

+ 管道操作
 
   |表达式|描述|
   |:---:|:---|
   |$group|将集合中的文档分组，可用于统计结果|
   |$project|修改输入文档的结构，可以用来重命名、增加或删除，也可以用于创建计算结果以及嵌套文档|
   |$match|用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作|
   |$limit|用来限制MongoDB聚合管道返回的文档数|
   |$skip|在聚合管道中跳过指定数量的文档，并返回余下的文档|
   |$sort|将输入文档排序后输出|
   |$geoNear|输出接近某一地理位置的有序文档|

#### 2.3.5 MapReduce编程模型
```
db.collection.mapReduce(
    function(){}, // map函数  
    function(){},   // reduce函数 
    {
        out:collection,
        query:document,
        sort:document,
        limit:number
        finalize:<function>,
        verbose:<boolean>
    }
)
```

|参数|说明|
|:---:|:---|
|map|JavaScript函数，负责每个输入文档转换为零个或多个文档，生成键值对序列，作为reduce函数参数|
|reduce|JavaScript函数，对map操作的输出做合并操作|
|out|统计结果存放集合|
|query|筛选条件，满足条件的数据会进入map函数|
|sort|发往map之前排序，和limit结合的sort排序参数，可以优化分组机制|
|limit|发往map函数中document的数量|
|finalize|可以对reduce输出结果再一次修改|
|verbose|结果信息中是否包括时间信息，默认：false|

## 第三部分：MongoDB索引Index
### 3.1 什么是索引
### 3.2 索引类型
#### 3.2.1 单键索引（Single Field）
+ 普通单键索引：db.collection.createIndex({字段:排序顺序})
+ 过期单键索引：db.collection.createIndex({日期字段:排序顺序}, {expireAfterSeconds: seconds})

```
注意：
   1. 排序顺序，1正序，-1倒序
   2. 单键索引可以忽略排序顺序
   3. 过期索引只能在单键索引上建立，且字段必须为日期类型，到期之后数据会自动删除
```

#### 3.2.2 复合索引（Compound Index）
db.collection.createIndex({字段1:排序顺序,字段2:排序顺序,...,字段N:排序顺序})
```
注意：
   1. 复合索引需要注意字段顺序和排序顺序，否则容易造成索引失效
   2. 复合索引使用最左原则，不满足最左原则的也会造成索引失效
```

#### 3.2.3 多键索引（Multikey indexes）
多键索引是针对包含数组类型的情况，MongoDB支持针对数组中每一个element创建索引，
Multikey indexes支持strings、numbers和nested documents

#### 3.2.4 地理空间索引（Geospatial Index）
针对地理空间坐标数据创建索引
+ 2dsphere索引，用于存储和查找球面上的点
+ 2d索引，用于存储和查找平面上的点

#### 3.2.5 全文索引
db.collection.createIndex({"字段":"text"})
db.collection.find({"$text":{"$search":"coffee"}})

#### 3.2.6 哈希索引（Hash Index）
db.collection.createIndex({"字段":"hashed"})
Hash索引只支持等于查询


### 3.3 索引和explain分析
#### 3.3.1 索引管理
+ 创建索引并在后台执行
   db.collection.createIndex({"字段":"排序方式",{background:true}})

+ 获取某个collection上的全部索引
   db.collection.getIndexes()
  
+ 索引的大小
   db.collection.totalIndexSize()
  
+ 索引的重建
   db.collection.reIndex()
  
+ 索引的删除
   db.collection.dropIndex("index_name")
   db.collection.dropIndexes()
   注意：_id对应的索引是删除不了的

#### 3.3.2 explain
explain接受的参数
|queryPlanner|默认参数，具体执行计划信息参考下面的表格|
|executionStats|会返回执行计划的一些统计信息|
|allPlansExecution|用来获取所有的执行计划，基本与executionStats相同|

##### queryPlanner默认参数
|参数|含义|
|:---:|:---|
|plannerVersion|查询计划版本|
|namespace|要查询的集合|
|indexFilterSet|该查询是否有indexFilter|
|parsedQuery|查询条件|
|winningPlan|被选中的执行计划|
|winningPlan.stage|被选中执行计划的查询方式，常见：COLLSCAN/全表扫描，IXSCAN/所有扫描，FETCH/根据索引检索文档，SHARD_MERGE/合并分片结果，IDHACK/针对_id进行查询|
|winningPlan.inputStage|用来描述子stage，并且为其父stage提供文档和索引关键字|
|winningPlan.stage的child|如果此处是IXSCAN，表示进行的是index scanning|
|winningPlan.keyPattern|所扫描的index内容|
|winningPlan.indexName|选用的index|
|winningPlan.isMultiKey|是否是Multikey|
|winningPlan.direction|查询顺序，forward代表正序，backward代表倒序|
|filter|过滤条件|
|winningPlan.indexBounds|索引扫描范围|
|rejectedPlans|被拒绝的执行计划与winningPlan中的意义相同|
|serverInfo|MongoDB服务器信息|

### 3.4 慢查询分析
### 3.5 MongoDB索引底层实现原理分析

## 第四部分：MongoDB应用实战
### 4.1 MongoDB的适用场景
+ 网络数据
+ 缓存
+ 大尺寸
+ 高伸缩性的场景
+ 用于对象及JSON数据的存储

### 4.2 MongoDB的行业具体应用场景
+ 游戏场景
+ 物流场景
+ 社交场景
+ 物联网场景
+ 直播场景

### 4.3 如何抉择是否使用MongoDB


## 第五部分：MongoDB架构
## 第六部分：MongoDB集群高可用
## 第七部分：MongoDB安全认证