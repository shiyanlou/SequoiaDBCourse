---
show: step
version: 88.0
enable_checker: true
---

## 课程介绍

本课程主要介绍如何在 Linux 环境中部署巨杉数据库的 SparkSQL 实例，以及讲解建表、查询等基本操作。

#### SparkSQL实例简介
Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎。Spark 为结构化数据处理引入了一个称为 Spark SQL 的编程模块。它提供了一个称为 DataFrame 的编程抽象，并且可以充当分布式 SQL 查询引擎。
#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SparkSQL 数据库实例包括2个 worker 节点，1个 SequoiaSQL-MySQL 数据库实例节点，1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1539/1207281/97a485c5a18b385fdd1a24f2e62ed888-0)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaDB-SparkSQL 实例连接器均为 3.4 版本。JDK 版本为 openjdk1.8。

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

查看 SequoiaDB 巨杉数据库引擎版本。

```shell
sequoiadb --version
```
操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表。

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/02fcaa58ac27e91688ead137fa748d6e)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤



## 创建 Spark 元数据库

本示例使用 MySQL 实例存储 Spark 引擎的元数据信息，故需要在 MySQL 实例中创建一个数据库进行存储。

1）使用 MySQL Shell 连接 SequoiaDB-MySQL 实例；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）赋予 root 用户远程连接权限；

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION ;
```

3）更新 root@localhost 用户密码；

```sql
SET PASSWORD FOR root@localhost = PASSWORD('root') ;
```

4）刷新权限；

```sql
FLUSH PRIVILEGES ;
```

5）创建元数据库；

```sql
CREATE DATABASE metastore CHARACTER SET 'latin1' COLLATE 'latin1_bin' ;
```

## 创建测试数据库实例及数据表
在 SequoiaSQL-MySQL 实例中创建数据表，为 Spark 安装完毕后进行数据操作测试提供测试数据。SequoiaSQL-MySQL 实例默认使用 SequoiaDB 数据库存储引擎，包含主键或唯一键的表将会默认以唯一键作为分区键，进行自动分区。

1）创建数据库实例，并切换到该数据库；

```sql
CREATE DATABASE company ;
USE company ;
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

```sql
\q
```

## Spark 实例配置

Spark 安装包和 MySQL 驱动已放置在 /home/sdbadmin/soft 目录下，本示例使用 Spark 的 standalone 模式安装。

#### 解压 Spark 安装包

1）进入安装包存放目录；

```shell
cd /home/sdbadmin/soft
```

2）解压安装包；

```shell
tar -zxvf spark-2.4.4-bin-hadoop2.7.tar.gz -C /opt
```

## 设置免密

部署 Spark 实例为了实现自动化操作,需要配置ssh免密码登陆方式。

1）执行ssh-keygen生成公钥和密钥，执行后连续回车即可；

```shell
ssh-keygen -t rsa
```

2）执行 ssh-copy-id，把公钥拷贝到本机的 sdbadmin 用户；

```shell
ssh-copy-id  sdbadmin@`hostname`
```

>
>Note:
>
> 用户 sdbadmin 的密码是：`sdbadmin`

操作截图：

![](https://doc.shiyanlou.com/courses/1539/1207281/4ddc60dccd197c82bb269987a723d96a-0)

## 设置 spark-env.sh

1）进入 Spark 的配置目录；

```shell
cd /opt/spark-2.4.4-bin-hadoop2.7/conf
```

2）从模板中拷贝 spark-env.sh 文件；

```shell
cp spark-env.sh.template spark-env.sh
```

3）设置 Spark 实例的 Master；

```shell
echo "SPARK_MASTER_HOST=`hostname`" >> spark-env.sh
```

4）查看 spark-env.sh 文件是否设置成功；

```shell
cat spark-env.sh
```

## 设置元数据库

指定 Spark 实例的元数据信息存放的数据库信息。

1）创建设置元数据数据库配置文件 hive-site.xml；

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
EOF
```

2）检查是否创建成功；

```shell
cat /opt/spark-2.4.4-bin-hadoop2.7/conf/hive-site.xml
```


## 拷贝相关驱动
用户只要将 SequoiaDB for Spark 连接器 spark-sequoiadb.jar 和 SequoiaDB 的 Java 驱动 sequoiadb.jar 加入 Spark 的 jar 目录即可，另外本示例使用了 MySQL 作为元数据存储数据库，也需要加入 MySQL 的 Java 驱动 mysql-jdbc.jar。

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

```shell
jps
```
操作截图：
![](https://doc.shiyanlou.com/courses/1539/1207281/81b2b0f2eee0b4ef8ac6ed28a191acca-0)

4）启动 spark-sql 客户端；

```shell
bin/spark-sql
```

5）创建 company 数据库；

```sql
CREATE DATABASE company ;
USE company ;
```


6）创建映射表；

```sql
CREATE TABLE company.employee (
empno INT,
ename STRING,
age INT
) USING com.sequoiadb.spark OPTIONS (host 'localhost:11810', collectionspace 'company', collection 'employee', username '', password '') ;
```

7）测试运行 sql ；

```sql
SELECT AVG(age) FROM company.employee ;
```

8）退出 Beeline 客户端；

```sql
!quit
```

## 总结
本课程介绍了 standalone 模式下的 Spark 如何与 SequoiaDB 数据库引擎进行对接，并进行了数据操作。 
















