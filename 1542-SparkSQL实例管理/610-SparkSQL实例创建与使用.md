---
show: step
version: 1.0 
enable_checker: true 
---


# SparkSQL 实例创建与使用

## 试验介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，进行 SparkSQL 实例的安装，并插入一定量的数据。

#### 知识点：

+ SequoiaDB 集合空间、集合的创建
+ SparkSQL 实例的配置
+ SparkSQL 实例中操作 SequoiaDB 数据库的数据

## 安装 SparkSQL 实例

#### 解压 SparkSQL 安装包

1) 检查 SparkSQL 安装包

```
ll /home/sdbadmin/soft/
```

![1542-610-1](https://doc.shiyanlou.com/courses/1542/1207281/26b2e3cb004f59c9c210afadc457ee4d)

2) 解压 SparkSQL 安装包

由于 sdbadmin 用户没有 /opt 目录下的创建目录权限，因此使用 sudo 命令进行解压，sdbadmin 用户的密码是 sdbadmin。

```
sudo tar -zxvf /home/sdbadmin/soft/spark-2.4.4-bin-hadoop2.7.tar.gz -C /opt
sudo chown -R sdbadmin:sdbadmin_group /opt/spark-2.4.4-bin-hadoop2.7
```

#### 放置驱动包

1) 拷贝 SequoiaDB 和 SparkSQL 的连接驱动到 SparkSQL 的 jars 目录下

```
cp /opt/sequoiadb/spark/spark-sequoiadb_2.11-3.4.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

2) 拷贝 MySQL 驱动到 SparkSQL 的 jars 目录下

```
cp /home/sdbadmin/soft/mysql-jdbc.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

3) 拷贝 SequoiaDB 的 JAVA 驱动到 SparkSQL 的 jars 目录下

```
cp /opt/sequoiadb/java/sequoiadb-driver-3.4.jar /opt/spark-2.4.4-bin-hadoop2.7/jars/
```

#### 配置 SparkSQL

1) 设置 spark-env.sh

```
cp /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh.template /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh

echo "SPARK_WORKER_INSTANCES=2" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh 
echo "SPARK_MASTER_IP=127.0.0.1" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-env.sh
```

2) 设置 spark-defaults.conf

```
cp /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf.template /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf

echo "spark.sql.cbo.enabled  true" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
echo "spark.sql.cbo.joinReorder.dp.star.filter true" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
echo "spark.sql.cbo.joinReorder.dp.threshold  1024" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
echo "spark.sql.cbo.joinReorder.enabled true" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
echo "spark.sql.cbo.starSchemaDetection true" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
echo "spark.sql.crossJoin.enabled true" >> /opt/spark-2.4.4-bin-hadoop2.7/conf/spark-defaults.conf
```

3) 设置 slaves

```
cp /opt/spark-2.4.4-bin-hadoop2.7/conf/slaves.template /opt/spark-2.4.4-bin-hadoop2.7/conf/slaves
```

4) 创建设置元数据数据库配置文件 hive-site.xml

创建 hive-site.xml 文件，并使用 vi 打开文件

```
touch /opt/spark-2.4.4-bin-hadoop2.7/conf/hive-site.xml
vi /opt/spark-2.4.4-bin-hadoop2.7/conf/hive-site.xml
```

按 `i` 进入插入模式，输入下面的配置信息

```xml
<configuration>
   <property>
     <name>hive.metastore.schema.verification</name>
     <value>false</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://sdbserver1:3306/metastore</value>
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
```

文件保存后，退出编辑模式。

#### 配置 SparkSQL 元数据库

1) 使用 Linux 命令进入 MySQL shell

```
mysql -h 127.0.0.1 -u root
```

![1542-610-2](https://doc.shiyanlou.com/courses/1542/1207281/35b933f575a8668502935c2d3c26772c)

2) 创建 metauser 用户

```sql
CREATE USER 'metauser'@'%' IDENTIFIED BY 'metauser' ;
```

![1542-610-3](https://doc.shiyanlou.com/courses/1542/1207281/4bf8426e52c44bb09919c1f778b56f4d)

3) 给 metauser 用户授权

