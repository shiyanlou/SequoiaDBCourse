---
show: step
version: 1.0
enable_checker: true
---


## 课程介绍
本实验一步一步的带领你在linux环境中部署巨杉数据库的SparkSQL实例。

#### SparkSQL实例简介
Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎。Spark为结构化数据处理引入了一个称为Spark SQL的编程模块。它提供了一个称为DataFrame的编程抽象，并且可以充当分布式SQL查询引擎。

## 在 SequoiaDB-MySQL 实例中创建 Spark 元数据库
1）使用MySQL Shell 连接 SequoiaDB-MySQL 实例；
```shell
bin/mysql -S database/3306/mysqld.sock -u root
```
2) 设置远程连接权限
```sql
UPDATE mysql.user SET host='%' WHERE user='root';
```
3）刷新权限
```sql
FLUSH PRIVILEGES;
```

4）设置 SequoiaDB-MySQL 实例的root用户密码；
```sql
ALTER USER root@'%' IDENTIFIED BY 'root' ;
```

5）创建元数据库；
```sql
CREATE DATABASE metastore CHARACTER SET 'latin1' COLLATE 'latin1_bin' ;
```

## 创建测试数据库实例及数据表
1）创建数据库实例，并切换到该数据库；
```
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

配置用户互信，已经安装了ssh软件包；由于是由sdbadmin用户启动Spark,所以需要配置sdbadmin用户的ssh互相关系。

1）执行ssh-keygen生成公钥和密钥，执行后连续回车即可；
```
ssh-keygen -t rsa
```

2）查看本机主机名

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


4）配置Spark运行参数，编辑文件spark-env.sh
```
cd /opt/spark-2.1.3-bin-hadoop2.7/conf
```

5）编辑 spark-env.sh 文件

```shell
vi conf/spark-env.sh
```
增加以下两行
```
SPARK_WORKER_INSTANCES=2
SPARK_MASTER_HOST=本机主机名
```
6）创建设置元数据数据库配置文件hive-site.xml

```
vi conf/hive-site.xml
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


## 4 启动 Spark 实例

1）启动 Spark 
```shell
sbin/start-all.sh
```

2）查看 Spark 的 master 和 worker 是否启动完成
jps

3）启动thriftserver
```shell
sbin/start-thriftserver.sh --master spark://`hostname`:7077  --executor-cores 2    --total-executor-cores 4 --executor-memory 1g
```

4）进入 beeline 客户端测试 sql；

```shell
bin/beeline
```
5）连接10000端口获取 thriftserver 连接
```
!connect jdbc:hive2://localhost:10000
```

6）创建 company 数据库'
```
CREATE DATABASE company ;
```

7）创建映射表;

```
CREATE TABLE company.employee(
empno int,
ename string,
age int
) USING com.sequoiadb.spark OPTIONS ( host 'localhost:11810' ,collectionspace 'company', collection 'employee', username '',password '') ;
```

8）测试运行 sql ；
```
SELECT avg(age) FROM company.employee ;
```

## 停止SparkSQL实例
1）停止thriftserver
``` 
./stop-thriftserver.sh 
```
2）停止spark
```
./stop-all.sh
```
## 总结




