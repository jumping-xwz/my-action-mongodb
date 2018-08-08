# my-action-mongodb

此处列出我关于 mongodb  的学习实践。

Official Document:  https://docs.mongodb.com/manual/

# Handbook

## 概念

MongoDB以BSON格式的文档（Documents）形式存储。Databases中包含集合（Collections），集合（Collections）中存储文档（Documents）。 

| SQL术语/概念  | MongoDB术语/概念   | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | table joins                         |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |


## 数据格式

MongoDB中保存的数据是`bson`格式的数据 

## 常用操作

#### 1. 连接数据库

```bash
mongo host:port/databaseName -u user -p passwd
mongo 127.0.0.1/local
```

#### 2.插入数据库

insert()操作会创建名为testDb的database和名为demoDb的collection（如果他们不存在的话）。 

```mysql
user testDb;
db.demoDb.inset({name: "wjpdev"});
db.getCollection("demoDb").insertOne({name: "wjpdev"});
```

#### 3. 查询数据

```mysql
db.demoDb.find(query, projection);

db.user.find({
    name:"wjpdev",
    age:30
});

db.user.find({
    name:"wjpdev",
    age:{
        $gte:30
    }
});

db.user.find({
    $or: [
        {name:"wjpdev"},
        {age:30}
    ]
});
```



#### 4. 对内嵌对象的查询

sample:

```json
db.user.insert(
{
    name:"张三",
    age:20,
    sex:"男",
    sale_books:[
        {
            name:"钢铁怎么练成的",
            price:69
        },
        {
            name:"Spring揭秘",
            price:87
        },
        {
            name:"MongoDb实战",
            price:54
        }
    ],
     address: {
         zipCode: 632185,
         name: "腾讯大学287号"
     }
}
);
```

使用 projection 的方式：

```mysql
db.user.find(
{
    "sale_books.name" : "MongoDb实战"
},
{
    # $用于遍历数组的下标
    "sale_books.$":1
});

或者这样

db.user.find(
sale_books: {
  $elemMatch: {
   name: "MongoDb实战"
  }
}
},
{
  "sale_books.$":1
}
)
```

#### 5. projection 用法

```mysql
# inclusion模式 指定返回的键，不返回其他键
db.collection.find(query, {name: 1, age: 1})

# exclusion模式 指定不返回的键,返回其他键,默认会返回_id,如果不想返回需要手动指定
db.collection.find(query, {name: 0, age: 0})

# 两种模式不可混用（因为这样的话无法推断其他键是否应返回）
# 错误
db.collection.find(query, {title: 1, by: 0})


# 相当于select name from user where age = 20;
db.user.find({age:20},{name:1});
```

#### 6. 修改数据

```mysql
db.collection.update(<filter>, <update>, <options>)
db.collection.updateOne(<filter>, <update>, <options>)
db.collection.updateMany(<filter>, <update>, <options>)
db.collection.replaceOne(<filter>, <update>, <options>)
```

sample:

```
db.getCollection('user').updateOne(
{
    name:"张三"
},
{
    $inc:
        {
            age:1
        }
}
);
```

`<options>` 参数：

```json
{
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
}
```

- upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入update指定的对象,true为插入，默认是false，不插入。
- multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- writeConcern :可选，抛出异常的级别。

#### 7. 册除数据

```mysql
db.collection.deleteMany()
db.collection.deleteOne()
```



#### 8. 备份数据

```bash
mongoexport -h host:port -u username -p password -d databaseName -c collectionName -o exportFilePath --type json/csv -f field

参数说明:
    -h : 主机地址
    -u : 用户名
    -p ：密码
    -d ：数据库名
    -c ：collection名
    -o ：输出的文件名
    --type ： 输出的格式，默认为json，可选为CSV
    -f ：输出的字段，如果-type为csv，则需要加上-f "字段名"

mongoexport -h localhost:27017
```

#### 9. 导入数据

```bash
mongoimport -h host:port -u username -p password  -d dbname -c collectionname --file filename --headerline --type json/csv -f field

参数说明：
    -h : 主机地址
    -d ：数据库名
    -c ：collection名
    --type ：导入的格式默认json
    -f ：导入的字段名
    --headerline ：如果导入的格式是csv，则可以使用第一行的标题作为导入的字段
    --file ：要导入的文件

mongoimport -h localhost:27017 -d local-test -c userSetting --file E:/userSetting.json --type json
```

#### 10. 执行mongo js脚本

script sample:

```javascript
function insertUser(country) {
    var collectionName = "user_info";
    var userData = db.getCollection(collectionName).find({"country":country});
    var desc = "Hello, my country is " + country;
    db.getCollection(collectionName).insert({"conuntry":country,"desc":desc});
}
["CN","UK"].forEach(function(c) {insertUser(c)});
```

run script:

```bash
mongo host:port/databaseName filePath.js
```



## 常用 Query 操作符



比较：

- `$eq`
- `$gt`
- `$gte`
- `$in`
- `$lt`
- `$lte`
- `$ne`
- `$nin`

逻辑：

- `$and`
- `$not`
- `$nor`
- `$or`

元素:

- `$exists`
- `$type`

计算:

- `$expr`
- `$jsonSchema`
- `$mod`
- `$regex`
- `$text`
- `$where`

位置:

- `$geoIntersects`
- `$geoWithin`
- `$near`
- `$nearSphere`

数组:

- `$all`
- `$elemMatch`
- `$size`

按位:

- `$bitsAllClear`
- `$bitsAllSet`
- `$bitsAnyClear`
- `$bitsAnySet`

注释:

- `$comment`

其它:

- `$`
- `$elemMatch`
- `$meta`
- `$slice`



