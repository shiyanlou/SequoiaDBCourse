## 二，MySQL实例简介步骤
MySQL 是一款开源的关系型数据库管理系统，也是目前最流行的关系型数据库管理系统之一，支持标准的 SQL 语言。 SequoiaDB 支持创建MySQL实例，完全兼容MySQL语法和协议，用户可以使用SQL语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他MySQL语法操作。SequoiaDB所支持的 MySQL 版本 MySQL 5.7.24+ 。

### 2.1 课程环境准备步骤
在属主机上启动docker 课程容器，并进入container。

docker run -it --privileged=true --name sdbtestfu -h sdb sdbinstance5

以下步骤在Container中执行。
root用户进入介质目录，检查安装介质及启动sdb进程
```
cd /sequoiadb/sequoiadb-3.2.4
ls -l 
```
查看MySQL实例安装介质：
sequoiasql-mysql-3.2.4-linux_x86_64-installer.run

启动巨杉数据库进程
```
su - sdbadmin
sdbstart -t all
sdblist -l
```

### 2.2 root用户安装MySQL实例程序
```
./sequoiasql-mysql-3.2.4-linux_x86_64-installer.run --mode text
```

使用缺省的安装选项，完成输出结果如下：
```
----------------------------------------------------------------------------
Please wait while Setup installs SequoiaSQL MySQL Server on your computer.

 Installing
 0% ______________ 50% ______________ 100%
 #########################################

----------------------------------------------------------------------------
Setup has finished installing SequoiaSQL MySQL Server on your computer.
```
### 2.3 创建MySQL实例
```
su - sdbadmin

cd /opt/sequoiasql/mysql/bin
```
查看MySQL实例情况
```
./sdb_sql_ctl status
```
预期输出：
INSTANCE   PID        SVCNAME    SQLDATA                                  SQLLOG                                  
Total: 0; Run: 0
没有实例

创建实例：
```
./sdb_sql_ctl addinst myinst -D database/3306/

Adding instance myinst ...
Start instance myinst ...
```
查看实例：
```
./sdb_sql_ctl status

INSTANCE   PID        SVCNAME    SQLDATA                                  SQLLOG                                  
myinst     1059       -          /opt/sequoiasql/mysql/bin/database/3306/ /opt/sequoiasql/mysql/myinst.log        
Total: 1; Run: 1
```
检查配置文件：
```
ls -l /opt/sequoiasql/mysql/bin/database/3306/auto.cnf
```
-rw-r-----. 1 sdbadmin sdbadmin_group 2084 Jan  6 06:46 /opt/sequoiasql/mysql/bin/database/3306/auto.cnf

编辑auto.cnf配置文件：
```
vim /opt/sequoiasql/mysql/bin/database/3306/auto.cnf
```

重要参数：

SequoiaDB addresses（包括巨杉数据库协调节点的主机名/ip地址，端口号；可以配置多个协调节点）

sequoiadb_conn_addr=localhost:11810

Replica size of write operations.（副本同步策略，sdb多副本的部署情况使用-1.）

sequoiadb_replica_size=1

### 2.4 MySQL实例操作
查看实例状态
```
./sdb_sql_ctl status
```
如果在运行，先停止实例
```
./sdb_sql_ctl stop myinst
```
启动mysql实例
```
./sdb_sql_ctl start myinst 

Check port is available ...

./sdb_sql_ctl: line 78: netstat: command not found

Starting instance myinst ...

ok (PID: 13162)
```
### 2.5 在MySQL实例中创建数据库

首页sdbadmin用户执行：
```
 mysql -h127.0.0.1 -uroot -p
```
设置root密码：

set password for 'root'@'localhost'='root';

检查现有数据库：
```
mysql> show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
退出mysql命令行
```
mysql> quit:
```

在sdb命令行中观察CS,CL情况：
```
sdbadmin@sdb:/opt/sequoiasql/mysql/bin$ sdb
```
Welcome to SequoiaDB shell!
help() for help, Ctrl+c or quit to exit
```
> db=new Sdb()

localhost:11810
Takes 0.003555s.
```
查看sequoiadb中的集合信息：
```
> db.list(4)

Return 0 row(s).
Takes 0.011117s.
```
退出sdb命令行：
```
> quit
```
在Sequoiadb中还没有集合空间/集合。

在MySQL实例中创建新数据库testdb：
```
mysql -h127.0.0.1 -uroot -p

mysql> create database testsdb;
```
Query OK, 1 row affected (0.00 sec)
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testsdb            |
+--------------------+
5 rows in set (0.01 sec)
```

