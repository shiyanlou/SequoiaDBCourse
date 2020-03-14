---
show: step
version: 6.2
enable_checker: true
---

# SNAPSHOT 性能监控

## 课程介绍

在 SequoiaDB 巨杉数据库中，快照是一种得到系统当前状态的命令，用于监视当前系统状态的方法，通过使用 snapshot 命令，我们可以自由选择监控某个节点的状态信息，从而得到应用的使用情况，进而判断如何对数据库信息进行维护，本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，学习如何正常使用快照查看我们对应的 SequoiaDB 数据库信息情况。

#### 部署架构

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 SequoiaSQL-MySQL 数据库实例节点、1 个引擎协调节点，1 个编目节点与3个数据节点。

![880-1](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

关于 SequoiaDB 巨杉数据库系统架构的详细信息，请参考如下链接：

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin。

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

![880-2](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist
```

操作截图：

![880-3](https://doc.shiyanlou.com/courses/1469/1207281/02fcaa58ac27e91688ead137fa748d6e)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤。
> 
>C: 编目节点，S：协调节点，D：数据节点

## 使用数据表并插入数据

进入 MySQL Shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库，为接下来验证 MySQL 语法特性做准备。

#### 登录 MySQL Shell 

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

#### 使用数据库

```sql
CREATE DATABASE company ;
USE company ;
```

#### 创建数据表

在 SequoiaSQL-MySQL 实例中创建的表将会默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。

1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (
    empno INT AUTO_INCREMENT PRIMARY KEY,
    ename VARCHAR(128),
    age INT
) ;
```


2）数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

3）查看数据情况；

```sql
SELECT * FROM employee ;
```

4）退出 MySQL Shell ；

```sql
\q
```

## SNAPSHOT 监控

本章节列了常用的快照类型，具体性能快照请参考如下链接：

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

1）模拟 MySQL 查询。编写 shell 查询脚本，脚本名为 select.sh，用于模拟前端 MySQL 实例发起的查询；

```shell
cat > select.sh <<EOF
#!/bin/bash
i=1
while [ \$i -le 600 ]
do
mysql -uroot -h127.0.0.1 -Dcompany -e "SELECT * FROM employee;"
i=\$((\${i}+1))
sleep 1s
done
EOF
```

2）使用 nohup 命令使 select.sh 脚本后台运行；

```shell
nohup sh select.sh &
```

3）在 Linux 命令行中进入 SequoiaDB Shell 交互式界面；

```shell
sdb
```

4）使用 JavaScript 连接协调节点，并获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```

5）查看SDB数据库中的会话；

```javascript
db.snapshot (SDB_SNAP_SESSIONS,{ Source : { $regex:'MySQL.*' } } ) ;
```

通过 LastOpType 字段可以知道当前会话最后一次操作类型，通过 TotalDataRead 和 TotalIndexRead 查看当前会话数据记录读和索引读情况,通过 TotalSelect 字段知道当前会话查询总选取记录数量 ，通过 UserCPU 和 SysCPU 了解当前会话 cpu 资源使用的，通过 LastOpInfo 字段可以查看到当前会话对集合操作的具体信息，而通过LastOpBegin 和 LastOpEnd 可以知道最后一次操作的耗时。

>Note:
>
>{ Source:{$regex:'MySQL.*'} 使用正则匹配的方式过滤 Source 列中以 MySQL 为开头的会话信息。

会话快照中查询出来的字段信息，详情请查看：
* [SequoiaDB 会话快照列表说明](http://doc.sequoiadb.com/cn/index-cat_id-1479173713-edition_id-304)

操作截图：

 ![880-4](https://doc.shiyanlou.com/courses/1544/1207281/50d1d98c8cb2175bff047df8b5ed2353-0)

 其中

6）查看数据库状态；

```javascript
db.snapshot (SDB_SNAP_DATABASE) ;
```

如果数据库状态异常，ErrNodes 字段会列出异常信息。

操作截图：

 ![880-4](https://doc.shiyanlou.com/courses/1544/1207281/5c38a23657aa02b6fd6f92b8ddc4c590)

7）查看集合空间快照；

```javascript
db.snapshot (SDB_SNAP_COLLECTIONSPACES, { Name : 'company' } ) ;
```

Name 参数指定了需要查看的集合空间名，如果想查看所有集合空间，可以使用如下命令：

```javascript
db.snapshot (SDB_SNAP_COLLECTIONSPACES) ;
```

8）查看集合；

```javascript
db.snapshot (SDB_SNAP_COLLECTIONS, { Name : 'company.employee' } ) ;
```

Name 参数指定了需要查看的集合，如果想查看所有集合，可以使用如下命令:

```javascript
db.snapshot (SDB_SNAP_COLLECTIONS) ;
```

9）关闭 db 数据库连接；
```javascript
db.close();
```

10）退出 SequoiaDB Shell ；

```javascript
quit ;
```

10）若查询集合快照信息输出过大，可以把查询结果输出到一个文件中 ；

```shell
/opt/sequoiadb/bin/sdb 'var db=new Sdb("localhost", 11810) ;'
/opt/sequoiadb/bin/sdb 'db.snapshot (SDB_SNAP_COLLECTIONS, { Name : "company.employee" } ) ;' > /home/sdbadmin/snap_collection.log
```


## 总结

SequoiaDB 巨杉数据库提供了多种快照类型，以供我们能够高效准确的定位系统当前的健康状态，判断是否存在某些相应的系统问题，有利于我们快速分析系统问题并作出判断以及对系统的性能进行监控及早预防可能存在的问题。
