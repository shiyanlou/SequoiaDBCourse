---
show: step
version: 2.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 SequoiaSQL-SparkSQL 实例和启动了 thriftserver 的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的插入、查询操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中SequoiaSQL-SparkSQL 数据库实例包括 2 个 worker 节点，SequoiaDB 巨杉数据库包括 1 个引擎协调节点，1 个编目节点与 3 个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/f94f233be5f5d42622a2f29ec0c30c1f)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-SparkSQL 实例连接器均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-SparkSQL 实例的操作系统用户为 sdbadmin。
```shell
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量少于上图中的数量，请稍等初始化完成并重试该步骤

检查 SparkSQL 实例
```shell
jps
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/38da9d7707133b1d6623538ccc6b2ea8)

## 创建集合空间、集合

进入 SequoiaDB Shell，在 SequoiaDB 中创建集合空间 company，集合 employee，存储 SparkSQL 操作的数据。

1）使用 Linux 命令行进入 SequoiaDB Shell；
```shell
sdb
```

2）使用 javascript 语言连接协调节点；
```javascript
var db = new Sdb ("localhost", 11810) ;
```

3）创建 company_domains 逻辑域；

```javascript
db.createDomain ("company_domain", ["group1", "group2", "group3"], { AutoSplit : true } ) ;
```

4）创建 company 集合空间；
```javascript
db.createCS ("company", { Domain : "company_domain" } ) ;
```

5）创建 employee 集合，并使用 _id 进行 hash 分区；
```javascript
db.company.createCL ( "employee", { "ShardingKey" : { "_id" : 1 }, "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```

6）退出 SequoiaDB Shell；
```javascript
quit ;
```

## SparkSQL 实例中创建数据库及数据表
在 SparkSQL 实例中创建数据库及数据表并与 SequoiaDB 数据库引擎中的集合空间和集合关联。

#### 创建数据库

1）使用 Beeline 客户端连接 SparkSQL 实例服务；

```shell
/opt/spark/bin/beeline -u 'jdbc:hive2://localhost:10000'
```

2）在 SparkSQL 实例中创建数据库 company，并切换至company库；

```sql
create database company ;
use company ;
```


#### 关联实例与数据库引擎中的集合

在 SparkSQL 实例中创建表并与 SequoiaDB 数据库存储引擎中的集合空间、集合关联；

```sql
CREATE TABLE employee (
empno      INT,
ename STRING,
age INT
) USING com.sequoiadb.spark 
 OPTIONS ( host 'localhost:11810', collectionspace 'company', collection 'employee' ) ;
```

从 SparkSQL 实例中创建视图、表及数据类型对应关系的详细说明请参考：
[SparkSQL 实例访问SequoiaDB数据库存储引擎](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

## 关联表数据操作
使用 SparkSQL 实例操作关联表中的数据。

#### 通过关联表插入数据
在 SparkSQL 实例关联表 employee 中插入数据：
```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
INSERT INTO employee VALUES (10005, 'Kyoichi', 23) ;
INSERT INTO employee VALUES (10006, 'Anneke', 19) ;
```

#### 查询关联表插入数据

使用 SparkSQL 实例查询员工年龄为20至24之间的记录。

```sql
SELECT * FROM employee  WHERE age BETWEEN  20 AND 24;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/90bd7360e38d6af4ada588588f30c58b)


#### 通过已有表创建表
1）通过已有表 employee 创建表 employee_bak，并将表中的数据存放到指定域、集合空间中；
```sql
CREATE TABLE employee_bak USING com.sequoiadb.spark OPTIONS (
host 'localhost:11810',
domain 'company_domain',
collectionspace 'company_bak',
collection 'employee',
autosplit true,
shardingkey '{_id:1}',
shardingtype 'hash',
compressiontype 'lzw'
)
AS SELECT * FROM employee ;
```

2）查看 employee_bak 表中的数据；
```sql
SELECT * FROM employee_bak ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/9b2f6201953abb8679f52b5d3e02ffc1)

退出 SparkSQL 的客户端

```sql 
0: jdbc:hive2://localhost:10000> !q
```

## 总结

通过本课程，我们学习了通过 SparkSQL 操作 SequoiaDB 巨杉数据库中的数据。 同时讲解了 SequoiaDB 巨杉数据库所支持的 SparkSQL 语法。

