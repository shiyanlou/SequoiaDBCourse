---
show: step
version: 59.0
enable_checker: true
---

# HATP混合交易分析业务管理

## 课程介绍

本课程主要介绍 SequoiaDB 巨杉数据库的 HTAP 能力，在 HTAP 场景下数据库产品的资源隔离能力至关重要，目前巨杉数据库的资源隔离方案有：

1）基于多租户架构规划：一个数据库通常承载着多种业务系统，为了避免不同业务之间的性能影响，根据业务系统的存储量、并发大小、数据生命周期等划分不同的数据域承载相应的数据读写；

2）基于副本的资源隔离：不同业务通过连接不同的分区副本，实现 OLTP 与 OLAP 业务资源隔离，进而提高整体性能；

由于业务形式的发展，越来越多的需求需要对交易数据进行实时分析，例如推荐、决策、监控等，传统的处理办法是使用ETL的方式把OLTP业务产生的数据同步到OLAP的数据数据库，导致了数据需要在不同的数据库之间流转，耗费时间成本的同时需要耗费人力成本运维多套数据库产品。

SequoiaDB巨杉数据库采用“计算存储分离”架构，支持 MySQL、PostgreSQL 与 SparkSQL 三种关系型数据库实例、类 MongoDB 的 JSON 文档类数据库实例、以及 S3 对象存储与 Posix 文件系统的非结构化数据实例。在事务场景可以利用 SDBAPI、MySQL 和 PGSQL 实例对数据进行操作，在分析场景借助分布式计算框架 Spark 的并发计算性能，提高计算效率。


#### 部署架构

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区三副本，其中包括：1 个 SequoiaSQL-MySQL 数据库实例节点、1 个 SparkSQL 实例节点、3 个引擎协调节点，3 个编目节点与 3 个数据节点。

