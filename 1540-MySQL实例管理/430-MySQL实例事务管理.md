---
show: step
version: 2.0
enable_checker: true
---
# MySQL 实例事务管理


## 课程介绍

事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你既需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务。本课程主要讲解 MySQL 事务的基本操作。

#### 请点击右侧选择使用的实验环境

#### 事务的 ACID 属性

- 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 巨杉数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin：

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

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

#### 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图:  

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/02fcaa58ac27e91688ead137fa748d6e)


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
CREATE DATABASE company ;
USE company ;
```

#### 创建数据表并初始化数据

在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。


1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

2）插入数据；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 21) ;
```

## MySQL 实例事务管理
MySQL 实例的事务是基于 SequoiaDB 巨杉数据库存储引擎的，如果需要 MySQL 实例支持事务，存储引擎也必须开启事务。本课程环境中  SequoiaDB 巨杉数据库存储引擎使用的是默认值及开启事务，隔离级别为读未提交；下面对 MySQL 实例自身事务控制参数进行说明。

1）查看是否已打开事务；
```sql
SHOW VARIABLES LIKE '%sequoiadb_use_transaction%' ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/30b998dd08806864e0421301f7808fd1-0)

图片中 sequoiadb_use_transaction 参数的值为 on ，说明事务功能已经打开。

> Note:
> - SequoiaSQL-MySQL 的事务开启关系需要修改相应数据库实例下的 auto.cnf 文件的 sequoiadb_use_transaction 参数信息，然后重启实例。
> - sequoiadb_use_transaction 参数默认值为 on , 在业务无需事务功能时，我们可以将它设成 OFF，从而节省不必要的开销。

2) 查看事务隔离级别配置内容；
```sql
SHOW VARIABLES LIKE '%transaction_isolation%' ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/2b74b78aca9089c7d6125b2c1020953e-0)

图片中 transaction_isolation 参数的值为 REPEATABLE-READ ，说明事务隔离级别为读未提交。

>Note:
>- 目前版本的 SequoiaDB 支持三种数据库隔离级别，分别为 RU，RC，RS ；

## 事务提交

SequoiaDB 巨杉数据库的 MySQL 数据库实例支持完整的事务操作能力，本小节将验证其基本的回滚与提交能力。

#### 执行事务提交操作

1）开启事务操作；

```sql
BEGIN ;
```

2）执行SQL语句，包含写入与更新；
```sql
INSERT INTO employee (ename, age) VALUES ("Ben", 25) ;
UPDATE employee SET age = 22 WHERE ename = "Alice" ;
```

3）执行事务提交操作；

```sql
COMMIT ;
```

#### 事务提交操作的结果验证

查询数据，验证事务提交的数据结果，是否写入与更新成功。

```sql
SELECT * FROM employee ;
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/2f95a5b013de24775c07d49865a08b6c-0)



> 如操作截图显示，事务内的操作均被提交成功：雇员 Ben 的信息已经写入，并且 Alice 的年龄被更新成功。


## 事务回滚

#### 执行事务回滚操作

1）开启事务操作；

```sql
BEGIN ;
```

2）执行SQL语句，主要包含插入与更新；

```sql
INSERT INTO employee (ename, age) VALUES ("Janey", 27) ;
UPDATE employee SET age = 26 WHERE ename = "Ben" ;
```

3）执行事务回滚操作；

```sql
ROLLBACK ;
```

#### 事务回滚操作的结果验证

查询数据，验证事务回滚后的数据结果；

```sql
SELECT * FROM employee ;
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/2f95a5b013de24775c07d49865a08b6c-0)

> 如操作截图显示，雇员 Janey 的信息未写入到数据库；而 Ben 的年龄也没有更新。


## 总结
本课程通过在 SequoiaSQL-MySQL 实例上创建数据库和数据表，并对其事务进行了验证。


