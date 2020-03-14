---
show: step
version: 13.0
enable_checker: true
---

# MySQL 实例事务管理

## 课程介绍

事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你既需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务。本课程主要讲解 MySQL 事务的基本操作。

#### 事务的 ACID 属性

- 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

本课程中 SequoiaDB 巨杉数据库的集群由一个 SQL 引擎和一组三分区单副本的巨杉数据库引擎组成；其中，SQL引擎包括 1 个 SequoiaSQL-MySQL 数据库实例节点，数据库引擎包括 1 个协调节点、1 个编目节点和 3 个数据节点。

#### 请点击右侧选择使用的实验环境

#### 环境架构：

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 巨杉数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


## 切换用户及查看数据库版本

切换至部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户并查看数据库引擎版本。

#### 切换到 sdbadmin 用户

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

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/6cccf5951f048e01b4789f3c08483bb0-0)

#### 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图:  

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/810c1187bb311b8a506bdb6731e1f73f-0)


>Note:
>
>如果显示的节点数量与预期不符，请稍等节点初始化完成并重试该步骤。

#### 检查 MySQL 实例进程

1）查看 MySQL 数据库实例；

```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/92856e2e05fee65495cb876332cd34c6)

2）查看数据库实例进程；

```shell
ps -elf | grep mysql
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/41b259ef9f2b7f16466b3d89606998c4)


## 创建数据库及数据表

进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，验证实例的创建及使用是否成功。

#### 登录 MySQL shell 

登录 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

#### 创建数据库

创建数据库；
```sql
CREATE DATABASE company;
USE company;
```

#### 创建数据表并初始化数据

在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。


1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee 
(
empno INT AUTO_INCREMENT PRIMARY KEY, 
ename VARCHAR(128), 
age INT
);
```

2）插入数据；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36);
INSERT INTO employee (ename, age) VALUES ("Alice", 21);
```

## 事务提交与回滚

SequoiaDB 巨杉数据库的 MySQL 数据库实例支持完整的事务操作能力，本小节将验证其基本的回滚与提交能力。

#### 执行事务提交操作

1）开启事务操作；

```sql
BEGIN;
```

2）执行SQL语句，包含写入与更新；
```sql
INSERT INTO employee (ename, age) VALUES ("Ben", 25);
UPDATE employee SET age = 22 WHERE ename = "Alice";
```

3）执行事务提交操作；

```sql
COMMIT;
```

#### 事务提交操作的结果验证

查询数据，验证事务提交的数据结果，是否写入与更新成功。

```sql
SELECT * FROM employee;
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/2f95a5b013de24775c07d49865a08b6c-0)

> 如操作截图显示，事务内的操作均被提交成功：雇员 Ben 的信息已经写入，并且 Alice 的年龄被更新成功。


#### 执行事务回滚操作

1）开启事务操作；

```sql
BEGIN;
```

2）执行SQL语句，主要包含插入与更新；

```sql
INSERT INTO employee (ename, age) VALUES ("Janey", 27);
UPDATE employee SET age = 26 WHERE ename = "Ben";
```

3）执行事务回滚操作；

```sql
ROLLBACK;
```

#### 事务回滚操作的结果验证

查询数据，验证事务回滚后的数据结果；

```sql
SELECT * FROM employee;
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/2f95a5b013de24775c07d49865a08b6c-0)

> 如操作截图显示，雇员 Janey 的信息未写入到数据库；而 Ben 的年龄也没有更新。


## MySQL 实例事务管理

MySQL 实例的事务是基于 SequoiaDB 巨杉数据库存储引擎的，如果需要 MySQL 实例支持事务，存储引擎也必须开启事务，本小节将讲解如何查看并关闭 MySQL 的事务功能，并对关闭事务功能后的 MySQL 实例进行验证。

1）查看 MySQL 是否已打开事务；

```sql
SHOW VARIABLES LIKE '%sequoiadb_use_transaction%';
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/30b998dd08806864e0421301f7808fd1-0)

图片中 sequoiadb_use_transaction 参数的值为 on ，说明事务功能已经打开。

2）退出 MySQL Shell；

```sql
\q
```

3）关闭 MySQL 的事务功能；
```shell
cat >> /opt/sequoiasql/mysql/database/3306/auto.cnf << EOF
sequoiadb_use_transaction = OFF
EOF
```

4）重启 myinst 实例；
```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl restart myinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/12d6eb9fb778e26d119531f98b71ebac-0)

5）登录 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```


6）切换到 company 数据库；

```sql
USE company;
```

7）开启事务操作；

```sql
BEGIN;
```

8）执行SQL语句，主要包含插入与更新；

```sql
INSERT INTO employee (ename, age) VALUES ("GOGO", 55);
```

9）执行事务回滚操作；

```sql
ROLLBACK;
```
10）查询 employee 表的数据,验证事务数据是否回滚；
```sql
SELECT * FROM employee;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/4e12985da3d81aeb1fa26606563658bd-0)

> Note：由图可见刚刚插入的数据，说明 MySQL 的事务功能已经关闭，数据库没有执行回滚操作

## 总结
本课程通过在 SequoiaSQL-MySQL 实例上创建数据库和数据表，并对其事务功能进行了验证。








