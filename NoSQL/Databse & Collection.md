# Database

`use [db]`用来选择某个已经存在的数据库，或者创建一个数据库（当第一次向这个数据库存储数据时 MongoDB 就会创建这个数据库）。

命名规范：大小写敏感；Windows 环境下 database 名称不能包含这些字符 /\. "$*<>:|?

```shell
use myNewDB

db.myNewCollection1.insertOne( { x: 1 } )
```

如果 database 和 collection 都不存在那么`insertOne`就会创建两者。

# Collection

## 一、创建

命名规范：以下划线或者字母开头，不能出现以下情况：

- 包含 $
- 空字符串
- 包含 null 字符
- 不能以`system.`作为前缀

### 显式

```shell
db.createCollection()
```

这个方法用来显式创建 collection，并且用来指定一些设置，比如大小、校验规则等。如果并不想在创建的时候进行设置，那么就没有必要显式创建。

### 隐式

如果一个 collection 不存在，那么 MongoDB 会在你第一次存储数据的时候创建 collcetion：

```shell
db.myNewCollection2.insertOne( { x: 1 } )
db.myNewCollection3.createIndex( { y: 1 } )
```

这两个操作都会在各自的 collection 不存在时创建 collection。

## 二、标识符

每个 collection 都会被自动地赋予一个不可变的 UUID，在分布式存储的情况下，同一  collection 的每个副本或者碎片的 UUID 都相同。

```shell
db.getCollectionInfos()
```

[getCollectionInfos()](https://docs.mongodb.com/manual/reference/method/db.getCollectionInfos/#db.getCollectionInfos) 可以检索 collection 的 UUID。

## 三、View

可以对已存在的 databse 或者 view 创建 view。view 是只读的。

### 创建

注意，不能在一个 database 中对另一个 database 的 collection 创建 view。

```shell
db.createView(<view>, <source>, <pipeline>, <options> )
```





