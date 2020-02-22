---
show: step
version: 2.0
enable_checker: true
---



## 课程介绍
本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎的环境中，安装部署 SequoiaSQL-PostgreSQL 实例，并进行简单的数据操作验证安装环境。

#### PostgreSQL 实例简介
SequoiaDB 巨杉数据库支持创建 PostgreSQL 实例，完全兼容 PostgreSQL 语法，用户可以使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改及其他操作。
#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-PostgreSQL 数据库实例节点，1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ef825173c9cd86053b61306ca6df9c65)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-PostgreSQL 实例均为 3.4 版本。



##  安装 SequoiaSQL-PostgreSQL 实例程序
安装 SequoiaSQL-PostgreSQL 实例程序需要 root 系统用户，程序已经提前放置在 /home/shiyanlou/sequoiadb-3.4 目录。

1）切换至 root 用户，在 `[sudo] password for shiyanlou:` 后输入当前用户的密码；

```shell
sudo su
```
> Note:
> 当前用户的密码在右侧工具栏 [SSH直连]


2）进入软件包放置目录；

```shell
cd /home/shiyanlou/sequoiadb-3.4
```

3）设置 SequoiaSQL-PostgreSQL 实例程序权限为可执行；

```shell
chmod +x sequoiasql-postgresql-3.4-x86_64-installer.run  
```
4）安装 SequoiaSQL-MySQL 实例；

```shell
./sequoiasql-postgresql-3.4-x86_64-installer.run --mode text
```

安装步骤选择说明请参考（本示例安装仅使用默认选择）：
* [SequoiaSQL-PostgreSQL 实例安装向导说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519628019-edition_id-304)

## 创建 PostgreSQL 实例

1）切换 sdbadmin 用户；

```shell
su - sdbadmin
```

2）进入 SequoiaSQL-MySQL 实例安装目录；

```shell
cd /opt/sequoiasql/postgresql
```

3）创建 PostgreSQL 实例；

```shell
bin/sdb_sql_ctl addinst myinst -D database/5432/
```

4）启动 PostgreSQL 实例；

```shell
bin/sdb_sql_ctl start myinst
```

5）检查创建的实例状态；

```shell
bin/sdb_sql_ctl status
```

操作截图：

![](https://doc.shiyanlou.com/courses/1539/1207281/2e3459abdbbd684d7fb320c0d4002695-0)

## 创建域、集合空间、集合

在 SequoiaDB 巨杉数据库引擎中创建域、集合空间、集合，用于 PostgreSQL 实例创建的外部表进行映射。

1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```
2）通过 javascript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost", 11810 ) ;
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

6）在 JSON 实例集合 company 中插入数据；

```javascript
db.company.employee.insert ({ "empno" : 1 , "ename" : "Georgi" , "age" : 48 }) ;
db.company.employee.insert ({ "empno" : 2 , "ename" : "Bezalel" , "age" : 21 }) ;
```

7）退出 SequoiaDB Shell ；

```javascript
quit ;
```

## 创建数据库及配置实例

1）创建 company 数据库实例；

```shell
bin/sdb_sql_ctl createdb company myinst
```

2）进入 PostgreSQL shell；

```shell
bin/psql -p 5432 company
```

3）加载SequoiaDB连接驱动；

```sql
CREATE EXTENSION sdb_fdw ;
```

4）配置与SequoiaDB连接参数；

```sql
CREATE SERVER sdb_server FOREIGN DATA WRAPPER sdb_fdw 
OPTIONS (address '127.0.0.1', service '11810', preferedinstance 'A', transaction 'on') ;
```

5）创建 company 数据库外表；

```sql
CREATE FOREIGN TABLE employee (
  empno integer, 
  ename text,
  age integer
) SERVER sdb_server 
OPTIONS (collectionspace 'company', collection 'employee', decimal 'on') ;
```


## 在 PostgreSQL 实例中对数据进行操作

1）更新表的统计信息；

```sql
ANALYZE employee ;
```

2）查询 employee 表中的数据；

```sql
SELECT * FROM employee ;
```

3）写入数据；

```sql
INSERT INTO employee VALUES (3, 'Jack', 27) ;
```

4）更改数据，更改 Jack 的岁数为 28；

```sql
UPDATE employee SET age = 28 WHERE ename = 'Jack' ;
```

5）查看员工 Jack 的岁数是否更改为28；

```sql
SELECT * FROM employee WHERE ename = 'Jack' ;
```

6）退出 PostgreSQL shell；

```sql
\q
```

## 存储引擎中查看数据
查看 SequoiaSQL-MySQL 实例中 employee 数据表在 SequoiaDB 数据库存储引擎中对应的分区表，并查看数据记录。

1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```

2）通过 javascript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost", 11810) ;
```

3）查询 employee 集合中的数据是否与PostgreSQL 实例操作的结果一致；

```javascript
db.company.employee.find () ;
```

3）退出 SequoiaDB Shell；

```javascript
quit ;
```

## 总结
通过本课程我们学会了 SequoiaSQL-PostgreSQL 实例的安装部署和基本操作、在 SequoiaDB 数据库引擎中创建数据域、集合空间和集合。

