---
title: Python 和 MongoDB 其实很配
categories: [Tech]
tags: [Python, mongo]
date: 2021-08-30
---

MongoDB 其实就是一个大大的`JSON`，在 Python 的世界里`dict`也是最吃香的类型，所以，他们天生就是一对。

## MongoDB 的安装

推荐使用 Docker 来部署管理，一行命令就可以搞定，官方版本：

```bash
docker run -d --name mongodb \
    -e MONGO_INITDB_ROOT_USERNAME=admin \
    -e MONGO_INITDB_ROOT_PASSWORD=admin \
    -v ~/data/mongo_dir:/data/db \
    -p 27017:27017 \
    mongo
```

官方版本的 Docker 啥都好，就是体积有点大。还有一个小体积的`alpine`版本，开发时使用很方便，不过不能配置账户和密码。

```bash
docker run -d --name mongo-lite \
  -p 27018:27017 \
  -v ~/data/mongo_lite:/data/db \
  mvertes/alpine-mongo
```

如果想尝试 Mongo 的命令行 （Mongo Shell），直接进到 Docker 里：

```bash
$ docker exec -it mongo-lite mongo
MongoDB shell version v4.0.6
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
...

> use mydb
switched to db mydb

> db.User.insertOne({"name":"Toby",age:18})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("612c84c5d93795436ad27ebc")
}

> db.User.find()
{ "_id" : ObjectId("612c84c5d93795436ad27ebc"), "name" : "Toby", "age" : 18 }
```

Mongo Shell 官方文档： <https://docs.mongodb.com/manual/reference/mongo-shell/>

## PyMongo 五分钟上手

安装 PyMongo 可以通过`pip`搞定。

```
pip install pymongo
```

以下内容也可以参考官方文档： <https://pymongo.readthedocs.io/en/stable/>

### 连接数据库

常见方式如下：

```python
from pymongo import MongoClient

# 连接有密码的Mongo
client = MongoClient('mongodb://admin:admin@localhost:27017/')

# 连接没密码的Mongo
client = MongoClient('mongodb://localhost:27018/')

# 列出所有已经存在的DB
for db in client.list_databases():
    print(db)

# 使用Mongo里的某个DB，这个DB可以不存在，后面写数据时会被创建出来
db = client.mydb
```

### 插入数据

插入的每条数据都是一个`dict`，一样的字段允许类型不一样，也允许每次插入的数据字段不一样，可以理解成动态类型数据，你想放什么都行，唯一的约束就是他们会被放在同一个`Document`里。

```python
# 插入一条数据
def add_one_user():
    db.User.insert_one({
        'name': 'Toby',
        'age': 18
    })

# 插入多条数据
def add_many_users():
    db.User.insert_many([{
        'name': 'Tom',
        'age': 10
    }, {
        'name': 'Toby',
        'age': 'unknown',
        'hobbies': ['write bugs', 'raise dogs']
    }])
```

这里的`User`约等于关系型数据库的表，但它的名字叫`Document`，每次数据插入完成后会返回一个`_id`，这是 Mongo 里最重要的东西了，它就是靠这个`_id`来保证数据的一致性，后续的数据修改和删除主要就是靠这个`_id`来完成，所以一般针对某条特定的数据的处理，都是需要先查询它的`_id`，然后再进行后面的操作。

### 查询数据

```python
# 查询多个数据
def show_users():
    # 一个表里所有数据
    for e in db.User.find():
        print(e)

    # 匹配条件的多条数据
    for e in db.User.find({'name': 'Toby'}):
        print(e)


# 查询单个数据
def query_user(name):
    return db.User.find_one({'name': name})


# 忽略大小写
def query_user_ignore_case(name):
    return db.User.find_one({'name': re.compile(name, re.IGNORECASE)})


# 使用运算符 https://docs.mongodb.com/manual/reference/operator/query/
def query_teenager():
    return db.User.find_one({'age': {'$lt': 18}})
```

Mongo 的查询主要还是依赖 DB 自己提供的运算符，在 PyMongo 里要注意，这里不会抛出异常，如果找不到数据，默认返回 None。

- 通过运算符查询数据：<https://docs.mongodb.com/manual/reference/operator/query/>
- 通过聚合查询数据：<https://docs.mongodb.com/manual/aggregation/>