![870-1](https://doc.shiyanlou.com/courses/1544/1207281/136759e29ff1e16b9d0903cdf18bea8c-0)

关于 SequoiaDB 巨杉数据库系统架构的详细信息，请参考如下链接：

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本、SequoiaDB-Spark驱动连接器版本为3.4、 SparkSQL 版本为 2.4.4。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin:

```shell
su - sdbadmin
```

>Note:
>
>用户 sdbadmin 的密码为 `sdbadmin`

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![870-2](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist  -l
```

操作截图：

![870-3](https://doc.shiyanlou.com/courses/1544/1207281/6068aa2c80ecba03a4c5ed401f8f68ec-0)

>Note:
>
>如果显示的节点数量与预期不符，请稍等节点初始化完成后再重试该步骤。

## 在 MySQL 实例创建表

在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎。

1）登录 MySQL Shell；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建数据库，并切换到该数据库；

```sql
CREATE DATABASE company ;
USE company ;
```

3）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (
    empno INT AUTO_INCREMENT PRIMARY KEY,
    ename VARCHAR(128),
    age INT
) ;
```

4）向表中插入数据；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

5）查看数据情况；

```sql
SELECT * FROM employee ;
```

6）查看 MySQL 实例读写数据连接的协调节点；

```sql
SHOW VARIABLES LIKE '%sequoiadb_conn_addr%' ;
```

7）退出 MySQL Shell ；

```sql
\q
```

## SequoiaDB 巨杉数据库访问隔离设置

数据库访问隔离功能的实现主要是通过修改数据库节点配置参数 instanceid 、 preferedinstance 和 preferedinstancemode、preferedstrict 来实现。

关于数据库配置的更多说明，请参考如下链接：

* [ SequoiaDB 数据库配置 ](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190643-edition_id-0)

#### 修改数据节点配置

1）使用 Linux 命令行进去 SequoiaDB Shell ；

```shell
sdb
```

2）使用 javascript 语法连接协调节点，获取数据库连接；

```javascript
var db=new Sdb("localhost", 11810) ;
```
>Note:
>
>以下 3-5 步骤会报 -264 错误，这个是因为修改数据节点 instanceid 参数需要重启节点后才生效，可通过 getLastErrObj() 接口获取详细信息。

3）修改 11820 数据节点实例 id 为 1；

```javascript
db.updateConf ( { instanceid : 1 } ,{svcname : {"$in":["11820"]}} ) ;
```

4）修改 21820 数据节点实例 id 为 2；

```javascript
db.updateConf ( { instanceid : 2 } ,{svcname : {"$in":["21820"]}} ) ;
```

5）修改 31820 数据节点实例 id 为 3；

```javascript
db.updateConf ( { instanceid : 3 } ,{svcname : {"$in":["31820"]}} ) ;
```

操作截图：

 ![870-4](https://doc.shiyanlou.com/courses/1544/1207281/aecb10be850489dad57c1f96c4f9afc1-0)


#### 修改协调节点配置

1）修改 11810 协调节点读取数据时的读取策略；

```javascript
db.updateConf ( { preferedinstance : "1,2,3" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "11810" } ) ;
```

2）修改 21810 协调节点读取数据时的读取策略；

```javascript
db.updateConf ( { preferedinstance : "2,1,3" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "21810" } ) ;
```

3）修改 31810 协调节点读取数据时的读取策略；

```javascript
db.updateConf ( { preferedinstance : "3,2,1" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "31810" } ) ;
```

preferedinstance 指定执行读请求时优先选择的实例。上面命令为三个协调节点分配了不同的读取策略。例如：31810优先读取实例id为3的数据节点，即31820。

操作截图：

 ![870-5](https://doc.shiyanlou.com/courses/1544/1207281/b7104906de30188804edf405dca5ee75-0)

4）退出 SequoiaDB Shell；

```javascript
quit ;
```

## 重启 SequoiaDB 数据节点

数据节点 instanceid 参数为需要节点重启后生效，修改完参数后，重启数据节点。

```shell
sdbstop -r data
sdbstart -r data
```

操作截图：

 ![870-3](https://doc.shiyanlou.com/courses/1544/1207281/4a47a0db6e60853dedfcfdf08a5ca249-0)

## 查看节点参数修改状态

1）进入 SequoiaDB Shell；

```shell
sdb
```

2）使用 javascript 语法连接协调节点，获取数据库连接；

```javascript
var db=new Sdb("localhost", 11810) ;
```

3）查看数据节点参数修改状态；

```javascript
db.snapshot ( SDB_SNAP_CONFIGS , {Role : "data" } , { NodeName : "" , instanceid : ""} ) ;
```

此时，所有数据节点的 instanceid 均已修改完成。

操作截图：

 ![870-6](https://doc.shiyanlou.com/courses/1544/1207281/089c566d13e548eda5d6a3adfdd7de1c-0)

4）查看协调节点参数修改状态；

```javascript
db.snapshot ( SDB_SNAP_CONFIGS , {Role : "coord" } , { NodeName : "" , preferedinstance : ""} ) ;
```

操作截图：

 ![870-6](https://doc.shiyanlou.com/courses/1544/1207281/b132ec3cc727b718dc7b93c27871a99d-0)

 此时，所有数据节点的 preferedinstance 均已修改完成。

 5）退出 SequoiaDB Shell；

```javascript
quit ;
```

## SparkSQL 设置

SparkSQL 用于处理 OLAP 类的业务，例如推荐、决策、监控等。

1）登录 Beeline 客户端，连接 SparkSQL 实例；

```shell
/opt/spark/bin/beeline -u 'jdbc:hive2://localhost:10000'
```

2）创建 company 数据库；

```sql
CREATE DATABASE company ;
```

3）在 SparkSQL 中创建表并设置预读数据节点实例 id 顺序为优先使用实例 3，再使用实例 2；

```sql
CREATE TABLE company.employee(
    empno INT,
    ename STRING,
    age INT
) USING com.sequoiadb.spark OPTIONS (
    host 'localhost:11810',
    collectionspace 'company',
    collection 'employee',
    username '',
    password '',
    preferredinstance '3,2',
    preferredinstancemode 'ordered',
    preferredinstancestrict true
) ;
```

更多的 SparkSQL 创建表参数说明，请参考如下链接：

[巨杉数据库 Spark 实例建表说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

3）数据的查询，此时 SparkSQL 从副本 3 读取数据；

```sql
SELECT * FROM company.employee ;
```

操作截图：

 ![870-7](https://doc.shiyanlou.com/courses/1544/1207281/8a092beecee3d6b5b1a46ac393367ba4-0)

## 总结

本课程通过给数据组的副本设置实例 id 和协调节点设置数据读取的顺序，使得 SparkSQL 实例读取的数据节点实例 id 为 3，2。由于生产环境中 SequoiaDB 巨杉数据库集群部署的节点一般是 1 台机器 1 个数据组为 1个副本，这样的读取顺序能够把两个不同的实例分开，实现资源隔离。
