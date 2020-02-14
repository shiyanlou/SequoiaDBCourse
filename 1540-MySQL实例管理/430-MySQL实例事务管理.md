---
show: step
version: 1.0
enable_checker: true
---
# MySQL 实例事务管理


## 课程介绍

事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你既需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务。
本课程主要讲解 MySQL 事务的基本操作。

#### 请点击右侧选择使用的实验环境

#### 事务的 ACID 属性

- 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


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

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/03eb5c621476f2788a52a6ea755b23bd)

#### 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist 
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/cdc72e13c0eb5bedfbeb94c800c94f36)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤

#### 检查 MySQL 实例进程

查看 MySQL 数据库实例
```
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/92856e2e05fee65495cb876332cd34c6)

查看数据库实例进程
```
ps -elf | grep mysql
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/41b259ef9f2b7f16466b3d89606998c4)


## 事务提交

#### 登录到 MySQL 实例

登录 MySQL 实例
```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root -p
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/b667a6cc7f74c4b19d832efe32054996)

#### 执行事务提交操作

选择 company 数据库
```sql
USE company ;
```

开启事务操作
```sql
START TRANSACTION ;
```

执行SQL语句，主要包含插入与更新
```sql
INSERT INTO employee VALUES (10007,'1953-09-02','Georgi','Facello','M','1986-06-26') ;

INSERT INTO employee VALUES (10008,'1964-06-02','Bezalel','Simmel','F','1985-11-21') ;

INSERT INTO employee VALUES (10009,'1959-12-03','Parto','Bamford','M','1986-08-28') ;

UPDATE employee
SET first_name = 'Anneke_UPDATE', 
    last_name = 'Preusig_UPDATE', 
    hire_date = '2020-01-01'
WHERE emp_no = 10006 ;
```

执行事务提交操作
```sql
COMMIT ;
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/dec1dd362ea3dc685251563778fab9c2)

#### 事务提交操作的结果验证

查询数据， 验证事务提交的数据结果
```sql
SELECT * FROM employee WHERE emp_no IN (10006, 10007, 10008, 10009) ;
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/f78e6cb02f4a2adb0a8badacfca821db)

> 如操作截图显示，其中empo_no 为 10007，10008，10009 的记录插入到数据库；而 empo_no 为 10006 的记录更新成功


## 事务回滚

#### 执行事务提交操作

开启事务操作
```sql
START TRANSACTION ;
```

执行SQL语句，主要包含插入与更新
```sql
INSERT INTO employee VALUES (10010,'1953-09-02','Georgi','Facello','M','1986-06-26') ;

INSERT INTO employee VALUES (10011,'1964-06-02','Bezalel','Simmel','F','1985-11-21') ;

INSERT INTO employee VALUES (10012,'1959-12-03','Parto','Bamford','M','1986-08-28') ;

UPDATE employee
SET first_name = 'Anneke_ROLEBK', 
    last_name = 'Preusig_ROLEBK', 
    hire_date = '2020-01-01'
WHERE emp_no = 10006 ;
```

执行事务回滚操作
```sql
ROLLBACK ;
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/9e8cde81aa0230e22de427f8e2b61d29)


#### 事务回滚操作的结果验证

查询数据， 验证事务回滚后的数据结果
```sql
SELECT * FROM employee WHERE emp_no IN (10010, 10011, 10012, 10006) ;
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/a3ae2f29c1ed07dabef88e9fc9c4429a)

> 如操作截图显示，其中 empo_no 为 10010，10011，10012 的记录未插入到数据库；而 empo_no 为 10006 的记录也没有更新。


