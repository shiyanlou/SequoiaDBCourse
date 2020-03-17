---
show: step
version: 6.0
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎和 SparkSQL 实例环境中，使用 Beeline 客户端访问 SparkSQL 完成对数据的写入、查询操作，另外重点介绍如何使用 JDBC 接口访问 SparkSQL 进行应用开发。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中SequoiaSQL-SparkSQL 数据库实例包括 2 个 worker 节点，SequoiaDB 巨杉数据库包括 1 个引擎协调节点，1 个编目节点与 3 个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1539/1207281/0dee87fa137ac6f95d919e3236dfc617-0)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-SparkSQL 实例连接器均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-SparkSQL 实例的操作系统用户为 sdbadmin。
```shell
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/6cccf5951f048e01b4789f3c08483bb0-0)

## 查看节点启动列表

1）查看 SequoiaDB 巨杉数据库引擎节点列表；  

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/810c1187bb311b8a506bdb6731e1f73f-0)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤。
> 
>C: 编目节点，S：协调节点，D：数据节点

2）检查 SparkSQL 实例；  
执行 jps 查看进程是否启动：
```shell
jps
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/964e94f7729b4d411cb0c423d538fed2-0)

>Note:
>
>如果显示的进程少于上图中的数量，请稍等初始化完成并重试该步骤，环境初始化过程可能会有点慢，需要耐心等待一下。

## SequoiaDB 创建集合空间和集合

进入 SequoiaDB Shell，在 SequoiaDB 中创建集合空间 company，集合 employee，存储 SparkSQL 操作的数据。

1）使用 Linux 命令行进入 SequoiaDB Shell；
```shell
sdb
```

2）使用 javascript 语言连接协调节点；
```javascript
var db = new Sdb("localhost", 11810);
```

3）创建 company_domains 逻辑域；

```javascript
db.createDomain("company_domain", [ "group1", "group2", "group3" ], { AutoSplit: true } );
```

4）创建 company 集合空间；
```javascript
db.createCS("company", { Domain: "company_domain" } );
```

5）创建 employee 集合，并使用 _id 进行 hash 分区；
```javascript
db.company.createCL("employee", { "ShardingKey": { "_id": 1 }, "ShardingType": "hash", "ReplSize": -1, "Compressed": true, "CompressionType": "lzw", "AutoSplit": true, "EnsureShardingIndex": false });
```

6）退出 SequoiaDB Shell；
```javascript
quit;
```

## SparkSQL 创建数据库及映射表

在 SparkSQL 实例中创建数据库及数据表并与 SequoiaDB 数据库引擎中的集合空间和集合关联。

#### 创建数据库

1）使用 Beeline 客户端连接 SparkSQL 实例服务；

```shell
/opt/spark/bin/beeline -u 'jdbc:hive2://localhost:10000'
```

2）在 SparkSQL 实例中创建数据库 company，并切换至company库；

```sql
CREATE DATABASE company;
USE company;
```

#### 关联实例与数据库引擎中的集合

在 SparkSQL 实例中创建表并与 SequoiaDB 数据库存储引擎中的集合空间和集合关联；

```sql
CREATE TABLE employee 
(
empno      INT,
ename STRING,
age INT
) USING com.sequoiadb.spark OPTIONS ( host 'localhost:11810', collectionspace 'company', collection 'employee' );
```

从 SparkSQL 实例中创建视图、表及数据类型对应关系的详细说明请参考：
[SparkSQL 实例访问SequoiaDB数据库存储引擎](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)

## 在 Beeline 中进行数据操作
使用 SparkSQL 实例操作关联表中的数据。


1）在 SparkSQL 实例关联表 employee 中插入数据；
```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48);
INSERT INTO employee VALUES (10002, 'Bezalel', 21);
INSERT INTO employee VALUES (10003, 'Parto', 33);
INSERT INTO employee VALUES (10004, 'Chirstian', 40);
INSERT INTO employee VALUES (10005, 'Kyoichi', 23);
INSERT INTO employee VALUES (10006, 'Anneke', 19);
```

