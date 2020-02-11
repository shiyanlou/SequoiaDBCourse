---
show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 SparkSQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的插入、查询操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中SequoiaSQL-SparkSQL 数据库实例包括2个worker节点，SequoiaDB 巨杉数据库包括1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/f94f233be5f5d42622a2f29ec0c30c1f)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-SparkSQL 实例连接器均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-SparkSQL 实例的操作系统用户为 sdbadmin。
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

检查 SparkSQL 实例
```
jps
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/38da9d7707133b1d6623538ccc6b2ea8)

## 创建数据库及数据表

#### 登录到 SparkSQL 实例 Beeline Shell
```
/opt/spark/bin/beeline -u jdbc:hive2://localhost:10000
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/01f1446fa12682164739d1dc36724334)

#### 创建数据库
在 SparkSQL 实例中创建数据库company，并切换至company库：
```
create database company;
use company;
```
## 关联集合空间、集合
在 SparkSQL 实例中关联 SequoiaDB 数据库中的集合空间、集合。

#### 在 SequoiaDB 中创建集合空间、集合
进入 SequoiaDB Shell，在 SequoiaDB 中创建集合空间 company，集合 employee：
```javascript
db = new Sdb();
db.createDomain("company_domains", ["group1", "group2", "group3"], {AutoSplit:true});
db.createCS("company",{Domain:"company_domains"});
db.company.createCL("employee",{"ShardingKey":{"_id":1},"ShardingType":"hash","ReplSize":-1,"Compressed":true,"CompressionType":"lzw","AutoSplit":true,"EnsureShardingIndex":false});
```

#### 在 SparkSQL 实例中创建表并与 SequoiaDB 集合空间、集合关联
进入SparkSQL Beeline Shell中，在 SparkSQL 实例中创建表并与 SequoiaDB 中的集合空间、集合关联：
```
create table employee(
empno      INT,
ename VARCHAR(128),
age INT
)using com.sequoiadb.spark options(
host 'localhost:11810',
collectionspace 'company',
collection 'employee'
);
```

>Note: 
>
> [SparkSQL 实例参数配置](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

## 关联表数据操作
使用 SparkSQL 实例操作关联表中的数据。

#### 通过关联表插入数据
在 SparkSQL 实例关联表 employee 中插入数据：
```
INSERT INTO employee VALUES (10001,'Georgi',48);
INSERT INTO employee VALUES (10002,'Bezalel',21);
INSERT INTO employee VALUES (10003,'Parto',33);
INSERT INTO employee VALUES (10004,'Chirstian',40);
INSERT INTO employee VALUES (10005,'Kyoichi',23);
INSERT INTO employee VALUES (10006,'Anneke',19);
```

#### 查询关联表插入数据
查询 SparkSQL 实例关联表 employee 中 age 大于20，小于30的数据：
```
select * from employee where age > 20 and age < 30;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/90bd7360e38d6af4ada588588f30c58b)


#### 通过已有表创建表
1）通过已有表 employee 创建表 employee_bak，并将表中的数据存放到指定域、集合空间中：
```
create table employee_bak using com.sequoiadb.spark options(
host 'localhost:11810',
domain 'company_domains',
collectionspace 'company_bak',
collection 'employee',
autosplit true,
shardingkey '{_id:1}',
shardingtype 'hash',
compressiontype 'lzw'
)
as select * from employee;
```

2）查看 employee_bak 表中的数据：
```
select * from employee
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/9b2f6201953abb8679f52b5d3e02ffc1)

## 总结

通过本课程，我们验证了 SequoiaDB 巨杉数据库所支持的 SparkSQL 语法，并对底层数据存储分布进行了直接验证。可以看出：
- 可以通过 SparkSQL 操作SequoiaDB巨杉数据库中的数据；
- SequoiaDB 巨杉数据库底层存储为分布式架构，数据可均匀分布在多个分区中；
