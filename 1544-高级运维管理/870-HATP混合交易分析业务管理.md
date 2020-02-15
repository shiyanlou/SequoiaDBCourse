---
show: step
version: 1.0
enable_checker: true
---

# HATP混合交易分析业务管理

## 课程介绍

本课程主要介绍Sequoiadb巨杉数据库的HTAP能力。通过连接不同的分区副本，实现OLTP与OLAP业务分离，进而提高整体性能。

**请点击右侧选择使用的实验环境**

**部署架构**

本课程中的SequoiaDB巨杉数据库的集群拓扑结构为三分区三副本。并已完成MySQL实例，SparkSQL的安装。

**实验环境**

课程使用的实验环境为Ubuntu Linux 16.04 64位版本。

## 切换用户及查看数据库信息

#### 切换到sdbadmin用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin。

```
su - sdbadmin
```

> Note:
>
> 用户 sdbadmin 的密码为 sdbadmin

#### 查看数据库状态

`sdblist  -t all -l -m local`

操作截图：

![870-1](https://doc.shiyanlou.com/courses/1544/1207281/4ea9a3ba18a9188dbb85a79a3d910f18)

这里可以从PRY列，列出了个节点是否主节点，若为主节点则为”Y“，若为从节点则为“N”。

## MySQL实例初始化

MySQL实例初始化后，用于处理OLTP类的业务。通过MySQL实例进行数据的增，删，改。

1）登录MySQL实例创建数据库

```
mysql -uroot -h127.0.0.1
create database company;
use company;
```

2）创建表

创建一张hash分区表。

```
create table company.order_info(order_id int AUTO_INCREMENT PRIMARY KEY,
product_type_code varchar(20),
price float(5,2))
ENGINE=sequoiadb 
COMMENT="sequoiadb:{ShardingKey:{'order_id':1},ShardingType:'hash',Partition:4096,AutoSplit:true}";
```

参数AutoSplit设置为true表示自动切分至所有数据节点上。

3）插入数据

运行insert.sh的shell脚本，通过MySQL实例插入数据。

```
#!/bin/bash
i=1
while [ $i -le 10 ]
do
a=`tr -dc "0-9" < /dev/urandom | head -c 1`
b=`tr -dc "1-9" < /dev/urandom | head -c 3`
mysql -uroot -h127.0.0.1 -e  "insert into company.order_info(product_type_code,price) values('$a',$b)"
i=`expr $i + 1`
done
```

## 巨杉数据库设置

1）登录SDB控制台并连接

```
db=new Sdb()
```

操作截图：

 ![870-2](https://doc.shiyanlou.com/courses/1544/1207281/c2759ecd14d1066a7c6fbe1757e10829)



2）修改SDB参数

通过修改参数instanceid，确定每个副本的编号。这个编号为稍后的SparkSQL创建表做准备。

关于参数的更多说明，请参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190643-edition_id-0](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190643-edition_id-0)

```
db.updateConf({instanceid:1},{GroupName:"group1",svcname:"11820"})
db.updateConf({instanceid:2},{GroupName:"group1",svcname:"21820"})
db.updateConf({instanceid:3},{GroupName:"group1",svcname:"31820"})
db.updateConf({instanceid:1},{GroupName:"group2",svcname:"11830"})
db.updateConf({instanceid:2},{GroupName:"group2",svcname:"21830"})
db.updateConf({instanceid:3},{GroupName:"group2",svcname:"31830"})
db.updateConf({instanceid:1},{GroupName:"group3",svcname:"11840"})
db.updateConf({instanceid:2},{GroupName:"group3",svcname:"21840"})
db.updateConf({instanceid:3},{GroupName:"group3",svcname:"31840"})
```

SDB数据库共有3个分区，分别是group1，group2，group3。每个分区有三个副本，一个主节点副本，两个从节点副本。通过上面的命令把三个副本分别标示为1，2，3。主节点的副本编号为1，从节点的副本编为2和3。

操作截图：

 ![870-5](https://doc.shiyanlou.com/courses/1544/1207281/d77d0622551d155b9a13b1a4c1d51bf1)

3）重启SDB数据库 

instanceid参数为重启后生效，修改完参数后，重启数据库。

```
sdbstop -t all
sdbstart -t all
```

操作截图：

 ![870-3](https://doc.shiyanlou.com/courses/1544/1207281/084fe6c8d5b31a5199582b76c8c70588)

 ![870-4](https://doc.shiyanlou.com/courses/1544/1207281/ce86694217fb24781a5759c3cdbf20b7)

4）查看参数修改状态

```
db.snapshot(13,{},{NodeName:null,instanceid:null})
```

此时，所有数据节点的instanceid均已修改完成。

操作截图：

 ![870-6](https://doc.shiyanlou.com/courses/1544/1207281/fbae38cccb352b93f8ec66fc7993ed97)



## SparkSQL设置

SparkSQL设置完成后，用于处理OLAP类的业务。主要负责处理聚合类查询或者同级业务。

这里我们把SaprkSQL中的表连接至从节点。

1）登录SparkSQL

```
/opt/spark/bin/beeline -u 'jdbc:hive2://localhost:10000' -n metauser -p metauser
```

2）在SparkSQL中创建表的结构

```
CREATE TABLE order_info (order_id int,
product_type_code string,
price int) USING com.sequoiadb.spark
OPTIONS (host 'localhost:11810',collectionspace 'company',
collection 'order_info',
preferedinstance 3);
```

host：指定SequoiaDB协调节点。

collectionspace：指定集合空间名称。

collection：指定集合的名称。

preferredinstance：指定副本编号，这里指定副本编号为3（上面指定的instanceid）。

更多的SparkSQL创建表参数说明，请参考如下链接：

[http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

3）数据的查询

此时SparkSQL从副本3 读取数据。

```
select * from order_info;
```

操作截图：

 ![870-7](https://doc.shiyanlou.com/courses/1544/1207281/f1904ffca785124225d2c09936512753)
