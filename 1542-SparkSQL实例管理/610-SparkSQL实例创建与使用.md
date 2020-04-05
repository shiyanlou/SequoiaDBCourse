---
show: step
version: 2.456
enable_checker: true 
---


# SparkSQL 实例创建与使用



## 课程介绍




本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，进行 SparkSQL 实例的安装部署并启动 Spark Thrift Server 服务使用 Beeline 客户端进行数据操作。

#### SparkSQL 简介

SparkSQL 是 Spark 产品中一个组成部分，SQL 的执行引擎使用 Spark 的 RDD 和 Dataframe 实现。目前 SparkSQL 已经可以完整运行 TPC-DS99 测试，标志着 SparkSQL 在数据分析和数据处理场景上技术进一步成熟。SequoiaDB 巨杉数据库为 Spark 开发了 SequoiaDB for Spark 的连接器，让 Spark 支持从 SequoiaDB 中并发获取数据，再完成相应的数据计算。

#### Spark Thrift Server 介绍

Spark Thrift Server 是 Spark 社区基于 HiveServer2 实现的一个 Thrift 服务，旨在无缝兼容 HiveServer2。

Spark Thrift Server 的接口和协议都和 HiveServer2 完全一致，因此部署好 Spark Thrift Server 后，可以直接使用 hive 的 beeline 客户端访问 Spark Thrift Server 执行相关语句。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 SparkSQL 实例节点， 1 个引擎协调节点， 1 个编目节点与 3 个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/50d088eb3f655c3058e4ee9ea6a29446-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本；SequoiaDB 巨杉数据库引擎、SequoiaSQL-MySQL 实例和 SequoiaDB-Spark 连接组件均为 3.4 版本；SparkSQL 版本为 2.4.4；JDK 版本为 openjdk1.8。

## 切换用户及查看数据库版本

切换到系统用户 sdbadmin，并查看 SequoiaDB 巨杉数据库引擎的版本。

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin。

```shell
su - sdbadmin
```

>Note:
>
>用户 sdbadmin 的密码为 `sdbadmin` 。

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/6cccf5951f048e01b4789f3c08483bb0-0)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

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


## 安装 Spark 实例

下面开始安装 Spark 实例，并对 Spark 实例进行必要的配置。

#### 解压 Spark 安装包

1）检查 Spark 安装包；

```shell
ls -trl /home/sdbadmin/soft/
```

操作截图：

