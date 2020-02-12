---
show: step
version: 1.0
enable_checker: true
---
# MySQL 实例创建



## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，对 MySQL 进行实例创建，并向数据表中写入一定量数据。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据分区，9个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/bcf1c4b3404f34cb22cd48494c10135c)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

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

#### 创建数据库及数据表

进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，为接下来验证 MySQL 语法特性做准备。

#### 登录 MySQL Shell 

```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/b667a6cc7f74c4b19d832efe32054996)



## 创建 MySQL 实例和数据表

#### 创建数据库实例

在 MySQL 实例中创建新数据库company
```sql
CREATE DATABASE company ;
```

查看 MySQL 实例中的数据库：
```sql
SHOW DATABASES ;
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/736645861ddbc077ce588b4ab35f3ebd)


#### 创建分区表

在 MySQL 实例 company 数据库中创建分区表 employee
```sql
USE company ;
CREATE TABLE employee (
    emp_no      INT             NOT NULL,
    birth_date  DATE            NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    last_name   VARCHAR(16)     NOT NULL,
    gender      ENUM ('M','F')  NOT NULL,    
    hire_date   DATE            NOT NULL,
    PRIMARY KEY (emp_no)
)
ENGINE=sequoiadb COMMENT="雇员表, sequoiadb:{ table_options: { ShardingKey: { 'emp_no': 1 }, ShardingType: 'hash','Compressed':true,'CompressionType':'lzw','AutoSplit':true,'EnsureShardingIndex':false } }";
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/a08a299c8b2e94efa0b855bf18e7a0f0)

查看数据库引擎
```sql
SHOW ENGINES ;
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/d2caef2b1c019578ea0f2d211678ea01)

查看 company 数据库中创建分区表 employee
```sql
SHOW CREATE TABLE employee;
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/8fd64f8113c906f81a437b044d1262e2)



## 数据表中写入数据

#### 分区表中插入数据
在分区表 employee 中插入数据：
```sql
INSERT INTO employee VALUES (10001,'1953-09-02','Georgi','Facello','M','1986-06-26');

INSERT INTO employee VALUES (10002,'1964-06-02','Bezalel','Simmel','F','1985-11-21');

INSERT INTO employee VALUES (10003,'1959-12-03','Parto','Bamford','M','1986-08-28');

INSERT INTO employee VALUES (10004,'1954-05-01','Chirstian','Koblick','M','1986-12-01');

INSERT INTO employee VALUES (10005,'1955-01-21','Kyoichi','Maliniak','M','1989-09-12');

INSERT INTO employee VALUES (10006,'1953-04-20','Anneke','Preusig','F','1989-06-02');
```

操作截图:  
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/dd2fb29c7602c86ee9c5c382c9c1f377)



#### 查询分区表 employee 中的数据：
查看总的数据
```sql
SELECT * FROM employee ;
```

预期输出:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/baba6318103bed34547796ab1b598a05)


