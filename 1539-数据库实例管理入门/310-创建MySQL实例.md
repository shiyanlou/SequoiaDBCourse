---
show: step
version: 3.0
enable_checker: true
---


## 课程介绍
本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎的环境中，安装部署 SequoiaSQL-MySQL 实例，并进行简单的使用验证安装环境。

#### MySQL 实例简介
MySQL 是一款开源的关系型数据库管理系统，也是目前最流行的关系型数据库管理系统之一，支持标准的 SQL 语言。 SequoiaDB 支持创建 MySQL 实例，完全兼容 MySQL 语法和协议，用户可以使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他 MySQL 语法操作。

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点，1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


##  安装 SequoiaSQL-MySQL 实例程序
安装 SequoiaSQL-MySQL 实例程序需要 root 系统用户，程序已经提前放置在 /home/shiyanlou/sequoiadb-3.4 目录。

1）切换至 root 用户，在 `[sudo] password for shiyanlou:` 后输入当前用户的密码；

```shell
sudo su
```
> Note:
> 当前用户的密码在右侧工具栏 [ SSH 直连]

2）进入软件包放置目录；

```shell
cd /home/shiyanlou/sequoiadb-3.4
```

3）设置 SequoiaSQL-MySQL 实例程序权限为可执行；

```shell
chmod +x sequoiasql-mysql-3.4-linux_x86_64-installer.run  
```
4）安装 SequoiaSQL-MySQL 实例；

```shell
./sequoiasql-mysql-3.4-linux_x86_64-installer.run --mode text
```
安装步骤选择说明请参考（本示例安装仅使用默认选择）：
* [SequoiaSQL-MySQL 实例安装向导说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1521595270-edition_id-0)


## 创建 MySQL 实例

1）切换 sdbadmin 用户；

```shell
su - sdbadmin
```
> Note:
>
> sdbadmin 用户的密码为 `sdbadmin`

2）进入 SequoiaSQL-MySQL 实例安装目录；

```shell
cd /opt/sequoiasql/mysql
```

3）创建数据库实例；

```shell
bin/sdb_sql_ctl addinst myinst -D database/3306/
```

4）查看实例；

```shell
bin/sdb_sql_ctl status
```

操作截图：

![](https://doc.shiyanlou.com/courses/1539/1207281/afd8e3ad9fafd763b9fe9784426b2e03-0)

## 查看配置文件

实例数据目录下的配置文件 auto.cnf，在 [mysqld] 下添加/更改对应配置项。

1）配置文件位置；

```shell
ls -l /opt/sequoiasql/mysql/database/3306/auto.cnf
```

2）查看 auto.cnf 配置文件内容；

```shell
cat /opt/sequoiasql/mysql/database/3306/auto.cnf
```

Note:
>配置参数有三种修改方式： 
>- 使用工具 sdb_sql_ctl 修改配置，配置生效需重新启动 MySQL 服务。例如：
> bin/sdb_sql_ctl chconf myinst --sdb-auto-partition=OFF
> 
> - 修改实例数据目录下的配置文件 auto.cnf，在 [mysqld] 下添加/更改对应配置项，配置生效需重新启动 MySQL 服务。例如：
> sequoiadb_auto_partition=OFF
> 
> - 通过 MySQL 命令行修改，配置临时有效，当重启MySQL服务后配置将失效。例如：
> SET GLOBAL sequoiadb_auto_partition=OFF ;
>
> 详细配置参数说明：[MySQL 实例配置](http://doc.sequoiadb.com/cn/SequoiaDB-cat_id-1566551297-edition_id-304#引擎配置使用说明)

## 创建数据库及数据表

进入 MySQL shell，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，为接下来验证 MySQL 实例是否安装成功提供测试数据。

#### 登录 MySQL shell 

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

#### 创建数据库实例

```sql
CREATE DATABASE company ;
USE company ;
```

#### 创建数据表
在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。


1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

2）进行基本的数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

3）退出 MySQL Shell；

```shell
\q
```

## 存储引擎中查看数据
查看 SequoiaSQL-MySQL 实例中 employee 数据表在 SequoiaDB 数据库存储引擎中对应的分区表，并查看数据记录。

1）使用 Linux 命令行进去 SequoiaDB Shell；

```shell
sdb
```
2）使用javascript 语法连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost", 11810) ;
```

3）查看存储引擎中的集合信息

```javascript
db.list (SDB_LIST_COLLECTIONS) ;
```

4）查找 employee 中的数据，查看是否为 SequoiaSQL-MySQL 实例中插入的数据；

```javascript
db.company.employee.find () ;
```

4）向 employee 集合中插入数据；

```javascript
db.company.employee.insert ({ ename : "Ben" , age : 20 }) ;
```

5）退出 SequoiaDB Shell

```javascript
quit ;
```

#### 查询 MySQL 实例数据

验证 SequoiaDB Shell 中插入的数据能否在 SequoiaDB-MySQL 实例查询。

1）使用 MySQL Shell 连接 SequoiaSQL-MySQL 实例；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建数据库实例，并切换到该数据库；

```sql
USE company ;
```
3）查询 employee 表中数据，是否存在 ename 字段值 Ben 的数据；

```sql
SELECT * FROM employee WHERE ename = 'Ben' ;
```

4）退出 MySQL Shell；

```sql
\q
```

## 总结
本课程主要讲述 SequoiaSQL-MySQL 实例的安装部署，同时也创建了数据库和数据表进行测试。
