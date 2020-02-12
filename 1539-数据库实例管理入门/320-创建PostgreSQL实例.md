---
show: step
version: 1.0
enable_checker: true
---

# PostgreSQL 实例简介
SequoiaDB支持创建PostgreSQL实例，完全兼容PostgreSQL语法，用户可以使用SQL语句访问SequoiaDB数据库，完成对数据的增、删、查、改及其他操作。

## 1 课程介绍
本实验基于Sequoiadb数据库提供的Docker镜像，能够一步一步的带领你在linux环境中部署巨杉数据库的PostgreSQL实例。

## 2 准备课程环境
课程环境是一个docker 容器，已经安装了一个单副本的巨杉数据库，包括了多种实例的安装介质，可以在这容器中完成多种实例的安装配置。

在属主机上启动docker课程容器，并进入container。
```
docker run -it --privileged=true --name sdbtestfu -h sdb sdbinstance5
```
一下步骤在Container中执行。

启动sdb：
```
su - sdbadmin
sdbstart -t all
sdblist -l
```
进入PostgreSQL安装目录，检查是否有PostgreSQL安装包.
```
cd /sequoiadb/sequoiadb-3.2.4
ls -l ls -l sequoiasql-postgresql-3.2.4-x86_64-installer.run
```

## 3 安装PostgreSQL实例,使用缺省的安装参数：
```
./sequoiasql-postgresql-3.2.4-x86_64-installer.run --mode text
```
```
Please wait while Setup installs SequoiaSQL PostgreSQL Server on your computer.

 Installing
 0% ______________ 50% ______________ 100%
 #########################################

----------------------------------------------------------------------------
Setup has finished installing SequoiaSQL PostgreSQL Server on your computer.
```
## 4 创建pgsql实例,数据库及配置实例
```
su - sdbadmin
cd /opt/sequoiasql/postgresql/bin
```
检查pgsql实例情况
```
./sdb_sql_ctl status

INSTANCE   PID        SVCNAME    PGDATA                                   PGLOG                                   
Total: 0; Run: 0
```
创建PostgreSQL实例：
```
./sdb_sql_ctl addinst myinst -D database/5432/
Adding instance myinst ...
ok
```
再次检查实例
```
./sdb_sql_ctl status
INSTANCE   PID        SVCNAME    PGDATA                                   PGLOG                                   
myinst     -          -          /opt/sequoiasql/postgresql/bin/database/5432/ /opt/sequoiasql/postgresql/myinst.log   
Total: 1; Run: 0
```
启动pgsql实例
```
./sdb_sql_ctl start myinst
Starting instance myinst ...
ok (PID: 120650)
```
检查实例状态：
```
./sdb_sql_ctl status      
INSTANCE   PID        SVCNAME    PGDATA                                   PGLOG                                   
myinst     120650     5432       /opt/sequoiasql/postgresql/bin/database/5432/ /opt/sequoiasql/postgresql/myinst.log   
Total: 1; Run: 1
```
创建 SequoiaSQL PostgreSQL 的 database
```
./sdb_sql_ctl createdb pgsdb myinst

Creating database myinst ...
ok
```
进入 SequoiaSQL PostgreSQL shell 环境
```
./psql -p 5432 pgsdb
```
管理sdb与pgsql 的数据库和表

加载SequoiaDB连接驱动
```
pgsdb=# create extension sdb_fdw;
CREATE EXTENSION
```
配置与SequoiaDB连接参数
```
pgsdb=# create server sdb_server foreign data wrapper sdb_fdw options(address '127.0.0.1', service '11810', preferedinstance 'A', transaction 'off');
CREATE SERVER
```
退出psql命令行：
```
pgsdb=# \q
```
## 5 在sequoisdb中创建测试集合并关联到pgsql实例
在sdb中创建CL