### 修改数据

```python
# 修改一个数据
def update_user(user, attributes: dict):
    user.update(attributes)
    result = db.User.replace_one({'_id': user['_id']}, user, upsert=True)
    return {'affected_count': result.modified_count}

u = query_user_ignore_case('toby')
result = update_user(u, {'code': 'python'})

# 修改多个数据，注意有坑，Replace 和 Update是不一样的
def update_many():
    todo = [
        UpdateOne({'age': 19}, {'$set': {'name': 'Toby'}}),
        ReplaceOne({'name': 'Tom'}, {'age': 19}),  # name 会被吃掉
    ]
    result = db.User.bulk_write(todo)
    print(result.matched_count)
```

`Replace` 是替换，所以要带上原有字段，这里有点坑。`Update` 不接受单独的`dict`，需要用 `$set` / `$unset` 来标识修改的字段的方式。

```json
[
  { "$set": { "status": "Modified", "comments": ["$misc1", "$misc2"] } },
  { "$unset": ["misc1", "misc2"] }
]
```

### 删除数据

```python
def delete_user(name):
    result = db.User.delete_one({'name': name})
    return {'affected_count': result.deleted_count}
```

删除多个数据：

```python
>>> db.test.count_documents({'x': 1})
3
>>> result = db.test.delete_many({'x': 1})
>>> result.deleted_count
3
>>> db.test.count_documents({'x': 1})
0
```

## 常见问题

### 有什么办法可以让 Mongo 不自动添加 `_id`到我的数据里？

几乎没有，这是 MongoDB 的特性决定的，如果你的数据没有 ID 的话，并且进行高并发插入时，大概率会遇到`BulkWriteError`这个错误。

```python
>>> doc = {}
>>> collection.insert_many(doc for _ in range(10))
Traceback (most recent call last):
...
pymongo.errors.BulkWriteError: batch op errors occurred
>>> doc
{'_id': ObjectId('560f171cfba52279f0b0da0c')}

>>> docs = [{}]
>>> collection.insert_many(docs * 10)
Traceback (most recent call last):
...
pymongo.errors.BulkWriteError: batch op errors occurred
>>> docs
[{'_id': ObjectId('560f1933fba52279f0b0da0e')}]
```

如果你不想要自动生成的`ID`，可以自己在插入数据前指定这个字段。

### 为啥我指定了`_id`还是查询不到我的数据？

比如我要查询数据库里的某个 post:

```python
>>> post_id_as_str = str(post_id)
>>> posts.find_one({"_id": post_id_as_str}) # No result
```

因为 pyMongo 里的这个 ID 不是字符串类型，你需要做一下数据转换。

```python
from bson.objectid import ObjectId

# The web framework gets post_id from the URL and passes it as a string
def get(post_id):
    # Convert from string to ObjectId:
    document = client.db.collection.find_one({'_id': ObjectId(post_id)})
```

### 用标准库里的 json 模块来序列化和反序列化 Mongo 的数据会有什么问题？

有一些数据类型在反序列后会得不到预期的结果，比如`ObjectId` 和 `DBRef`，PyMongo 为了解决这个问题自己封装了一个辅助类`json_util`，可以很好的解决这些问题。

```python
from bson.json_util import loads
from bson.json_util import dumps
```

## 总结

Mongo 属于非关系型数据库，使用 Mongo 作为 DB 的思维需要做比较大的转变：

1. 关系型数据库一般读写容易，修改难，容易理解
2. 非关系型数据库一般是读写改容易，设计难（相对而言）

> 关系型数据库支持 ACID (Atomicity, Consistency, Isolation, Duration) 即原子性，一致性，隔离性和持续性。 相对而言，NoSQL 采用更宽松的模型 BASE (Basically Available, Soft state, Eventual Consistency) 即基本可用，软状态和最终一致性。

NoSQL 在精心的设计下查询性能会更高，数据结构也十分有弹性，特别适合快速发展和属性不确定的产品功能，但 Mongo 不支持事务，如何确保数据一致性是个挺大的挑战。

在选择上可以考虑从以下角度去思考：

1. 需要 ACID 还是 BASE
2. 需要结构化数据还是非结构化数据
3. 需要对数据进行灵活扩展
4. 开发人员的经验

很多情况只考虑最后一点就可以了。
