---
title: python 扩展之 PyMongo安装和使用
permalink: python-pymongo-install-use
date: 2020-05-06 20:06:06
updated: 2020-05-06 20:59:50
tags: 
    - python
    - PyMongo
categories: python
toc: true
is_original: 0
excerpt: PyMongo是一个用于MongoDB的Python工具，也是一个被推荐的Python操作MongoDB数据库的方式。
---

PyMongo是一个用于MongoDB的Python工具，也是一个被推荐的Python操作MongoDB数据库的方式。
本翻译文档包含以下内容：

使用PyMongo插入数据
使用PyMongo查询数据
使用PyMongo更新数据
使用PyMongo删除数据
使用PyMongo进行数据聚合
使用PyMongo创建索引

[原文链接https://docs.mongodb.com/getting-started/python/introduction/](https://docs.mongodb.com/getting-started/python/introduction/)

### 安装PyMongo
```
pip install pymongo
```

### 引入pymongo
在一个Python交互shell中完成此过程，或者也可以在其他Python环境中完成，例如Python脚本、Python模块、Python项目。

从pymongo中引入MongoClicent。
```
from pymongo import MongoClient
```

### 创建一个连接
使用MongoClient创建一个连接：
```
client = MongoClient()
```

如果你没有特别为MongoClient指定参数，MongoClient将使用MongoDB默认参数即localhost接口和27017端口。
你可以使用一个完整的MongoDB URI来定义连接：
```
client = MongoClient("mongodb://mongodb0.example.net:27019")
```

这个MongoClient代表了一个运行于mongodb.example.net端口号为27019的MongoDB连接。

### 访问数据库对象
第一个你用pymongo来交互的基本类是Database，它代表了MongoDB中的数据库(database)结构。数据库保存了逻辑上相关的集合组。MongoDB将在第一次使用新的数据库的时候自动创建它。

你可以使用属性访问的方式，赋值一个数据库（例如名字为primer）给本地变量db，如下所示：
```
db = client.primer
```

你也可以使用字典形式访问一个数据库，这会移除Python特有的命名限制，如下所示：
```
db = client['primer']
```

### 访问集合对象
第二个你将使用pymongo进行交互的基本类是Collection，它代表了MongoDB中的集合（collection）结构。集合（collection）保存了逻辑上相关的文档组。
你可以直接使用字典形式访问集合或者通过一个访问一个数据库对象的属性来访问集合，如下所示：
```
db.dataset
db['dataset']
```

你也可以给一个变量进行赋值，以在其他地方使用这个集合，如下所示：
```
coll = db.dataset
coll = db['dataset']
```

### 使用PyMongo插入数据

#### 概览
你可以使用insert_one()方法和insert_many()方法来向MongoDB的集合中插入文档。如果你所插入的集合在MongoDB中不存在，MongoDB将为你自动创建一个集合。

#### 先决条件
在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```

### 插入一个文档
向集合restaurants中插入一个文档。如果集合不存在，这个操作将创建一个新的集合。
```
from datetime import datetime
result = db.restaurants.insert_one(
    {
        "address": {
            "street": "2 Avenue",
            "zipcode": "10075",
            "building": "1480",
            "coord": [-73.9557413, 40.7720266]
        },
        "borough": "Manhattan",
        "cuisine": "Italian",
        "grades": [
            {
                "date": datetime.strptime("2014-10-01", "%Y-%m-%d"),
                "grade": "A",
                "score": 11
            },
            {
                "date": datetime.strptime("2014-01-16", "%Y-%m-%d"),
                "grade": "B",
                "score": 17
            }
        ],
        "name": "Vella",
        "restaurant_id": "41704620"
    }
)
```

这个操作返回了一个InsertOneResult对象，它包括了insert_id属性表示被插入的文档的_id。访问insert_id的方式如下：
```
result.inserted_id
```

你插入的文档的ObjectId将和如下所示的不同。
```
ObjectId("54c1478ec2341ddf130f62b7")
```

如果你传递给insert_one()方法的参数不包含_id字段，MongoClient将自动添加这个字段并且生成一个ObjectId设置为这个字段的值。

### 使用PyMongo查询数据
#### 概览
你可以通过find()方法产生一个查询来从MongoDB的集合中查询到数据。MongoDB中所有的查询条件在一个集合中都有一个范围。

查询可以返回在集合中的所有数据或者只返回符合筛选条件（filter）或者标准（criteria）的文档。你可以在文档中指定过滤器或者标准，并作为参数传递给find()方法。

find()方法返回一个查询结果的游标，这是一个产生文档的迭代对象。

#### 先决条件
本示例中使用test数据库中的restaurants集合。与使用示例数据填充集合有关的介绍请见Import Example Dataset。

在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```

### 在一个集合中查询所有文档
调用find()方式不需要传值即可得到集合中所有的文档。举例来说，如下所示的操作即是返回restaurants集合中所有文档。
```
cursor = db.restaurants.find()
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果包含了所有restaurants集合中的所有文档。

#### 指定相等条件
对某一个字段的相等条件查询有如下形式：
```
{ <field1>: <value1>, <field2>: <value2>, ... }
```
如果字段（<field>）在某个文档的某一个数组内，则使用点操作符（dot notation）去访问该字段。

#### 使用一个顶级字段进行查询
如下所示的操作将查询borough字段等于Manhattan的文档。
```
cursor = db.restaurants.find({"borough": "Manhattan"})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

#### 在一个嵌入式的文档中查询
要指定嵌入文档中的字段的查询条件，需要使用点操作符。使用点操作符需要使用双引号将字段名包裹。下面的操作将指定一个文档的地址字典中的邮编字段的一个相等的条件。
```
cursor = db.restaurants.find({"address.zipcode": "10075"})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

更多的关于嵌入式文档的查询条件信息，请参阅Embedded Documents。

#### 在一个数组中查询
grades数组包含一个嵌入式文档作为其元素。在该文档的字段上指定一个相等条件需要用到点操作符。使用点操作符需要使用双引号将字段名包裹。如下所示的查询将查询一个有嵌入式文档的grades字段，该字段中的grade等于B。
```
cursor = db.restaurants.find({"grades.grade": "B"})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

更多的关于数组内查询条件的信息，例如数组中特殊的混合条件，请参阅Array及$elemMatch操作符。

#### 使用操作符指定条件
MongoDB支持使用操作符指定查询条件，例如比较操作符。虽然这其中有一些例外，例如$or和$and条件操作符。使用操作符进行查询一般有如下形式：
```
{ <field1>: { <operator1>: <value1> } }
```

完整的操作符列表请查阅query opeartors(http://docs.mongodb.org/manual/reference/operator/query)。

#### 大于（$gt）操作符
查询字段grades包含一个嵌入式文档，其中score大于30。
```
cursor = db.restaurants.find({"grades.score": {"$gt": 30}})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

#### 小于（$lt）操作符
查询字段grades包含一个嵌入式文档，其中score小于10。
```
cursor = db.restaurants.find({"grades.score": {"$lt": 10}})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

#### 组合条件
你可以使用逻辑与（AND）或者逻辑或（OR）组合多个查询条件。

#### 逻辑与
你可以使用一个列表指定一个逻辑与条件查询操作，使用逗号分隔条件。
```
cursor = db.restaurants.find({"cuisine": "Italian", "address.zipcode": "10075"})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

#### 逻辑或
你可以使用$or操作符进行逻辑或条件的指定。
```
cursor = db.restaurants.find(
    {"$or": [{"cuisine": "Italian"}, {"address.zipcode": "10075"}]})
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件的文档。

#### 对结果进行排序
要指定结果集的顺序，可以通过追加sort()方法进行查询。给sort()方法传递需要排序的字段和配需类型等。

- pymongo.ASCENDING表示升序排序。
- pymongo.DESCENDING表示降序排序。
如果要通过多个键星星排序，可以传递键的列表和以及对应的排序类型的列表。举例来说，如下操作将返回restaurants集合中所有的文档，并且先通过borough字段进行升序排序，然后在每个borough内，通过"address.zipcode"字段进行升序排序。
```
import pymongo
cursor = db.restaurants.find().sort([
    ("borough", pymongo.ASCENDING),
    ("address.zipcode", pymongo.ASCENDING)
])
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
```
结果将只包含符合条件且经过排序的文档。

### 使用PyMongo更新数据
#### 概览
你可以使用update_one()和update_many方法更新集合中的文档。update_one()方法一次更新一个文档。使用update_many()方法可以更新所有符合条件的文档。方法接受以下三个参数：

- 一个筛选器，可以对符合条件的文档进行更新。
- 一个指定的修改语句
- 自定义更新时的操作参数
要指定更新时的过滤器，使用和查询条件时相同的结构即可。参见使用PyMongo查询数据获取查询条件的信息。
你不能更新_id字段。

#### 先决条件
本示例中使用test数据库中的restaurants集合。与使用示例数据填充集合有关的介绍请见Import Example Dataset。

在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```

#### 更新特定的字段
要改变一个特定字段的值，MongoDB提供了更新操作符，例如$set操作符可以修改值。例如$set之类的操作符将在没有该字段的时候新建这个字段。可以查阅update operators作为参考。

#### 更新顶级字段
如下操作将更新第一个符合name等于Juni这个条件的文档。使用$set操作符更新cuisine字段且将lastModified修改为当前日期。
```
result = db.restaurants.update_one(
    {"name": "Juni"},
    {
        "$set": {
            "cuisine": "American (New)"
        },
        "$currentDate": {"lastModified": True}
    }
)
```
这个操作返回了一个UpdateResult对象。这个对象报告了符合条件的文档数目以及被修改的文档数目。

要查看符合筛选器条件的文档数目，通过访问UpdateResult对象的matched_count属性。
```
result.matched_count
matched_count值为：1
```

要查看更新操作中被修改的文档数目，通过访问UpdateResult对象的modified_count属性。

modified_count值为： 1

#### 更新嵌入式文档中的字段
要更新一个嵌入式文档中的字段，需要使用点操作符。当使用点操作符时，使用点操作符需要使用双引号将字段名包裹。下面的操作将更新address字段中的street值。
```
result = db.restaurants.update_one(
    {"restaurant_id": "41156888"},
    {"$set": {"address.street": "East 31st Street"}}
)
```
这个操作返回了一个UpdateResult对象。这个对象报告了符合条件的文档数目以及被修改的文档数目。

要查看符合筛选器条件的文档数目，通过访问UpdateResult对象的matched_count属性。
```
result.matched_count
matched_count值为：1
```
要查看更新操作中被修改的文档数目，通过访问UpdateResult对象的modified_count属性。

modified_count值为：1


#### 更新多个文档
update_one()方法更新了一个文档，要更新多个文档，需要使用update_many()方法。下面的操作将更新所有的address.zipcode等于10016以及cuisine等于Other的文档，将cuisine字段设置为Category To Be Determined以及将lastModified更新为当前日期。
```
result = db.restaurants.update_many(
    {"address.zipcode": "10016", "cuisine": "Other"},
    {
        "$set": {"cuisine": "Category To Be Determined"},
        "$currentDate": {"lastModified": True}
    }
)
```
这个操作返回了一个UpdateResult对象。这个对象报告了符合条件的文档数目以及被修改的文档数目。

要查看符合筛选器条件的文档数目，通过访问UpdateResult对象的matched_count属性。
```
result.matched_count
matched_count值为：20
```
要查看更新操作中被修改的文档数目，通过访问UpdateResult对象的modified_count属性。

modified_count值为：20

#### 替换一个文档
要替换整个文档（除了_id字段），将一个完整的文档作为第二个参数传给update()方法。替代文档对应原来的文档可以有不同的字段。在替代文档中，你可以忽略_id字段因为它是不变的。如果你包含了_id字段，那它必须和原文档的值相同。

重要：
在更新之后，该文档将只包含替代文档的字段。

在如下的更新操作后，被修改的文档将只剩下_id、name和address字段。该文档将不再包含restaurant_id、cuisine、grades以及borough字段。
```
result = db.restaurants.replace_one(
    {"restaurant_id": "41704620"},
    {
        "name": "Vella 2",
        "address": {
            "coord": [-73.9557413, 40.7720266],
            "building": "1480",
            "street": "2 Avenue",
            "zipcode": "10075"
        }
    }
)
```
replace_one操作返回了一个UpdateResult对象。这个对象报告了符合条件的文档数目以及被修改的文档数目。

要查看符合筛选器条件的文档数目，通过访问UpdateResult对象的matched_count属性。
```
result.matched_count
matched_count值为：20
```

要查看更新操作中被修改的文档数目，通过访问UpdateResult对象的modified_count属性。

modified_count值为：20

### 使用PyMongo删除数据
#### 概览

你可以使用delete_one()以及delete_many()方法从集合中删除文档。方法需要一个条件来确定需要删除的文档。

要指定一个删除条件，使用和查询条件时相同的结构即可。参见使用PyMongo查询数据获取查询条件的信息。

#### 先决条件
本示例中使用test数据库中的restaurants集合。与使用示例数据填充集合有关的介绍请见Import Example Dataset。

在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```
#### 步骤
删除所有符合条件的文档
下面的操作将删除所有复合条件的文档。
```
result = db.restaurants.delete_many({"borough": "Manhattan"})
```
这个操作返回了一个DeleteResult对象。这个对象报告了被删除的文档数目。

要查看被删除的文档数目，通过访问DeleteResult对象的deleted_count属性。
```
result.deleted_count
deleted_count值为：10259
```
如果你已经插入或者更新了文档，那么你得到的结果将和示例不同。

#### 删除所有文档
要删除一个集合中的所有文档，给delete_many()方法传递一个空的条件参数即可。
```
result = db.restaurants.delete_many({})
```
这个操作返回了一个DeleteResult对象。这个对象报告了被删除的文档数目。

要查看被删除的文档数目，通过访问DeleteResult对象的deleted_count属性。
```
result.deleted_count
deleted_count值为：15100
```

如果你已经插入或者更新了文档，那么你得到的结果将和示例不同。

#### 销毁一个集合
删除所有文档的操作只会清空集合中的文档。该集合以及集合的索引将依旧存在。要清空一个集合，销毁该集合以及它的索引并且重建集合和索引可能是相比于清空一个集合更加高效的操作方式。使用drop()方法可以销毁一个集合，包括它所有的索引。
```
db.restaurants.drop()
```

### 使用PyMongo进行数据聚合
#### 概览
MongoDB可以进行数据聚合操作，例如可以针对某一字段进行分组或者对某一字段不同的值进行统计。

使用aggregate()方法可以使用基于步骤的聚合操作。appregate()方法接受多个数组作为每一步的操作。每一个阶段按照顺序处理，描述了对数据操作的步骤。
```
db.collection.aggregate([<stage1>, <stage2>, ...])
```

#### 先决条件
这部分的例子使用了test数据库中的restaurants集合。需要查看在集合中填充的实例数据，请参阅Import Example Dataset。

在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```

#### 根据一个字段分组文件并计算总数
使用$group操作符去利用一个指定的键进行分组。在$group操作中，指定需要分组的字段为_id。$group通过字段路径访问字段，该字段需要有一个美元符号$作为前缀。$group操作可以使用累加器对本次分组进行计算。下面的例子将使用borough字段对restaurants集合进行操作，并且使用$sum累加器进行文档的统计计算。
```
cursor = db.restaurants.aggregate(
    [
        {"$group": {"_id": "$borough", "count": {"$sum": 1}}}
    ]
)
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
结果将由如下文档组成：
{u'count': 969, u'_id': u'Staten Island'}
{u'count': 6086, u'_id': u'Brooklyn'}
{u'count': 10259, u'_id': u'Manhattan'}
{u'count': 5656, u'_id': u'Queens'}
{u'count': 2338, u'_id': u'Bronx'}
{u'count': 51, u'_id': u'Missing'}
```

_id字段包含了不同的borough值，即根据键的值进行了分组。

#### 筛选并分组文档
使用$match操作来删选文档。$match使用MongoDB查询语法。下面的管道使用$match来对restaurants进行一个borough等于"Queens"且cuisine等于Brazilian的查询。接着$group操作对命中的文档使用address.zipcode字段分组并使用$sum进行统计。$group通过字段路径访问字段，该字段需要有一个美元符号$作为前缀。
```
cursor = db.restaurants.aggregate(
    [
        {"$match": {"borough": "Queens", "cuisine": "Brazilian"}},
        {"$group": {"_id": "$address.zipcode", "count": {"$sum": 1}}}
    ]
)
```
迭代游标（cursor）并且打印文档内容。
```
for document in cursor:
    print(document)
结果将由如下文档组成：

{u'count': 1, u'_id': u'11368'}
{u'count': 3, u'_id': u'11106'}
{u'count': 1, u'_id': u'11377'}
{u'count': 1, u'_id': u'11103'}
{u'count': 2, u'_id': u'11101'}
```
_id字段包含了不同的zipcode值，即根据键的值进行了分组。

### PyMongo上的索引
#### 概览
索引可以对查询的高效执行起到支持。如果没有索引，MongoDB必须进行全表扫描，即扫描集合中的每个文档，来选择符合查询条件的文档。如果一个适当的索引存在于一个查询中，MongoDB可以使用索引限制必须检查文档的数量。

使用create_index()方法来为一个集合创建索引。所以可以对查询的高效执行起到支持。MongoDB会在创建文档的时候自动为_id字段创建索引。

要为一个或多个字段创建索引，使用一个包含字段和索引类型的列表作为参数：
```
[ ( <field1>: <type1> ), ... ]
```
要创建一个升序的索引，指定pymongo.ASCENDING为索引类型（）。
要创建一个降序的索引，指定pymongo.DESCENDING为索引类型（）。
create_index()只会在索引不存在的时候创建一个索引。

#### 先决条件
下面的例子将使用test数据库中的restaurants集合。需要查看在集合中填充的实例数据，请参阅Import Example Dataset。

在Python命令行或者IDLE中，使用MongoClient连接一个正在运行的MongoDB实例，且已经打开test数据库。
```
from pymongo import MongoClient

client = MongoClient()
db = client.test
```

#### 创建一个单字段索引
在restaurants集合中的cuisine字段上创建自增的索引。
```
import pymongo
db.restaurants.create_index([("cuisine", pymongo.ASCENDING)])
```
该方法将返回被创建的索引的名字。

"u'cuisine_1'"

#### 创建一个复合索引
MongoDB支持在多个字段上创建符合索引。这几个字段的命令将声明这个索引包含的键。举个例子，下面的操作将在cuisine字段和address.zipcode字段上创建一个复合索引。该索引将先对cuisine的值输入一个升序的命令，然后对address.zipcode的值输入一个降序命令。
```
import pymongo
db.restaurants.create_index([
    ("cuisine", pymongo.ASCENDING),
    ("address.zipcode", pymongo.DESCENDING)
])
```
该方法将返回被创建的索引的名字。
```
"u'cuisine_1_address.zipcode_-1'"
```

> 本文转载自煦的博客的个人博客,原文链接为https://www.cnblogs.com/zhouxuchen/p/5544227.html
