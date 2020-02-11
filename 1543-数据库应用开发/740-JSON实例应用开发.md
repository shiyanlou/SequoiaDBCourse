---
show: step
version: 1.0 
---

## 课程介绍

之前的小节通过 SequoiaSQL-MySQL、 SequoiaSQL-PostgreSQL 数据库实例对数据进行增删改查。同时，对分布式存储引擎中的数据，SequoiaDB 巨杉数据库支持通过 JSON API 的方式直接进行查询与修改。SequoiaDB 巨杉数据库存储引擎的交互式命令行界面为 SequoiaDB Shell，使用 JavaScript 作为其脚本开发语言。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/dd11cb78ba9732f36f58883df952282a)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库的操作系统用户为 sdbadmin。
```
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本

```
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤

## 创建域、集合空间、集合
1）进入到 SequoiaDB Shell，并连接数据库
```javascript
db = new Sdb();
```

2）在 SequoiaDB 中创建域 company_domains、集合空间 company ，集合 employees ：
```javascript
db.createDomain("company_domains", ["group1", "group2", "group3"], {AutoSplit:true});
db.createCS("company",{Domain:"company_domains"});
db.company.createCL("employee",{"ShardingKey":{"_id":1},"ShardingType":"hash","ReplSize":-1,"Compressed":true,"CompressionType":"lzw","AutoSplit":true,"EnsureShardingIndex":false});
```

## 集合数据操作
通过 SequoiaDB Shell 操作集合中数据。

#### 集合中插入数据
在 JSON 实例集合 company 中插入数据 ：
```javascript
db.company.employee.insert({"empno":10001,"ename":"Georgi","age":48});
db.company.employee.insert({"empno":10002,"ename":"Bezalel","age":21});
db.company.employee.insert({"empno":10003,"ename":"Parto","age":33});
db.company.employee.insert({"empno":10004,"ename":"Chirstian","age":40});
db.company.employee.insert({"empno":10005,"ename":"Kyoichi","age":23});
db.company.employee.insert({"empno":10006,"ename":"Anneke","age":19});
```

#### 查询集合中的数据
查询集合 employees 中age 大于20，小于30的数据：
```javascript
db.company.employee.find({"$and":[{"age":{"$gt":20}},{"age":{"$lt":30}}]});
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/95b0770b9305772d6c795fe29a0b02d6)

#### 更新集合中的数据
1）更新JSON 实例集合 employees 中的数据，将 empno 为10001的记录 age 更改为34：
```javascript
db.company.employee.update({"$set":{"age":34}},{"empno":10001});
```

2）查询数据结果确认 empno 为10001的记录更新是否成功：
```javascript
db.company.employee.find({"empno":10001});
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/39b91f46f3ce79d27b342f224e4a8535)

#### 删除集合中的数据
1）删除集合 employees 中的数据，将 empno 为10006的记录删除：
```javascript
db.company.employee.remove({"empno":10006});
```

2）查询数据结果确认 empno 为10006的记录是否成功删除：
```javascript
db.company.employee.find();
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/b7d47eedfcfbf827afc606f55af5565e)


## 索引使用
1）在集合 employee 的 ename 字段上创建索引：
```javascript
db.company.employee.createIndex("idx_ename",{ename:1},false);
```

2）查看集合 employee 上创建的索引：
```javascript
db.company.employee.listIndexes();
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/668e701adf5c780653096b32391a9f4c)

3）显示集合 employees 查询语句执行计划：
```javascript
db.company.employee.find({"ename":"Georgi"}).explain();
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/0afc05df8deddc2ac5b285768c0b372e)

## 总结

通过本课程，我们验证了 SequoiaDB 巨杉数据库 JSON 数据操作接口，可以看出：
- SequoiaDB 巨杉数据库底层存储为分布式架构，数据可均匀分布在多个分区中；
- SequoiaDB 巨杉数据库 JSON 实例提供丰富的数据操作接口；