```sql
GRANT ALL ON *.* TO 'metauser'@'%' ;
```

![1542-610-4](https://doc.shiyanlou.com/courses/1542/1207281/3bd44330222ef26134db41719d3e1517)

4) 创建 SparkSQL 元数据库

```sql
CREATE DATABASE metastore CHARACTER SET 'latin1' COLLATE 'latin1_bin' ;
```

![1542-610-5](https://doc.shiyanlou.com/courses/1542/1207281/1b51e810ab6f8ef935395d21579b503d)

5) 刷新权限
```sql
FLUSH PRIVILEGES ;
```

![1542-610-6](https://doc.shiyanlou.com/courses/1542/1207281/c4e6789a504b6158f5e3b86a44809115)

#### 启动 SparkSQL 服务

1) 启动 SparkSQL

```
/opt/spark-2.4.4-bin-hadoop2.7/sbin/start-all.sh
```

![1542-610-7](https://doc.shiyanlou.com/courses/1542/1207281/8db1588d0b54de7d2fe3dcfbcacf9d3f)

2) 启动 thriftserver

```
/opt/spark-2.4.4-bin-hadoop2.7/sbin/start-thriftserver.sh
```

![1542-610-8](https://doc.shiyanlou.com/courses/1542/1207281/b4bc251d26d926395cb7a0d05d8d4f98)

## 连接测试

#### 在 SequoiaDB 中建立集合空间和集合

1) 使用 Linux 命令进入 SequoiaDB Shell 命令行

```
sdb
```

2) 连接 SequoiaDB 数据库

```javascript
db = new Sdb ("localhost", 11810) ;
```

2) 创建 company_domains 逻辑域

```javascript
db.createDomain ("company_domains", ["group1", "group2", "group3"], { AutoSplit : true }) ;
```

3) 创建 company 集合空间

```javascript
db.createCS ("company",{Domain: "company_domains"}) ;
```

4) 创建 employee 集合

```javascript
db.company.createCL ("employee", {"ShardingKey" : { "_id" : 1} , "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```

![1542-610-9](https://doc.shiyanlou.com/courses/1542/1207281/574ce264d392979ae4ef35c939e1e598)

#### 在 SparkSQL 中关联 SequoiaDB 的集合空间、集合

进入 SparkSQL Beeline Shell，在 SparkSQL 实例中创建 employee 表并与 SequoiaDB 中的集合空间、集合关联。

1) 登录到 SparkSQL 实例 Beeline Shell

```
/opt/spark-2.4.4-bin-hadoop2.7/bin/beeline -u 'jdbc:hive2://localhost:10000'
```

2) 创建 employee 表

创建 employee 表，并且与 SequoiaDB 中的集合 company.employee 进行关联

```sql
CREATE TABLE employee (
  empno  INT,
  ename  VARCHAR(128),
  age    INT
) USING com.sequoiadb.spark OPTIONS (
  host 'localhost:11810',
  collectionspace 'company',
  collection 'employee'
) ;
```

![1542-610-10](https://doc.shiyanlou.com/courses/1542/1207281/7513456f4f9c0730b34e5ebf1dcce0a4)

#### 在 SparkSQL 中进行数据操作

1) 插入数据

```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
```

![1542-610-11](https://doc.shiyanlou.com/courses/1542/1207281/5a29365c408c0525cbec5dc7e7441426)

2) 进行数据查询

```sql
SELECT * FROM employee ;
```

![1542-610-12](https://doc.shiyanlou.com/courses/1542/1207281/2a5fa712de8bc2dcb23f453a8b56023b)

## 总结

通过本课程，我们学习了如何在安装有 SequoiaDB 数据库及 MySQL 实例的环境中安装 SparkSQL，并且学习了如何在 SparkSQL 中操作 SequoiaDB 数据库的数据。

今天我们学习到的知识点为：

+ SequoiaDB 集合空间、集合的创建
+ SparkSQL 实例的配置
+ SparkSQL 实例中操作 SequoiaDB 数据库的数据
