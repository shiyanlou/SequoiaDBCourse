---
show: step
version: 1.0
enable_checker: true
---


# HATP混合交易分析业务管理

## 课程介绍

本课程主要介绍 SequoiaDB 巨杉数据库的 HTAP 能力。通过连接不同的分区副本，实现 OLTP 与 OLAP 业务资源隔离，进而提高整体性能。


#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区三副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个 SequoiaSQL-PostgreSQL 数据库实例节点、1个 SparkSQL 实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本、SequoiaDB-Spark驱动连接器版本为3.4、 SparkSQL 版本为 2.4.4。





## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin。
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

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist  -t all -l -m local 
```

操作截图：

![870-1](https://doc.shiyanlou.com/courses/1544/1207281/4ea9a3ba18a9188dbb85a79a3d910f18)
>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤


## 在 MySQL 实例创建表

在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎。


1）使用 MySQL Shell 连接 SequoiaSQL-MySQL 实例；
```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建数据库实例，并切换到该数据库；

```
create database company ;
use company;
```

3）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

4）进行基本的数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

5）查看 MySQL 实例读写数据连接的协调节点；
```sql
SHOW VARIABLES LIKE '%sequoiadb_conn_addr%' ;
```

6）退出 MySQL Shell ；

```shell
\q
```

## 巨杉数据库设置

1）使用 Linux 命令行进去 SequoiaDB Shell；
```
sdb
```
2）使用javascript 语法连接协调节点，获取数据库连接；

```
var db=new Sdb("localhost", 11810) ;
```


3）修改巨杉数据库复制组实例id；

通过修改参数instanceid，确定每个副本的编号。这个编号为稍后的SparkSQL创建表做准备。

关于参数的更多说明，请参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190643-edition_id-0](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190643-edition_id-0)

修改数据节点复制组实例id：
```
db.updateConf ( { instanceid : 1 } ,{ GroupName : "group1" , svcname : "11820" } ) ;
db.updateConf ( { instanceid : 2 } ,{ GroupName : "group1" , svcname : "21820" } ) ;
db.updateConf ( { instanceid : 3 } ,{ GroupName : "group1" , svcname : "31820" } ) ;
db.updateConf ( { instanceid : 1 } ,{ GroupName : "group2" , svcname : "11830" } ) ;
db.updateConf ( { instanceid : 2 } ,{ GroupName : "group2" , svcname : "21830" } ) ;
db.updateConf ( { instanceid : 3 } ,{ GroupName : "group2" , svcname : "31830" } ) ;
db.updateConf ( { instanceid : 1 } ,{ GroupName : "group3" , svcname : "11840" } ) ;
db.updateConf ( { instanceid : 2 } ,{ GroupName : "group3" , svcname : "21840" } ) ;
db.updateConf ( { instanceid : 3 } ,{ GroupName : "group3" , svcname : "31840" } ) ;
```

修改协调节点读取数据时的读取策略
```
db.updateConf ( { preferedinstance : "1,2,3" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "11810" } ) ;
db.updateConf ( { preferedinstance : "3,2,1" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "11810" } ) ;
db.updateConf ( { preferedinstance : "2,1,3" , preferedinstancemode : "ordered" , preferedstrict : true} ,{ GroupName : "SYSCoord" , svcname : "11810" } ) ;
```

SDB数据库共有3个分区，分别是group1，group2，group3。每个分区有三个副本，一个主节点副本，两个从节点副本。通过上面的命令把三个副本分别标示为1，2，3。主节点的副本编号为1，从节点的副本编为2和3。

操作截图：

 ![870-5](https://doc.shiyanlou.com/courses/1544/1207281/d77d0622551d155b9a13b1a4c1d51bf1)

3）重启SDB数据库 

instanceid参数为重启后生效，修改完参数后，重启数据库。

```
sdbstop -t all
sdbstart -t all
```

操作截图：

 ![870-3](https://doc.shiyanlou.com/courses/1544/1207281/084fe6c8d5b31a5199582b76c8c70588)

 ![870-4](https://doc.shiyanlou.com/courses/1544/1207281/ce86694217fb24781a5759c3cdbf20b7)

4）查看参数修改状态

```
db.snapshot ( SDB_SNAP_CONFIGS , { } , { NodeName : "" , instanceid : "" } )
```

此时，所有数据节点的instanceid均已修改完成。

操作截图：

 ![870-6](https://doc.shiyanlou.com/courses/1544/1207281/fbae38cccb352b93f8ec66fc7993ed97)



## SparkSQL设置

SparkSQL设置完成后，用于处理OLAP类的业务。主要负责处理聚合类查询或者同级业务。

这里我们把SaprkSQL中的表连接至从节点。

1）登录 Beeline 客户端，连接 SparkSQL 实例；

```
/opt/spark/bin/beeline -u 'jdbc:hive2://localhost:10000' 
```

2）创建 company 数据库；
```sql
create database company ;
```

2）在SparkSQL中创建表的结构

```sql
CREATE TABLE company.employee(
    empno int,
    ename string,
    age int
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

这里对涉及实例选择的参数进行说明：
- preferredinstance：指定分区优先选择的节点实例。取值可以为"M","S","A"（不区分大小写），以及实例ID的组合。 如果指定多个"M", "S", "A"实例，则只有第一个生效。实例ID取值为1-255。如果多个实例ID和"M"一起指定，则在有多个实例符合时的会在符合的实例中优先选择主节点；而当没有实例符合时，也会在其它节点中优先选择主节点。如果多个实例ID和"S"一起指定，则在有多个实例符合时的会在符合的实例中优先选择备节点；而当没有实例符合时，也会在其它节点中优先选择备节点。"A"表示任意节点。如果没有匹配的实例，将随机选择。

- preferredinstancemode：在preferredinstance有多个实例符合时的选择模式，取值可以是"random","ordered"。"random"表示从候选实例中随机选择，"ordered"表示按候选实例的顺序选择。

- preferredinstancestrict：在preferredinstance指定的实例ID都不符合时是否报错。

更多的SparkSQL创建表参数说明，请参考如下链接：
[巨杉数据库 Spark 实例建表说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

3）数据的查询；
此时SparkSQL从副本3 读取数据。

```sql
select * from company.employee ;
```

操作截图：

 ![870-7](https://doc.shiyanlou.com/courses/1544/1207281/f1904ffca785124225d2c09936512753)



## 总结

本课程通过给数据组的副本设置实例 id 和协调节点设置数据读取的顺序，使得 MySQL 实例连接的11810协调节点，读取数据的实例 id 顺序为1，2；SparkSQL 实例读取的数据节点实例 id 为3，2。由于生产环境中我们部署节点一般是1台机器1个数据组为1个副本，这样的读取顺序能够把两个不同的实例分开，实现资源隔离。