### 2.6 在MySQL实例中创建测试表
在mysql实例配置中缺省使用sequoiadb存储引擎，所以在生成表的时候可以不用指定存储引擎。
```
mysql> use testsdb;
Database changed
mysql> create table testtab1(my_name varchar(40),my_age int,my_address varchar(50));
Query OK, 0 rows affected (1.57 sec)
mysql> quit;
```
在sdb中观察新建表情况：
```
sdbadmin@sdb:/opt/sequoiasql/mysql/bin$ sdb
Welcome to SequoiaDB shell!
help() for help, Ctrl+c or quit to exit
> db=new Sdb()
localhost:11810
Takes 0.001248s.
```
查看sdb中的集合空间，能够看到以mysql实例中的database testsdb命名的集合空间：
```
> db.list(SDB_LIST_COLLECTIONSPACES)
{
  "Name": "testsdb"
}
Return 1 row(s).
Takes 0.000929s.
```
查看sdb中的集合，能够看到以mysql实例中的database testsdb中创建的表testtab1：
```
> db.list(SDB_LIST_COLLECTIONS)
{
  "Name": "testsdb.testtab1"
}
Return 1 row(s).
Takes 0.000762s.
```
退出sdb命令行：
```
> quit
```
进入mysql命令行：
```
mysql -h127.0.0.1 -uroot -p
```
操作mysql实例中的表及数据：
```
mysql> use testsdb;
mysql> insert into testtab1 values("name1",30,"chengdu,sichuan");
Query OK, 1 row affected (0.03 sec)

mysql> insert into testtab1 values("name2",31,"chengdu1,sichuan");
Query OK, 1 row affected (0.00 sec)

mysql> insert into testtab1 values("name3",33,"chengdu3,sichuan");
Query OK, 1 row affected (0.00 sec)

mysql> insert into testtab1 values("name4",34,"chengdu3,sichuan");
Query OK, 1 row affected (0.00 sec)

mysql> select * from testtab1;
+---------+--------+------------------+
| my_name | my_age | my_address       |
+---------+--------+------------------+
| name1   |     30 | chengdu,sichuan  |
| name2   |     31 | chengdu1,sichuan |
| name3   |     33 | chengdu3,sichuan |
| name4   |     34 | chengdu3,sichuan |
+---------+--------+------------------+
4 rows in set (0.00 sec)
mysql> quit;
```

进入sdb命令行：
```
sdb
> db=new Sdb()
localhost:11810
Takes 0.001046s.
> db.list(SDB_LIST_COLLECTIONS)
{
  "Name": "testsdb.testtab1"
}
Return 1 row(s).
Takes 0.000744s
```
在sdb命令行中查看CL和数据，与从mysql中插入的数据相同：
```
> db.testsdb.testtab1.find()
{
  "_id": {
    "$oid": "5e12de3de9115266008959c0"
  },
  "my_name": "name1",
  "my_age": 30,
  "my_address": "chengdu,sichuan"
}
{
  "_id": {
    "$oid": "5e12de5ce9115266008959c1"
  },
  "my_name": "name2",
  "my_age": 31,
  "my_address": "chengdu1,sichuan"
}
{
  "_id": {
    "$oid": "5e12de6ae9115266008959c2"
  },
  "my_name": "name3",
  "my_age": 33,
  "my_address": "chengdu3,sichuan"
}
{
  "_id": {
    "$oid": "5e12de71e9115266008959c3"
  },
  "my_name": "name4",
  "my_age": 34,
  "my_address": "chengdu3,sichuan"
}
Return 4 row(s).
Takes 0.002042s.
```
在sdb中删除数据：
```
> db.testsdb.testtab1.remove({my_name:"name4"})
{
  "DeletedNum": 1
}
Takes 0.001023s.
> quit
```
在mysql命令行查看数据情况：
```
mysql -h127.0.0.1 -uroot -p
mysql> use testsdb;
mysql> select * from testtab1;
+---------+--------+------------------+
| my_name | my_age | my_address       |
+---------+--------+------------------+
| name1   |     30 | chengdu,sichuan  |
| name2   |     31 | chengdu1,sichuan |
| name3   |     33 | chengdu3,sichuan |
+---------+--------+------------------+
3 rows in set (0.00 sec)
```
数据与sdb中相同。

在MySQL中更新数据：
```
mysql> update testtab1 set my_age=50 where my_name='name3';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from testtab1;                             
+---------+--------+------------------+
| my_name | my_age | my_address       |
+---------+--------+------------------+
| name1   |     30 | chengdu,sichuan  |
| name2   |     31 | chengdu1,sichuan |
| name3   |     50 | chengdu3,sichuan |
+---------+--------+------------------+
3 rows in set (0.00 sec)
```
删除数据：
```
mysql> delete from testtab1 where my_name='name3';
Query OK, 1 row affected (0.00 sec)

mysql> select * from testtab1;                    
+---------+--------+------------------+
| my_name | my_age | my_address       |
+---------+--------+------------------+
| name1   |     30 | chengdu,sichuan  |
| name2   |     31 | chengdu1,sichuan |
+---------+--------+------------------+
2 rows in set (0.00 sec)

mysql> quit
```
在sdb中观察数据，数据与mysql中的相同：
```
sdb
> db=new Sdb()
localhost:11810
Takes 0.001140s.
> db.testsdb.testtab1.find()
{
  "_id": {
    "$oid": "5e1bdcabd628c3ea57ffff11"
  },
  "my_name": "name1",
  "my_age": 30,
  "my_address": "chengdu,sichuan"
}
{
  "_id": {
    "$oid": "5e1bdcb4d628c3ea57ffff12"
  },
  "my_name": "name2",
  "my_age": 31,
  "my_address": "chengdu1,sichuan"
}
Return 2 row(s).
Takes 0.005336s.
> quit
```
执行完成，退出docker
```
$exit
#exit
```
删除Container
```
docker rm sdbtestfu
```
