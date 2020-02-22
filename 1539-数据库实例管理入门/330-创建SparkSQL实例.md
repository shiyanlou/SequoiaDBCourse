---
show: step
version: 1.0
enable_checker: true
---


## 课程介绍
本实验一步一步的带领你在linux环境中部署巨杉数据库的SparkSQL实例，同时讲述建表、查询等基本操作。

#### SparkSQL实例简介
Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎。Spark为结构化数据处理引入了一个称为Spark SQL的编程模块。它提供了一个称为DataFrame的编程抽象，并且可以充当分布式SQL查询引擎。
#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中SequoiaSQL-SparkSQL 数据库实例包括2个 worker 节点，1个 SequoiaSQL-MySQL 数据库实例节点，SequoiaDB 巨杉数据库包括1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/f94f233be5f5d42622a2f29ec0c30c1f)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaDB-SparkSQL 实例连接器均为 3.4 版本。JDK 版本为openjdk1.8。

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



## 创建 Spark 元数据库

本示例使用 MySQL 实例存储 Spark引擎的元数据信息，故需要在 MySQL 实例中创建一个数据库进行存储。

1）进入 SequoiaDB-MySQL 安装目录；

```
cd /opt/sequoiasql/mysql/
```

2）使用 MySQL Shell 连接 SequoiaDB-MySQL 实例；

```shell
bin/mysql -h 127.0.0.1 -P 3306 -u root
```

3）设置远程连接权限；

```sql
UPDATE mysql.user SET host='%' WHERE user='root';
```


4）设置 SequoiaDB-MySQL 实例的root用户密码；

```sql
ALTER USER root@'%' IDENTIFIED BY 'root' ;
```

5）刷新权限；

```sql
FLUSH PRIVILEGES;
```

6）创建元数据库；

```sql
CREATE DATABASE metastore CHARACTER SET 'latin1' COLLATE 'latin1_bin' ;
```

## 创建测试数据库实例及数据表
为 Spark 安装完毕后进行数据操作测试在 SequoiaSQL-MySQL 实例中创建的表，SequoiaSQL-MySQL 实例默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。

1）创建数据库实例，并切换到该数据库；
```sql
create database company ;
use company;
```
2）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

3）进行基本的数据写入操作；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

4）退出 MySQL Shell；
```shell
\q
```

## Spark 实例配置

Spark 安装包和 MySQL 驱动已放置在 /home/sdbadmin/soft 目录下，本示例使用 Spark 的 standalone 模式安装。

#### 解压 Spark 安装包

1）进入安装包存放目录；
```
cd /home/sdbadmin/soft
```

2）解压安装包；
```
tar -zxvf spark-2.4.4-bin-hadoop2.7.tar.gz -C /opt
```

#### 设置免密
1）执行ssh-keygen生成公钥和密钥，执行后连续回车即可；
```
ssh-keygen -t rsa
```

2）查看本机主机名；

```shell
hostname
```

3）执行 ssh-copy-id，把公钥拷贝到本机的 sdbadmin 用户；
```
ssh-copy-id  sdbadmin@本机主机名
```

>
>Note:
>
> sdbadmin的密码是：sdbadmin
> 单机不需要拷贝到其它服务器上，如果是多机部署，需要配置所有服务器的互相关系。

#### 设置 spark-env.sh

1）进入 Spark 的配置目录；

```
cd /opt/spark-2.4.4-bin-hadoop2.7/conf
```

2）从模板中拷贝 spark-env.sh 文件；

```shell
cp spark-env.sh.template spark-env.sh
```

3）编辑 spark-env.sh 文件；

```shell
vi spark-env.sh
```
增加设置 Spark 的 Master 机器主机名；
```
SPARK_MASTER_HOST=本机主机名
```

#### 拷贝相关驱动
用户只要将SequoiaDB for Spark 连接器 spark-sequoiadb.jar 和SequoiaDB 的 java 驱动 sequoiadb.jar 加入 Spark 的 jar 目录即可，另外我们使用了 MySQL 作为元数据存储数据库，也需要加入 MySQL 的驱动。

1）拷贝 spark-sequoiadb.jar 驱动连接器；
```shell
cp /opt/sequoiadb/java/sequoiadb-driver-3.4.jar  /opt/spark-2.4.4-bin-hadoop2.7/jars/
```


2）拷贝 SequoiaDB 的 java 驱动 sequoiadb.jar；
```shell
cp /opt/sequoiadb/spark/spark-sequoiadb_2.11-3.4.jar  /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

3）拷贝 MySQL 的 java 驱动 mysql-jdbc.jar；
```shell
cp /home/sdbadmin/soft/mysql-jdbc.jar  /opt/spark-2.4.4-bin-hadoop2.7/jars/  
```

#### 设置元数据库

1）创建设置元数据数据库配置文件 hive-site.xml；

```
vi hive-site.xml
```
输入以下内容到 hive-site.xml 文件
```
<configuration>
   <property>
     <name>hive.metastore.schema.verification</name>
     <value>false</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true</value>
      <description>JDBC connect string for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>root</value>
   </property>
   <property>
      <name>datanucleus.autoCreateSchema</name>
      <value>true</value>
      <description>creates necessary schema on a startup if one doesn't exist. set this to false, after creating it once</description>
   </property>
</configuration>
```


## 测试 SparkSQL 实例

之前的小节已经创建了 Spark 元数据库，并且通过 SequoiaSQL-MySQL 实例创建了分区表。

1）进入 Spark 的安装目录；

```
cd /opt/spark-2.4.4-bin-hadoop2.7
```

2）启动 Spark；

```shell
sbin/start-all.sh
```

3）查看 Spark 的 master 和 worker 是否启动完成；

```
jps
```

4）启动 thriftserver 服务；

```shell
sbin/start-thriftserver.sh   \
--master spark://`hostname`:7077 \
--executor-cores 1 \
--executor-memory 512m
```

4）进入 Beeline 客户端测试 sql；

```shell
bin/beeline
```

5）连接10000端口获取 thriftserver 连接；

```
!connect jdbc:hive2://localhost:10000
```

6）创建 company 数据库；

```
CREATE DATABASE company ;
```

7）创建映射表；

```sql
CREATE TABLE company.employee(
empno INT,
ename STRING,
age INT
) USING com.sequoiadb.spark OPTIONS ( host 'localhost:11810' ,collectionspace 'company', collection 'employee', username '',password '') ;
```

8）测试运行 sql ；

```sql
SELECT avg(age) FROM company.employee ;
```

9）退出 Beeline 客户端；

```
!quit
```

## 总结
本课程介绍了 standalone 模式下的 Spark 如何与 SequoiaDB 数据库引擎进行对接，并进行了数据操作。 
