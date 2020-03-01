---
show: step
version: 3.0 
---

## 课程介绍

之前的小节通过 SequoiaSQL-MySQL、 SequoiaSQL-PostgreSQL 数据库实例对数据进行增删改查。同时对分布式存储引擎中的数据，SequoiaDB 巨杉数据库支持通过 JSON API 的方式直接进行查询与修改。SequoiaDB 巨杉数据库存储引擎的交互式命令行界面为 SequoiaDB Shell，使用 JavaScript 作为其脚本开发语言。

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

```shell
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量少于上图中的数量，请稍等初始化完成并重试该步骤。

## 创建域、集合空间和集合

1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```

2）通过 javascript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```

3）创建 company_domain 逻辑域；

```javascript
db.createDomain ("company_domain", ["group1", "group2", "group3"], { AutoSplit : true }) ;
```

4）创建 company 集合空间；

```javascript
db.createCS ("company", { Domain : "company_domain" }) ;
```

5）创建 employee 集合；

```javascript
db.company.createCL ("employee", {"ShardingKey" : { "_id" : 1} , "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```


>Note:
>
> - 集合（Collection）是数据库中存放文档的逻辑对象。任何一条文档必须属于一个且仅一个集合。
> - 集合空间（CollectionSpace）是数据库中存放集合的物理对象。任何一个集合必须属于一个且仅一个集合空间。
> - 域（Domain）是由若干个复制组（ReplicaGroup）组成的逻辑单元。每个域都可以根据定义好的策略自动管理所属数据，如数据切片和数据隔离等。
>


## 集合数据操作
通过 SequoiaDB Shell 操作集合中数据。

#### 集合中插入数据
在 JSON 实例集合 company 中插入数据：
```javascript
db.company.employee.insert ({ "empno" : 10001 , "ename" : "Georgi" , "age" : 48 }) ;
db.company.employee.insert ({ "empno" : 10002 , "ename" : "Bezalel" , "age" : 21 }) ;
db.company.employee.insert ({ "empno" : 10003 , "ename" : "Parto" , "age" : 33 }) ;
db.company.employee.insert ({ "empno" : 10004 , "ename" : "Chirstian" , "age" : 40 }) ;
db.company.employee.insert ({ "empno" : 10005 , "ename" : "Kyoichi" , "age" : 23 }) ;
db.company.employee.insert ({ "empno" : 10006 , "ename" : "Anneke" , "age" : 19 }) ;
```

#### 查询集合中的数据
查询集合 employees 中age 大于20，小于30的数据：
```javascript
db.company.employee.find ( { "age" : { "$gt" : 20 , "$lt" : 30 } } ) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/95b0770b9305772d6c795fe29a0b02d6)

#### 更新集合中的数据
1）集合 employees 中的数据，将 empno 为10001的记录 age 更改为34；

```javascript
db.company.employee.update ( { "$set" : { "age" : 34 } } , { "empno" : 10001 }) ;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功；

```javascript
db.company.employee.find ( { "empno" : 10001 } ) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/39b91f46f3ce79d27b342f224e4a8535)

#### 删除集合中的数据
1）删除集合 employees 中的数据，将 empno 为10006的记录删除；

```javascript
db.company.employee.remove ( { "empno" : 10006 } ) ;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除；

```javascript
db.company.employee.find ({},{"empno":""}) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/b7d47eedfcfbf827afc606f55af5565e)


## 索引使用
1）在集合 employee 的 ename 字段上创建索引；
```javascript
db.company.employee.createIndex ("idx_ename", { ename : 1 }, false) ;
```

2）查看集合 employee 上创建的索引；
```javascript
db.company.employee.listIndexes () ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/668e701adf5c780653096b32391a9f4c)

3）显示集合 employees 查询语句执行计划；

```javascript
db.company.employee.find ( { "ename" : "Georgi" } ).explain() ;
```

操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/0afc05df8deddc2ac5b285768c0b372e)

4）退出 SequoiaDB Shell；

```javascript
quit ;
```

## 总结

我们通过 javascript 语法对 SequoiaDB 巨杉数据库 JSON 实例进行了创建集合空间、集合、索引以及数据的 CRUD 基本操作。
