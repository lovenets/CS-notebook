# 安装并运行

参考[官方文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)

1. 下载 .msi 安装文件，运行，选择安装模式位 Custom，自定义安装目录，然后下一步
2. 是否安装为服务，建议取消，因为有些电脑可能会因为这个而导致安装不成功。反正可以在安装成功之后再将 MongoDB 配置为服务。

![安装为 service](img\安装为 service.jpg)

3. 安装成功之后进入安装目录，新建目录`\data\db`和`\data\log`。
4. 以管理员模式运行 cmd，进入安装目录下的 bin 目录，执行`mongod.exe --dbpath="c:\data\db"`，`c:\data\db`就是刚才新建的目录。MongoDB 运行成功的话就会看到这条命令：`[initandlisten] waiting for connections`。
5. 以管理员模式运行另一个 cmd，进入安装目录下的 bin 目录，执行`mongo.exe`，就可以连接到 MongoDB 数据库。

# 配置为 Windows 服务

1. 以管理员模式运行 cmd，进入安装目录下的 bin 目录，执行`mongod --dbpath C:\mongod\data\db --logpath C:\mongod\logs\mongod.log --serviceName "Mongodb" --install`。这里就是要设定数据库目录和日志目录，mongod.log 这个文件会自动生成。

2. 创建成功后可以用服务管理器手动管理服务，也可以用命令行工具。以管理员模式运行 cmd，进入安装目录下的 bin 目录：

   ```shell
   net start [ServiceName] `启动`
   net stop [ServiceName] `停止`
   ```

3. 删除服务，以管理员模式运行 cmd，进入安装目录下的 bin 目录，然后执行：

```shell
sc.exe delete MongoDB
```

# MongoDB 的使用场景

##  一、web 应用

1. MongoDB 经常被用作 web 应用的一级存储
2. 动态查询和二级索引可以让开发者写出类似 SQL 的查询语句
3. 可以方便地进行水平扩展

## 二、敏捷开发

1. MongoDB 不像关系型数据库有固定的数据模式，因此可以节省用来设计表结构的时间
2. 不用操心 ORM

## 三、分析和日志

1. 原子性更新可以让客户端高效地累加值
2. MongoDB 固定集合（Capped Collections）是性能出色且有着固定大小的集合，对于大小固定，我们可以想象其就像一个环形队列，当集合空间用完后，再插入的元素就会覆盖最初始的头部的元素。这个数据结构适合用来做日志，因为它可以只存储最常使用的 document

## 四、缓存

1. 把数据存入 MongoDB 很方便，因为无需过多考虑数据的结构
2. 查询速度快

## 五、数据模式有可能改变

如果使用的是 JSON 数据而且在存储前不知道 JSON 的结构，MongoDB 可以简化数据模型，因为它没有严格的数据模式，所以可以只管把数据存进去。

MongoDB 提供了`mongoimport`这个工具用来导入 JSON 数据。

# 局限性

1. 通常 MongoDB 都运行在64位的计算机上，如果是32位，那么只能使用4 GB 内存。
2. 当查询的数据大小超过了内存大小时，MongoDB 性能会显著下降，因为在这种情况下 MongoDB 需要访问硬盘。
3. MongoDB 用来存储的数据结构从数据大小的角度来看不是很高效，比如说每个 document 都存储了一个 key，这意味着每个 document 都需要额外的空间来存储 key 的名称。
4. MongoDB 是面向开发者的，它的查询语言对熟悉了 SQL 的 DBA 来说不是很友好。
5. 如果使用集群，那么每个结点都需要进行配置。

# CRUD 概览

MongoDB 的整体结构是 database(collection(document))，一个 database 由若干个 collection 组成，一个 collection 又由若干个 document 组成。所有的 document 都是 JSON 对象。

 ## 一、创建数据库角色

首先显示当前有哪些数据库：

```shell
show dbs
```

然后使用一个数据库：

```shell
use local
```

创建数据库角色：

```shell
db.createUser({
	 user: "user"
	pwd: "password",
	roles: ["dbAdmin","readWrite"]
});

```

## 二、创建 collection

```shell
db.createCollection("customers");
```

所有的命令都以 `db.[collection]`开头

## 三、插入 document

```shell
db.customers.insert({first_name:"John",last_name:"Doe"});
```

MongoDB 会自动为每一个 document 创建 id。

也可以一次性插入多个 document：

```shell
db.customers.insert([
	{
		first_name:"Micah",
		last_name:"Christenson"	
	},	
	{
		first_name:"Jordan",
		last_name:"Larson",
		gender:"female"
	}
]);
```

## 四、查询

查询使用的函数是`find`：

```
db.collection.find(query, projection)
```

两个参数都是可选的，第一个指定查询条件，第二个指定需要返回的字段（如果省略就是返回所有字段）。

### 1.无条件查询

```shell
db.customers.find().pretty();
```

`pretty`函数的作用是对结果进行格式化，提高可读性。

### 2.简单条件查询

查询 `first_name`字段的值为"Matthew"的 document：

```shell
db.customers.find({first_name: "Matthew"}).pretty();
```

如果要指定的字段是嵌套 JSON 对象的字段，那么要注意写法，要用双引号包围。比如查询条件为嵌套 JSON 对象 address 的 city 字段：

```shell
db.customers.find({"address.city": "New York"}); 
```

也可以用逻辑运算符：

```shell
 db.customers.find(
	{$or: [
			{ first_name: "Micah" },
			{ first_name: "Matthew" }
		 ]
	}
);
```

### 3.计数

```shell
db.customers.find().count();
```

### 4.limit

```shell
db.customers.find().limit(3);
```

### 5.排序

```shell
db.customers.find().sort({last_name:1});
```

上面是进行了升序排序，如果要降序则是`sort({last_name:-1})`。

### 6.其他

```shell
db.customers.find({age: {$lt: 30}}); // 小于
db.customers.find({age: {$gt: 25}}); // 大于
```

MongoDB 支持函数式编程：

```shell
db.customers.find().forEach( function (doc) {print("Customer name:" + doc.first_name)});
```

## 五、更新

更新要用`update`函数：

```
db.collection.update(query, update, options)
```

第一个参数是条件，第二个是更新的值，第三个是一些可选的操作。

（1）覆盖原有 document

```shell
db.customers.update(
	{
		first_name:"John"
	},
	{
		first_name:"John",
		last_name:"Doe",
		gender:"male"
	}
);
```

（2）添加、删除字段和修改字段名

```shell
db.customers.update({first_name:"Micah"},{$set:{age:25}});
```

```shell
db.customers.update({first_name:"Micah"},{$unset:{age:26}});
```

```shell
db.customers.update(
	{ first_name: "Amy" },
	{ $rename: { "gender": "sex" } }
);
```

（3）修改指定字段值

```shell
db.customers.update({first_name:"Micah"},{$inc:{age:1}});
```

`$inc` 是给数值型字段加1。

（4）如果 document 不存在则插入一个新的 document

```shell
db.customers.update(
	{ first_name: "Mary" },
	{ first_name: "Mary", last_name: "Samson" },
	{ upsert: true }
);
```

## 六、删除

```shell
db.customers.remove( { first_name: "Mary" } );
```

如果有多个 document 满足条件，可以只删除第一个：

```shell
db.customers.remove(
	{ first_name: "Amy" },
	{ justOne: true }
 );
```



