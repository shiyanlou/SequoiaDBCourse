---
show: step
version: 1.0
enable_checker: true
---



## 课程介绍
本实验基于Sequoiadb数据库提供的Docker镜像，能够一步一步的带领你在linux环境中部署巨杉数据库的MySQL实例。

#### MySQL实例简介
MySQL 是一款开源的关系型数据库管理系统，也是目前最流行的关系型数据库管理系统之一，支持标准的 SQL 语言。 SequoiaDB 支持创建MySQL实例，完全兼容MySQL语法和协议，用户可以使用SQL语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他MySQL语法操作。SequoiaDB所支持的 MySQL 版本 MySQL 5.7.24+ 。

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


##  root用户安装MySQL实例程序
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
chmod +x sequoiasql-mysql-3.4-linux_x86_64-installer.run  
```
6）安装 SequoiaSQL-MySQL 实例；
```
./sequoiasql-mysql-3.4-linux_x86_64-installer.run --mode text
```
安装步骤选择说明请参考：
* [SequoiaSQL-MySQL 实例安装向导说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1521595270-edition_id-0)

## 4 创建MySQL实例

1）切换 sdbadmin 用户；
```
su - sdbadmin
```
2）进入 SequoiaSQL-MySQL 实例安装目录；
```
cd /opt/sequoiasql/mysql
```

3）查看MySQL实例情况；
```
./bin/sdb_sql_ctl status
```

预期输出：
INSTANCE   PID        SVCNAME    SQLDATA                                  SQLLOG                                  
Total: 0; Run: 0
没有实例

4）创建数据库实例；
```
./bin/sdb_sql_ctl addinst myinst -D database/3306/

Adding instance myinst ...
Start instance myinst ...
```
查看实例：
```
./bin/sdb_sql_ctl status
```
```
INSTANCE   PID        SVCNAME    SQLDATA                                  SQLLOG                                  
myinst     1059       -          /opt/sequoiasql/mysql/bin/database/3306/ /opt/sequoiasql/mysql/myinst.log        
Total: 1; Run: 1
```
## 配置文件讲解
1）配置文件位置
```
ls -l /opt/sequoiasql/mysql/database/3306/auto.cnf
```

2）查看 auto.cnf 配置文件内容；
```
cat /opt/sequoiasql/mysql/database/3306/auto.cnf
```
重要参数：

- sequoiadb_conn_addr : SequoiaDB addresses（包括巨杉数据库协调节点的主机名/ip地址，端口号；可以配置多个协调节点）

- sequoiadb_replica_size : Replica size of write operations.（副本同步策略，sdb多副本的部署情况使用-1.）


## 5 MySQL实例操作
查看实例状态
```
./bin/sdb_sql_ctl status
```
如果在运行，先停止实例
```
./bin/sdb_sql_ctl stop myinst
```
启动mysql实例
```
./sdb_sql_ctl start myinst 
```
## 6 在MySQL实例中创建数据库

1）使用 MySQL Shell 连接 SequoiaSQL-MySQL 实例；
```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```
2）检查现有数据库；
```
show databases ;
```

3）创建数据库实例，并切换到该数据库；
```
create database company ;
use company;
```
4）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

5）进行基本的数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

6）退出 MySQL Shell；
```shell
\q
```

## 在SequoiaDB 巨杉数据库引擎中查看存储情况
1）使用 Linux 命令行进去 SequoiaDB Shell；
```
sdb
```
2）使用javascript 语法连接协调节点，获取数据库连接；
```
var db=new Sdb("localhost", 11810) ;
```

3）查看sequoiadb中的集合信息
```
db.list(SDB_LIST_COLLECTIONS) ;
```
操作截图：
4）查找 employee 中的数据，查看是否为 SequoiaDB-MySQL 实例中插入的数据；

```
db.company.employee.find() ;
```

4）向 employee 集合中插入数据；
```
db.company.employee.insert({ ename:"Ben" , age:20 }) ;
```

5）退出 SequoiaDB Shell
```
quit ;
```

## 查询 SequoiaDB Shell 中插入的数据能否在 SequoiaDB-MySQL 实例查询


1）使用 MySQL Shell 连接 SequoiaSQL-MySQL 实例；

```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建数据库实例，并切换到该数据库；
```
use company;
```
3）查询 employee 表中数据，是否存在 ename 字段值 Ben 的数据；

```
select * from employee where ename = 'Ben' ;
```

4）退出 MySQL Shell；
```shell
\q
```

## 总结
本课程主要讲述 SequoiaSQL-MySQL 实例的安装部署，同时也创建了数据库和数据表进行测试。
