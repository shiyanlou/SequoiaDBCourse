---
show: step
version: 1.0
enable_checker: true
---

# snapshot性能监控


## 课程介绍

本课程主要介绍巨杉数据库中多维分区的规划。通过本课程了多维分区的概念，掌握域多维分区的使用方法。

**请点击右侧选择使用的实验环境**

**部署架构**

本课程中的SequoiaDB巨杉数据库的集群拓扑结构为三分区单副本。

**实验环境**

课程使用的实验环境为Ubuntu Linux 16.04 64位版本。

## MySQL实例创建数据库和表

1）登录MySQL实例

切换至sdbadmin用户，并登录MySQL实例

```
mysql -uroot -p127.0.0.1
```

2）创建数据库

```
create database compnay;
use company;
```

3）创建表

```
create table order_info
(order_id int AUTO_INCREMENT PRIMARY KEY,
product_name varchar(20),
price float(5,2))ENGINE=sequoiadb COMMENT="sequoiadb:{ShardingKey：{'order_id':1},ShardingType:'hash',Partition:4096,AutoSplit:true}";
```

这里直接在MySQL实例中创建表。ENGINE参数指定了使用巨杉数据库引擎sequoiadb

COMMENT参数指定了该表的一些配置项。这些配置项和createCL的配置项相同。

具体配置请参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190821-edition_id-304](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190821-edition_id-304)

4）插入数据

```
insert into order_info(product_name,price)values('pic',128.21);
```



## snapshot性能监控

本章节列了常用的snapshot，具体性能快照请参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1479173710-edition_id-304](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1479173710-edition_id-304)



1）执行shell查询脚本

select.sh用于模拟前端MySQL实例发起的查询。

```
#!/bin/bash
i=1
mysql -uroot -h127.0.0.1
while [ $i -le 600 ]
do
mysql -uroot -h127.0.0.1 -Dcompany -e "select * from order_info;"
sleep 1s
done
```

使用nohup命令使shell脚本后台运行

```
nohup ./select.sh &
```

2）登录SDB 控制台并创建连接

```
db=new Sdb()
```

3）查看SDB数据库中的会话

```
db.snapshot(SDB_SNAP_SESSIONS,{Source:{$regex:'MySQL.*'}},{})
```

{Source:{$regex:'MySQL.*'}使用正则匹配的方式过滤Source列中以MySQL为开头的会话信息。  

更多正则方式参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1464770442-edition_id-304](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1464770442-edition_id-304)

4）查看数据库状态

```
db.snapshot(SDB_SNAP_DATABASE)
```

如果数据库状态异常，ErrNodes列会列出异常信息。

操作截图：

 ![880-1](https://doc.shiyanlou.com/courses/1544/1207281/5c38a23657aa02b6fd6f92b8ddc4c590)

5）查看集合空间

```
db.snapshot(SDB_SNAP_COLLECTIONSPACES,{Name:'company'},{})
```

Name参数指定了需要查看的集合空间名，如果想查看所有集合空间，可以使用如下命令

```
db.snapshot(SDB_SNAP_COLLECTIONSPACES)
```

6）查看集合

```
db.snapshot(SDB_SNAP_COLLECTIONS,{Name:'company.order_info'},{})
```

Name参数指定了需要查看的集合，如果想查看所有集合，可以使用如下命令

```
db.snapshot(SDB_SNAP_COLLECTIONS)
```
