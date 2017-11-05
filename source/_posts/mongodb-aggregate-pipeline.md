---
title: MongoDB之aggregate, pipeline及map_reduce
date: 2017-11-05 16:18:15
tags: MongoDB
categories: 数据库
---

### MongoDB聚合
聚合操作处理数据记录并返回计算结果。 聚合操作将多个文档中的值组合在一起，并可对分组数据执行各种操作，以返回单个结果。 
在SQL中的 count(*)与group by组合相当于mongodb 中的聚合功能。

基本语法: `db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)`

### 实例讲解
数据:
```
{
   _id: 100, title: 'MongoDB Overview', description: 'MongoDB is no sql database', by_user: 'Maxsu', 
   url: 'http://www.yiibai.com', tags: ['mongodb', 'database', 'NoSQL'], likes: 100
},
{
   _id: 101, title: 'NoSQL Overview', description: 'No sql database is very fast', by_user: 'Maxsu',
   url: 'http://www.yiibai.com', tags: ['mongodb', 'database', 'NoSQL'], likes: 10
},
{
   _id: 102, title: 'Neo4j Overview', description: 'Neo4j is no sql database', by_user: 'Kuber',
   url: 'http://www.neo4j.com', tags: ['neo4j', 'database', 'NoSQL'], likes: 750
},
{
   _id: 103, title: 'MySQL Overview', description: 'MySQL is sql database', by_user: 'Curry',
   url: 'http://www.yiibai.com/mysql/', tags: ['MySQL', 'database', 'SQL'], likes: 350
}
```

聚合字段`by_user`的总数, 其中`by_user`作为新集合的`_id`, 统计的总数放在num_tutorial字段的值中. 
`db.article.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])`
结果如下:
```
{ "_id" : "Curry", "num_tutorial" : 1 }
{ "_id" : "Kuber", "num_tutorial" : 1 }
{ "_id" : "Maxsu", "num_tutorial" : 2 }
```
上述用例的Sql等效查询`select by_user, count(*) as num_tutorial from `article` group by by_user;`

### 其他用于聚合的表达式
$sum	从集合中的所有文档中求出定义的值。 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])
$avg	计算集合中所有文档的所有给定值的平均值。db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}])
$min	从集合中的所有文档获取相应值的最小值。	 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}])
$max	从集合中的所有文档获取相应值的最大值。	 db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}])
$push	将值插入到生成的文档中的数组中。	   db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}])
$addToSet	将值插入生成的文档中的数组，但不会创建重复项。	db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}])
$first	根据分组从源文档获取第一个文档。 通常情况下，这只适用于以前应用的“$sort”阶段。	db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}])
$last	根据分组从源文档获取最后一个文档。通常情况下，这只适用于以前应用的“$sort”阶段。	db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}])

### 管道
在UNIX命令中，shell管道可以对某些输入执行操作，并将输出用作下一个命令的输入。 
MongoDB也在聚合框架中支持类似的概念。每一组输出可作为另一组文档的输入，并生成一组生成的文档(或最终生成的JSON文档在管道的末尾)。
这样就可以再次用于下一阶段等等。

以下是在聚合框架可能的阶段 -

$project - 用于从集合中选择一些特定字段。
$match - 这是一个过滤操作，因此可以减少作为下一阶段输入的文档数量。
$group - 这是上面讨论的实际聚合。
$sort - 排序文档。
$skip - 通过这种方式，可以在给定数量的文档的文档列表中向前跳过。
$limit - 限制从当前位置开始的给定数量的文档数量。
$unwind - 用于展开正在使用数组的文档。使用数组时，数据是预先加入的，此操作将被撤销，以便再次单独使用文档。 因此，在这个阶段，将增加下一阶段的文件数量。