2）使用 SparkSQL 实例查询员工年龄为20至24之间的记录；

```sql
SELECT * FROM employee  WHERE age BETWEEN  20 AND 24;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/fe01b3e38585fe9fd13629c82189afcb-0)

3）退出 SparkSQL 的客户端；
```sql 
!quit
```
## 创建 JAVA 工程项目

SparkSQL 支持 JDBC 的访问方式，在使用前需要启动 Thriftserver 服务，在开发应用中通过 JDBC 访问 SparkSQL 时，需要在 Java 工程中加入 Thriftserver 依赖的库。

1）创建 JAVA 工程目录；

```shell
mkdir -p /home/sdbadmin/spark/lib
```

2）进入工程目录；

```shell
cd /home/sdbadmin/spark
```

3）拷贝 JDBC 接口需要的 jar 包；

```shell
cp /opt/spark/jars/commons-logging-1.1.3.jar ./lib
cp /opt/spark/jars/hadoop-common-2.7.3.jar ./lib
cp /opt/spark/jars/hive-exec-1.2.1.spark2.jar ./lib
cp /opt/spark/jars/hive-jdbc-1.2.1.spark2.jar ./lib
cp /opt/spark/jars/hive-metastore-1.2.1.spark2.jar ./lib
cp /opt/spark/jars/httpclient-4.5.6.jar ./lib
cp /opt/spark/jars/httpcore-4.4.10.jar ./lib
cp /opt/spark/jars/libthrift-0.9.3.jar ./lib
cp /opt/spark/jars/log4j-1.2.17.jar ./lib
cp /opt/spark/jars/slf4j-api-1.7.16.jar ./lib
cp /opt/spark/jars/slf4j-log4j12-1.7.16.jar ./lib
cp /opt/spark/jars/spark-network-common_2.11-2.4.4.jar ./lib
cp /opt/spark/jars/spark-hive-thriftserver_2.11-2.4.4.jar ./lib
```

4）复制以下代码到实验环境终端执行，生成通过 JDBC 接口操作 SparkSQL 数据的 Select.java 文件；
```shell
cat > /home/sdbadmin/spark/Select.java << EOF
package com.sequoiadb.spark;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Select {
    private static String driverName = "org.apache.hive.jdbc.HiveDriver";
    public static void main(String args[])
    {
        try {
            Class.forName(driverName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.exit(1);
        }
        String url = "jdbc:hive2://localhost:10000/company";
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(url, "sdbadmin", "");
        } catch (SQLException e) {
            e.printStackTrace();
        }
        System.out.println("Connection success!");
        try {
            System.out.println("--------- Get Records ---------");
            Statement st = conn.createStatement();
            String sql = " select * from employee";
            ResultSet rs = st.executeQuery(sql);
            while (rs.next()) {
                System.out.println(rs.getInt("empno") + "\t" + rs.getString("ename") + "\t" + rs.getInt("age"));
            }
            rs.close();
            st.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

EOF

```
5）查询是否已经生成 Select.java 文件；

```shell
ls -trl /home/sdbadmin/spark/Select.java
```


## 编译运行代码
上一小节已经创建了 JAVA 工程和代码并且拷贝了 JAVA 驱动，接下来我们对代码进行编译运行。

1）编译 Select.java 文件；

```shell
javac -d . Select.java
```

2）运行 Select 类代码，查询数据；

```shell
java -cp .:./lib/* com.sequoiadb.spark.Select
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ce14a3476a2b9cee89b001ed288dc84f-0)

## 总结

通过本课程，我们学习了通过 SparkSQL 操作 SequoiaDB 巨杉数据库中的数据，同时介绍了如何使用 JDBC 接口操作 SparkSQL 实例中的数据。

