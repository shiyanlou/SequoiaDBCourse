---
show: step
version: 1.0
enable_checker: true
---



## 1 课程介绍
#### PostgreSQL 实例简介
SequoiaDB支持创建PostgreSQL实例，完全兼容PostgreSQL语法，用户可以使用SQL语句访问SequoiaDB数据库，完成对数据的增、删、查、改及其他操作。

##  root用户安装 PostgreSQL 实例程序
#### 获取 root 密码
1）点击右侧工具栏的 “SSH 直连” 链接即可弹出shiyanlou的用户密码；
2）使用系统用户 shiyanlou 重置 root 密码；
```shell
sudo passwd root
```
3）切换到 root 用户；
```
su 
```
4）解压安装包；
```
tar -zxvf sequoiadb-3.4-linux_x86_64.tar.gz
```

5）进入解压目录；
```shell
cd sequoiadb-3.4
```

6）设置文件权限为可执行
```
chmod +x sequoiasql-postgresql-3.4-x86_64-installer.run 
```
6）安装 SequoiaSQL-MySQL 实例；
```
./sequoiasql-postgresql-3.4-x86_64-installer.run --mode text
```
安装步骤选择说明请参考：
* [SequoiaSQL-PostgreSQL 实例安装向导说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1521595270-edition_id-0)

## 4 创建 PostgreSQL 实例

1）切换 sdbadmin 用户；
```
su - sdbadmin
```
2）进入 SequoiaSQL-MySQL 实例安装目录；
```
cd /opt/sequoiasql/postgresql
```

3）查看MySQL实例情况；
```
./bin/sdb_sql_ctl status
```

预期输出：
INSTANCE   PID        SVCNAME    SQLDATA                                  SQLLOG                                  
Total: 0; Run: 0
没有实例

4）创建PostgreSQL实例：
```
./bin/sdb_sql_ctl addinst myinst -D database/5432/
```

5）检查创建的实例状态
```
./sdb_sql_ctl status
```
6）如果未展示，启动pgsql实例
```
./bin/sdb_sql_ctl start myinst
```

5）检查创建的实例状态
```
./sdb_sql_ctl status
```

## 创建数据库及配置实例

1）创建 company 数据库实例；
```
./sdb_sql_ctl createdb company myinst
```

2）进入 PostgreSQL shell；
```
./bin/psql -p 5432 company
```

3）加载SequoiaDB连接驱动
```
create extension sdb_fdw;
```

4）配置与SequoiaDB连接参数
```
pgsdb=# CREATE SERVER sdb_server FOREIGN DATA WRAPPER sdb_fdw OPTIONS(address '127.0.0.1', service '11810', preferedinstance 'A', transaction 'off');
```
5）退出PostgreSQL shell；
```
\q
```

## 在 SequoiaDB 巨杉数据库引擎中创建域、集合空间、集合
1）通过 Linux 命令行进入 SequoiaDB Shell
```
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

## 集合数据操作
通过 SequoiaDB Shell 操作集合中数据。

#### 集合中插入数据
1）在 JSON 实例集合 company 中插入数据 ：
```javascript
db.company.employee.insert ({ "empno" : 1 , "ename" : "Georgi" , "age" : 48 }) ;
db.company.employee.insert ({ "empno" : 2 , "ename" : "Bezalel" , "age" : 21 }) ;
```
2）退出 SequoiaDB Shell；
```
quit ;
```

## 关联 PostgreSQL 实例关联 SequoiaDB 的集合空间与集合
1）进入 PostgreSQL Shell 命令行：
```
./bin/psql -p 5432 company
```
2）创建 company 数据库外表；
```
CREATE FOREIGN TABLE employee (
  empno integer, 
  ename text,
  age integer
) SERVER sdb_server 
  OPTIONS ( collectionspace 'company', collection 'employee', decimal 'on') ;
```

## 6 在 PostgreSQL 实例中对数据进行操作
1）更新表的统计信息
```
analyze employee ;
```
2）查询 employee 表中的数据；
```
select * from employee ;
```

3）写入数据
```
insert into employee values (3, 'Jack', 27)
```

4）更改数据
update employee set age = 28 where ename = 'Jack';

5）查看员工 Jack 的岁数是否更改为28
```
select * from employee where ename = 'Jack' ;
```


## 巨杉数据库引擎中的数据是否与 PostgreSQL 实例中数据一致
1）通过 Linux 命令行进入 SequoiaDB Shell
```
sdb
```
2）通过 javascript 语言连接协调节点，获取数据库连接；
```javascript
var db = new Sdb ("localhost",11810) ;
```
3）查询 employee 集合中的数据是否与PostgreSQL 实例操作的结果一致；

```
db.company.employee.find() ;
```

3）退出 SequoiaDB Shell；
```
quit ;
```

## 总结


