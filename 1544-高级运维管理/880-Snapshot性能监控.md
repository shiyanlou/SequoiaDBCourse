---
show: step
version: 1.0
enable_checker: true
---
# 15分钟演示巨杉数据库基本操作

## 课程介绍
在 SequoiaDB 巨杉数据库中，快照是一种得到系统当前状态的命令。
本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，学习查看快照。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

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

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/02fcaa58ac27e91688ead137fa748d6e)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤


## 创建数据库及数据表

进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，为接下来验证 MySQL 语法特性做准备。

#### 登录 MySQL shell 

```
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

2）数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

## SNAPSHOT 性能监控

本章节列了常用的snapshot，具体性能快照请参考如下链接：
* [SequoiaDB 快照列表说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1479173710-edition_id-0)

快照列表：
| 快照标示 | 快照类型   |
| ----- | --------- | 
| SDB_SNAP_CONTEXTS             | 上下文快照          | 
| SDB_SNAP_CONTEXTS_CURRENT     | 当前会话上下文快照  | 
| SDB_SNAP_SESSIONS             | 会话快照            | 
| SDB_SNAP_SESSIONS_CURRENT     | 当前会话快照        | 
| SDB_SNAP_COLLECTIONS          | 集合快照            | 
| SDB_SNAP_COLLECTIONSPACES     | 集合空间快照        | 
| SDB_SNAP_DATABASE             | 数据库快照          | 
| SDB_SNAP_SYSTEM               | 系统快照            | 
| SDB_SNAP_CATALOG              | 编目信息快照        | 
| SDB_SNAP_TRANSACTIONS         | 事务快照            | 
| SDB_SNAP_TRANSACTIONS_CURRENT | 当前事务快照        | 
| SDB_SNAP_ACCESSPLANS          | 访问计划缓存快照    | 
| SDB_SNAP_HEALTH               | 节点健康检测快照    | 
| SDB_SNAP_CONFIGS              | 配置快照            | 
| SDB_SNAP_SVCTASKS             | 服务任务快照        | 
| SDB_SNAP_SEQUENCES            | 序列快照            | 
















1）执行shell查询脚本

select.sh 用于模拟前端MySQL实例发起的查询。

```
#!/bin/bash
i=1
while [ $i -le 600 ]
do
mysql -uroot -h127.0.0.1 -Dcompany -e "select * from employee;"
i=$((${i}+1))
sleep 1s
done
```

使用nohup命令使shell脚本后台运行

```
nohup sh select.sh &
```

2）在 Linux 命令行中进入 SequoiaDB Shell 交互式界面；

```
sdb
```

3）使用 JavaScript 连接协调节点，并获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```


4）查看SDB数据库中的会话；

```javascript
db.snapshot (SDB_SNAP_SESSIONS,{ Source : { $regex:'MySQL.*' } } ) ;
```

>{Source:{$regex:'MySQL.*'}使用正则匹配的方式过滤Source列中以MySQL为开头的会话信息。  


4）查看数据库状态；

```javascript
db.snapshot (SDB_SNAP_DATABASE) ;
```

如果数据库状态异常，ErrNodes列会列出异常信息。

操作截图：

 ![880-1](https://doc.shiyanlou.com/courses/1544/1207281/5c38a23657aa02b6fd6f92b8ddc4c590)

5）查看集合空间快照；

```
db.snapshot (SDB_SNAP_COLLECTIONSPACES, { Name : 'company' } ) ;
```

Name参数指定了需要查看的集合空间名，如果想查看所有集合空间，可以使用如下命令

```
db.snapshot (SDB_SNAP_COLLECTIONSPACES) ;
```

6）查看集合；

```
db.snapshot (SDB_SNAP_COLLECTIONS, { Name : 'company.employee' } ) ;
```

Name参数指定了需要查看的集合，如果想查看所有集合，可以使用如下命令

```
db.snapshot (SDB_SNAP_COLLECTIONS) ;
```


## 总结

SequoiaDB 巨杉数据库提供多种快照类型，获取快照我们能够得到系统当前的状态，有利于快速分析问题和对性能进行监控。