查看sdb中的CL
```
sdbadmin@sdb:/opt/sequoiasql/postgresql/bin$ sdb
Welcome to SequoiaDB shell!
help() for help, Ctrl+c or quit to exit
> db=new Sdb()
localhost:11810
Takes 0.003692s.
> db.list(4);
Return 0 row(s).
Takes 0.000809s.
```
创建集合空间pgsdb名字与PostgreSQL中创建的数据库名称相同：
```
> db.createCS("pgsdb")
localhost:11810.pgsdb
Takes 0.004101s.
```
创建集合testtab1并插入数据：
```
> db.pgsdb.createCL("testtab1")
localhost:11810.pgsdb.testtab1
Takes 0.450711s.
> db.pgsdb.testtab1.insert({my_name:"Name1",my_age:40,my_address:"chengd1.sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.001143s.
> db.pgsdb.testtab1.insert({my_name:"Name2",my_age:42,my_address:"chengd2.sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.000590s.
> db.pgsdb.testtab1.insert({my_name:"Name3",my_age:43,my_address:"chengd3.sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.000631s.
> db.pgsdb.testtab1.insert({my_name:"Name4",my_age:44,my_address:"chengd4.sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.000586s.

>  db.list(SDB_LIST_COLLECTIONS)
{
  "Name": "pgsdb.testtab1"
}
```
退出sdb命令行：
```
> quit
```
关联SequoiaDB的集合空间与集合,创建一个外表，结构与sdb中的集合一致：
进入psql命令行：
```
./psql -p 5432 pgsdb

create foreign table testtab1 (my_name text, my_age int,my_address text) server sdb_server options ( collectionspace 'pgsdb', collection 'testtab1', decimal 'on');
CREATE FOREIGN TABLE
```
更新表的统计信息
```
pgsdb=# analyze testtab1;
ANALYZE
```
检查表：
```
pgsdb=# \d
              List of relations
 Schema |   Name   |     Type      |  Owner   
--------+----------+---------------+----------
 public | testtab1 | foreign table | sdbadmin
(1 row)
```
看到一个foreign table。
```
gsdb=# \d testtab1;
        Foreign table "public.testtab1"
   Column   |  Type   | Modifiers | FDW Options 
------------+---------+-----------+-------------
 my_name    | text    |           | 
 my_age     | integer |           | 
 my_address | text    |           | 
Server: sdb_server
FDW Options: (collectionspace 'pgsdb', collection 'testtab1', "decimal" 'on')
```
## 6 在PostgreSQL实例中对数据进行操作
```
pgsdb=# select * from testtab1;
 my_name | my_age |   my_address    
---------+--------+-----------------
 Name1   |     40 | chengd1.sichuan
 Name2   |     42 | chengd2.sichuan
 Name3   |     43 | chengd3.sichuan
 Name4   |     44 | chengd4.sichuan
 
 pgsdb=# delete from testtab1 where my_age=44;
DELETE 1
pgsdb=# select * from testtab1;
 my_name | my_age |   my_address    
---------+--------+-----------------
 Name1   |     40 | chengd1.sichuan
 Name2   |     42 | chengd2.sichuan
 Name3   |     43 | chengd3.sichuan
(3 rows)

pgsdb=# insert into testtab1(my_name,my_age,my_address) values('Name4',44,'chengdu4.sichuan');
INSERT 0 1

pgsdb=# select * from testtab1;
 my_name | my_age |    my_address    
---------+--------+------------------
 Name1   |     40 | chengd1.sichuan
 Name2   |     42 | chengd2.sichuan
 Name3   |     43 | chengd3.sichuan
 Name4   |     44 | chengdu4.sichuan
(4 rows)
```
退出psql命令行：
```
pgsdb=# \q
```
## 7 在sdb中观看数据，数据与psql中看到的一致.
```
sdb
> db=new Sdb()
> db.pgsdb.testtab1.find()
{
  "_id": {
    "$oid": "5e12eed9a56655cfdcebe905"
  },
  "my_name": "Name1",
  "my_age": 40,
  "my_address": "chengd1.sichuan"
}
{
  "_id": {
    "$oid": "5e12eee6a56655cfdcebe906"
  },
  "my_name": "Name2",
  "my_age": 42,
  "my_address": "chengd2.sichuan"
}
{
  "_id": {
    "$oid": "5e12eef1a56655cfdcebe907"
  },
  "my_name": "Name3",
  "my_age": 43,
  "my_address": "chengd3.sichuan"
}
{
  "_id": {
    "$oid": "5e12f2d816120eac4b000000"
  },
  "my_name": "Name4",
  "my_age": 44,
  "my_address": "chengdu4.sichuan"
}
{
  "_id": {
    "$oid": "5e12f31e16120eac4b000001"
  },
  "my_name": "Name5",
  "my_age": 45,
  "my_address": "chengdu5.sichuan"
}
Return 5 row(s).
Takes 0.001041s.
> quit
```

删除pgsql 实例
```
root@sdb:/sequoiadb/sequoiadb-3.2.4# cd /opt/sequoiasql/postgresql
root@sdb:/opt/sequoiasql/postgresql# ls
bin  checksum.md5  compatible.sh  conf  include  lib  myinst.log  preUninstall.sh  share  uninstall  uninstall.dat
root@sdb:/opt/sequoiasql/postgresql# ./uninstall
```

## 结束课程

执行完成，退出docker
```
#exit
```
在属主机中，删除Container
```
docker rm sdbtestfu
```