![1542-610-1](https://doc.shiyanlou.com/courses/1542/1207281/bb4c027c1c181b51b6b426634b031b90-0)


2）解压 Spark 安装包；

```shell
tar -zxf /home/sdbadmin/soft/spark-2.4.4-bin-hadoop2.7.tar.gz -C /opt
```

#### 添加驱动包

1）拷贝 SequoiaDB for Spark 的连接器到 Spark 的 jars 目录下；

```shell
cp /opt/sequoiadb/spark/spark-sequoiadb_2.11-3.4.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

2）拷贝 MySQL 驱动到 Spark 的 jars 目录下；

```shell
cp /home/sdbadmin/soft/mysql-jdbc.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

3）拷贝 SequoiaDB 的 JAVA 驱动到 Spark 的 jars 目录下；

```shell
cp /opt/sequoiadb/java/sequoiadb-driver-3.4.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

#### 设置免密
1）执行 ssh-keygen 生成公钥和密钥，执行后连续回车即可；

```shell
ssh-keygen -t rsa
```

2）执行 ssh-copy-id，把公钥拷贝到本机的 sdbadmin 用户；

```shell
ssh-copy-id  sdbadmin@sdbserver1
```

>
>Note:
>
> sdbadmin 的密码是：`sdbadmin` 。
> 单机不需要拷贝到其它服务器上，如果是多机部署，需要配置所有服务器的互相关系。

#### 配置 Spark

1）设置 spark-env.sh；

从模板中复制一份 spark-env.sh 脚本：

```shell
cp /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh.template /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh
```

在 spark-env.sh 中设置 WORKER 的数量和 MASTER 的 IP；

```shell
echo "SPARK_WORKER_INSTANCES=2" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh
echo "SPARK_MASTER_IP=127.0.0.1" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh
```

2）复制以下代码到实验环境终端执行，用于创建设置元数据信息的数据库配置文件 hive-site.xml；

```shell
cat > /opt/spark-2.4.4-bin-hadoop2.7/conf/hive-site.xml << EOF
<configuration>
   <property>
     <name>hive.metastore.schema.verification</name>
     <value>false</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://localhost:3306/metastore</value>
      <description>JDBC connect string for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>metauser</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>metauser</value>
   </property>
   <property>
      <name>datanucleus.autoCreateSchema</name>
      <value>true</value>
      <description>creates necessary schema on a startup if one doesn't exist. set this to false, after creating it once</description>
   </property>
</configuration>
EOF
```

#### 配置 Spark 元数据库

1）使用 Linux 命令进入 MySQL Shell；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建 metauser 用户；

```sql
CREATE USER 'metauser'@'%' IDENTIFIED BY 'metauser';
```

操作截图：

![1542-610-3](https://doc.shiyanlou.com/courses/1538/1207281/dadb2110e578da42604ef2c2fe755b0a-0)

3）给 metauser 用户授权；

```sql
GRANT ALL ON *.* TO 'metauser'@'%';
```

操作截图：

![1542-610-4](https://doc.shiyanlou.com/courses/1538/1207281/be92979fc0c48b4df2524cb0567173f5-0)

4）创建 Spark 元数据库；

```sql
CREATE DATABASE metastore CHARACTER SET 'latin1' COLLATE 'latin1_bin';
```

操作截图：

![1542-610-5](https://doc.shiyanlou.com/courses/1538/1207281/402f6dd5278c7c5bed9bffc9f2a7169b-0)

5）刷新权限；

```sql
FLUSH PRIVILEGES;
```

操作截图：

![1542-610-6](https://doc.shiyanlou.com/courses/1538/1207281/a9d2c03cfd24eccc58195dae6d62afed-0)

6） 退出 MySQL Shell；

```shell
\q
```

## 启动 Spark 服务

本课程演示如何使用 Spark Thrift Server 服务连接 Spark 进行数据操作，所以需要启动 Spark 和 Spark Thrift Server 服务。

1） 启动 Spark；

```shell
/opt/spark-2.4.4-bin-hadoop2.7/sbin/start-all.sh
```

操作截图：

![1542-610-7](https://doc.shiyanlou.com/courses/1538/1207281/31c9e3da7b150cd27fc3604ea891ba31-0)

2）启动 thriftserver 服务；

```shell
/opt/spark-2.4.4-bin-hadoop2.7/sbin/start-thriftserver.sh
```

操作截图：

![1542-610-8](https://doc.shiyanlou.com/courses/1543/1207281/4575f232fdabedfc6054e9c59a30cb8d-0)

3） 检查进程启动状态；

```shell
jps
```

4） 检查端口监听状态；

```shell
netstat -anp | grep 10000
```

操作截图：

![1542-610-9](https://doc.shiyanlou.com/courses/1538/1207281/c59b79202d81658745530ab4abf754ee-0)

>
>Note:
>
> 本实验环境性能较低，启动 Spark 的耗时较长，请耐心等待 10000 端口的监听状态；如截图所示，此时 10000 端口监听成功即可继续执行后续操作。

## 在 SequoiaDB 建立集合空间和集合

进入 SequoiaDB Shell，在 SequoiaDB 巨杉数据库引擎中创建 company 集合空间和 employee 集合。

1）使用 Linux 命令进入 SequoiaDB Shell 命令行；

```shell
sdb
```

2）使用 JavaScript 语法，连接协调节点，获取数据库连接；

```javascript
var db = new Sdb("localhost", 11810);
```

2）创建 company_domain 逻辑域；

```javascript
db.createDomain("company_domain", [ "group1", "group2", "group3" ], { AutoSplit: true } );
```

3）创建 company 集合空间；

```javascript
db.createCS("company", { Domain: "company_domain" } );
```

4）创建 employee 集合；

```javascript
db.company.createCL("employee", { "ShardingKey": { "_id": 1 }, "ShardingType": "hash", "ReplSize": -1, "Compressed": true, "CompressionType": "lzw", "AutoSplit": true, "EnsureShardingIndex": false } );
```

5）退出 SequoiaDB Shell；

```shell
quit;
```

操作截图：

![1542-610-10](https://doc.shiyanlou.com/courses/1542/1207281/8a986975b479eecf299fb94eeaeb682f-0)

## 在 SparkSQL 关联集合空间和集合

SparkSQL 通过 Spark-SequoiaDB 连接组件关联 SequoiaDB 的集合空间和集合，将 SequoiaDB 巨杉数据库引擎作为 SparkSQL 的数据源进行相应的数据计算。

#### SparkSQL 与 SequoiaDB 的集合空间和集合关联

1）使用 Beeline 客户端工具连接至 thriftserver 服务；

```shell
/opt/spark-2.4.4-bin-hadoop2.7/bin/beeline -u 'jdbc:hive2://localhost:10000'
```

2）创建并切换至 company 数据库；

```sql
CREATE DATABASE company;
USE company;
```

3）创建 employee 表；

创建 employee 表，并且与 SequoiaDB 中的集合 employee 进行关联：

```sql
CREATE TABLE employee 
(
empno  INT,
ename  STRING,
age    INT
) 
USING com.sequoiadb.spark OPTIONS 
( 
host 'localhost:11810', 
collectionspace 'company', 
collection 'employee'
);
```

>Note:
>
> 当前环境未开启鉴权，因此忽略了 username 和 password 参数。

操作截图：

![1542-610-11](https://doc.shiyanlou.com/courses/1542/1207281/c6ed22f6000bb73cf874c187c1be79b2-0)

#### SequoiaDB-SparkSQL 建表语法说明

在 SparkSQL 中关联 SequoiaDB 集合空间和集合的 SQL 语法如下；

```txet
CREATE <[TEMPORARY] TABLE | TEMPORARY VIEW> <tableName> [(SCHEMA)]
USING com.sequoiadb.spark OPTIONS (<option>, <option>, ...);
```

语法说明：

- TEMPORARY 表示为临时表或视图，只在创建表或视图的会话中有效，会话退出后自动删除。

- 表名后紧跟的 SCHEMA 可不填，连接器会自动生成。自动生成的 SCHEMA 字段顺序与集合中记录的顺序不一致，因此如果对 SCHEMA 的字段顺序有要求，应该显式定义 SCHEMA。

- OPTIONS 为参数列表，参数是键和值都为字符串类型的键值对，其中值的前后需要有单引号，多个参数之间用逗号分隔。

#### SequoiaDB-SparkSQL 建表参数说明

下面是部分常用的 SequoiaDB-SparkSQL 建表参数说明，完整的建表参数请参考 [SequoiaDB-SparkSQL 参数说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)。

+ host ：SequoiaDB 协调节点/独立节点地址，多个地址以 “,” 分隔。例如：“server1:11810, server2:11810”。
+ collectionspace ：集合空间名称。
+ collection ：集合名称（不包含集合空间名称）。
+ username ：数据库用户名。
+ passwordtype : 密码类型，取值为“cleartext”或“file”，分别表示明文密码和文件密钥。
+ password ：数据库用户名对应的用户密码。
+ preferredinstance ：指定分区优先选择的节点实例。

## 在 Beeline 中进行数据操作

在 Beeline 客户端中对 SequoiaDB 巨杉数据库的集合进行数据写入、查询操作。

1）写入数据；

```sql
INSERT INTO employee VALUES ( 10001, 'Georgi', 48 );
INSERT INTO employee VALUES ( 10002, 'Bezalel', 21 );
INSERT INTO employee VALUES ( 10003, 'Parto', 33 );
INSERT INTO employee VALUES ( 10004, 'Chirstian', 40 );
```

![1542-610-12](https://doc.shiyanlou.com/courses/1543/1207281/6d7c7a5d31b09c8bce451aa1c5a32a4d-0)

2）进行数据查询；

```sql
SELECT * FROM employee;
```

操作截图：

![1542-610-13](https://doc.shiyanlou.com/courses/1543/1207281/41138f5eb03e5749b14c242e89cae8df-0)

## 通过连接器自动生成 Schema 创建表

SequoiaDB-SparkSQL 支持通过连接器自动生成 SCHEMA 来创建关联表，这样可以在建表时不指定 SCHEMA 信息。

1）通过连接器自动生成 SCHEMA 来创建 employee_auto_schema 表；

```sql
CREATE TABLE employee_auto_schema USING com.sequoiadb.spark OPTIONS 
(
host 'localhost:11810',
collectionspace 'company',
collection 'employee'
);
```

>Note:
>
>通过连接器自动生成 SCHEMA，要求在建表时 SequoiaDB 的关联集合中就已经存在数据记录。

2）查看表 employee_auto_schema 的结构信息；

```sql
DESC employee_auto_schema;
```
操作截图：

![1542-610-14](https://doc.shiyanlou.com/courses/1542/1207281/f5d0ae41a071828789c2dcd7cfbb5896-0)

3）查询 employee_auto_schema 的数据记录；

```SQL
SELECT * FROM employee_auto_schema;
```

>Note:
>
>SparkSQL 表 employee 和 employee_auto_schema 关联的都是 SequoiaDB 中的集合 company.employee，所以这两张 SparkSQL 表的对应数据是完全一致的。



## 通过 SQL 结果集创建表

SequoiaDB-SparkSQL 支持 `CREATE TABLE ... AS SELECT ...` 语法，通过 SQL 结果集创建新表。

#### 通过 SQL 结果集创建表

1）通过已有表 employee 创建表 employee_bak，并将表中的数据存放到指定域和集合空间中；

```sql
CREATE TABLE employee_bak USING com.sequoiadb.spark OPTIONS 
(
host 'localhost:11810',
domain 'company_domain',
collectionspace 'company',
collection 'employee_bak',
autosplit true,
shardingkey '{_id:1}',
shardingtype 'hash',
compressiontype 'lzw'
) AS SELECT * FROM employee;
```
操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/07b1ab4881eb54b734d27ff6105940f3-0)

2）查看 employee_bak 表中的数据；

```sql
SELECT * FROM employee_bak;
```



3）退出 Beeline Shell；

```shell
!quit
```

#### 使用 `CREATE TABLE ... AS SELECT ...` 语法建表参数说明

下面是部分 SequoiaDB-SparkSQL 使用 `CREATE TABLE ... AS SELECT ...` 语法建表的参数说明，完整的建表参数请参考 [SequoiaDB-SparkSQL 参数说明](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1432190712-edition_id-304)。

+ host ：SequoiaDB 协调节点/独立节点地址，多个地址以 “,” 分隔。例如：“server1:11810, server2:11810”。
+ domain ：创建集合空间时指定所属域。如果集合空间已存在，则忽略该参数。
+ collectionspace ：集合空间名称。
+ collection ：集合名称（不包含集合空间名称）。
+ autosplit ：创建集合时指定是否自动切分。必须配合散列分区和域使用。
+ shardingkey ：创建集合时指定的分区键。
+ shardingtype ：创建集合时指定的分区类型，取值可以是“hash”和“range”，分别表示散列分区和范围分区。
+ compressiontype ：创建集合时指定的压缩类型，取值可以是“none”、“lzw”和“snappy”，“none”表示不进行压缩。

## 总结

通过本课程，我们学习了如何在安装有 SequoiaDB 巨杉数据库及 MySQL 实例的环境中安装 Spark，并且学习了如何在 SparkSQL 中操作 SequoiaDB 巨杉数据库的数据。

今天我们学习到的知识点为：

+ SequoiaDB 集合空间、集合的创建
+ SparkSQL 实例的配置
+ SparkSQL 实例中操作 SequoiaDB 巨杉数据库的数据