### map_reduce
Map-reduce是将大量数据合并为有用的聚合结果的数据处理范例。 MongoDB使用mapReduce命令进行map-reduce操作。MapReduce通常用于处理大型数据集。
MapReduce命令:
```
db.collection.mapReduce(
   function() {emit(key,value);},  //map function
   function(key,values) {return reduceFunction}, //reduce function
   {
    query: query,  
    out: out,    //指定结果集以什么方式存储，可选参数包括：  
                //replace:如果文档(table)存在，则替换table，  
                //merge:如果文档中存在记录，则覆盖已存在的文档记录  
                //reduce: 如果文档中存在相同key的记录了，则先计算两条记录，然后覆盖旧记录  
                // {inline:1}  在内存中存储记录，不写入磁盘(用户数据量少的计算) 
    sort: sort,  
    limit: limit,  
    finalize: function  //这个function主要用来在存入out之前可以修改数据，function(key,values) {   
                        //return modifiedValues;}  
    scope: document,    //设置参数值，在这里设置的值在map，reduce，finalize函数中可见
    jsMode:boolean      //是否减少执行过程中BSON和JS的转换，默认true]
                        //false时 BSON-->JS-->map-->BSON-->JS-->reduce-->BSON,可处理非常大的mapreduce,
                        //true时BSON-->js-->map-->reduce-->BSON
                        
    verbose:boolean     //是否产生更加详细的服务器日志，默认true
    keytemp：boolean    //true或false，表明结果输出到的collection是否是临时的，如果为true，则会在客户端连接中断后自动删除，如果你用的是MongoDB的mongo客户端连接，  
                        //那必须exit后才会删除。如果是脚本执行，脚本退出或调用close会自动删除结果collection    

   }
)

必备参数：map,reduce, out
```
`map: function() {emit(this.cat_id,this.goods_number); }`
说明: 函数内部要调用内置的emit函数,cat_id代表根据cat_id来进行分组,
goods_number代表把文档中的goods_number字段映射到cat_id分组上的数据,
其中this是指向向前的文档的,这里的第二个参数可以是一个对象,如果是一个对象的话,也是作为数组的元素压进数组里面;

reduce: function(cat_id,all_goods_number) {return Array.sum(all_goods_number)}, 
cat_id代表着cat_id当前的这一组,all_goods_number代表当前这一组的goods_number集合,这部分返回的就是结果中的value值;

out: <output>, // 输出到某一个集合中,注意本属性来还支持如果输出的集合如果已经存在了,那是替换,合并还是继续reduce? 另外还支持输出到其他db的分片中,具体用到时查阅文档,筛选出现的键名分别是_id和value;

query: <document>, // 一个查询表达式,是先查询出来,再进行mapReduce的

sort: <document>, // 发往map函数前先给文档排序

limit: <number>, // 发往map函数的文档数量上限,该参数貌似不能用在分片模式下的mapreduce

finalize: function(key, reducedValue) {return modifiedObject; }, // 从reduce函数中接受的参数key与reducedValue,并且可以访问scope中设定的变量

scope: <document>, // 指定一个全局变量,能应用于finalize和reduce函数

jsMode: <boolean>, // 布尔值，是否减少执行过程中BSON和JS的转换，默认true,true时BSON-->js-->map-->reduce-->BSON,false时 BSON-->JS-->map-->BSON-->JS-->reduce-->BSON,可处理非常大的mapreduce。

verbose: <boolean> // 是否产生更加详细的服务器日志，默认true

 



map-reduce函数首先查询集合，然后将结果文档映射到发出的键值对，然后根据具有多个值的键进行减少。
* map是一个JavaScript函数，它将一个值与一个键映射并发出一个键值对；
* reduce是一个javascript函数，可以减少或分组具有相同键的所有文档；
* out指定map-reduce查询结果的输出位置,一般是输出到另一个新的集合里；
* query指定选择文档的可选选择条件；
* sort指定可选的排序条件；
* limit指定可选的最大文档数；

MapReduce查询可用于构建大型复杂聚合查询。 使用自定义JavaScript函数可以使用MapReduce，它非常灵活和强大。
