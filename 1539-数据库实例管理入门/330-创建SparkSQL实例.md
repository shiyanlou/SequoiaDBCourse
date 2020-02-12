## 四，SparkSQL实例简介
Sequoiadb作为一个分布式集群数据库，可以为Spark提供数据；使用SparkSQL接口可以通过SQL方式非常方便的访问sequoiadb中存储的数据；并且可以在一个多副本的集群中，把spark部署在一个数据副本的服务器上，与其它副本隔离
### 4.1 准备课程环境
在属主机上启动docker课程容器，并进入container。
```
docker run -it --privileged=true --name sdbtestfu -h sdb sdbinstance5
```
以下步骤在Container中执行。

Spark实例目录及启动sdb进程。
``` 
su - sdbadmin
sdbstart -t all
sdblist -l
cd /opt
ls -l

cd spark-2.1.3-bin-hadoop2.7
ls -l
```
本例中安装了spark 2.1.3

### 4.2 Spark 实例配置

配置用户互信，已经安装了ssh软件包；由于是由sdbadmin用户启动Spark,所以需要配置sdbadmin用户的ssh互相关系。

使用root用户，启动sshd：
```
/etc/init.d/ssh start
 * Starting OpenBSD Secure Shell server sshd 
```
为sdbadmin 用户配置互相：
```
su - sdbadmin

ssh sdb ssh-keygen -t rsa
ssh sdb cat ~/.ssh/id_rsa.pub>>~/.ssh/authorized_keys
```
sdbadmin的密码是：sdbadmin

单机不需要拷贝到其它服务器上，如果是多机部署，需要配置所有服务器的互相关系。
```
scp ~/.ssh/authorized_keys ~/.ssh/known_hosts sdb:~/.ssh/
```
测试互相关系：
```
ssh sdb
```

配置Spark运行参数，编辑文件spark-env.sh
```

cd /opt/spark-2.1.3-bin-hadoop2.7/conf
vi spark-env.sh
```
添加配置,使用ifconfig 看看Container的ip地址；根据输出的ip地址修改：
示例如下：
```
SPARK_MASTER_PORT="7077"
SPARK_MASTER_IP=172.17.0.3
SPARK_CLASSPATH=/opt/sequoiadb/spark/spark-sequoiadb_2.11-3.2.4.jar:/opt/sequoiadb/java/sequoiadb-driver-3.2.4.jar:/opt/spark-2.1.3-bin-hadoop2.7/jars/mysql-connector-java-5.1.47.jar
MASTER="spark://${SPARK_MASTER_IP}:${SPARK_MASTER_PORT}"
#SPARK_WORK_MEMORY=1g
#SPARK_EXECUTOR_CORES=2
export JAVA_HOME=/opt/sequoiadb/java/jdk
```
配置slaves
```
cd /opt/spark-2.1.3-bin-hadoop2.7/conf
cp slaves.template slaves
vi slaves
```
在文件末尾增加slave运行的服务器名称，本例在本机上运行：
```
localhost
```
配置spark的hive-site.xml文件，在本例中已经配置，使用mysql做为元数据库，mysql的用户名和密码是：sdbadmin
```
cd /opt/spark-2.1.3-bin-hadoop2.7/conf
vi hive-site.xml

sdbadmin@sdb:/opt/spark-2.1.3-bin-hadoop2.7/conf$ more hive*xml
```
示例如下：
```
<configuration>
   <property>
     <name>hive.metastore.schema.verification</name>
     <value>false</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://sdb:3306/metastore?createDatabaseIfNotExist=true</value>
      <description>JDBC connect string for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>sdbadmin</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>sdbadmin</value>
   </property>
   <property>
      <name>datanucleus.autoCreateSchema</name>
      <value>false</value>
      <description>creates necessary schema on a startup if one doesn't exist. set this to false, after creating it once</description>
   </property>
</configuration>
```
创建mysql实例,用于存储sparksql的元数据信息。

1.root用户安装mysql实例
```
cd /sequoiadb/sequoiadb-3.2.4
./sequoiasql-mysql-3.2.4-linux_x86_64-installer.run --mode text
```
缺省参数安装。

2.创建MySQL实例
```
su - sdbadmin
cd /opt/sequoiasql/mysql/bin
./sdb_sql_ctl addinst myinst -D database/3306/
```
启动mysql实例作为元数据库,并添加用户。

```
mysql -h127.0.0.1 -uroot -p

create user 'sdbadmin'@'%' identified by 'sdbadmin';
GRANT ALL PRIVILEGES ON metastore.* TO 'sdbadmin'@'%' IDENTIFIED BY 'sdbadmin' WITH GRANT OPTION;
FLUSH PRIVILEGES;
quit;
```

### 4.3启动spark实例
```
/opt/spark-2.1.3-bin-hadoop2.7/sbin/start-all.sh
```
检查spark进程是否启动：
```
netstat -an|grep 7077
sdbadmin@sdb:/opt/sequoiasql/mysql/bin$ netstat -an|grep 7077
tcp        0      0 172.17.0.3:7077         0.0.0.0:*               LISTEN     
tcp        0      0 172.17.0.3:37452        172.17.0.3:7077         ESTABLISHED
tcp        0      0 172.17.0.3:7077         172.17.0.3:37452        ESTABLISHED
```
启动用于在spark中执行SQL的thriftserver进程：
```
/opt/spark-2.1.3-bin-hadoop2.7/sbin/start-thriftserver.sh --master spark://sdb:7077
```
检查thriftserver是否启动：
```
netstat -an|grep 10000
tcp        0      0 0.0.0.0:10000           0.0.0.0:*               LISTEN     
```

### 4.4 在sdb中创建测试用集合空间和集合
使用sdb命令行工具：

```
sdb
> db=new Sdb()

> db.createCS("sparkCS")
localhost:11810.sparkCS
Takes 0.184759s.
> db.sparkCS.createCL("sparkCL")
localhost:11810.sparkCS.sparkCL
Takes 0.624262s.
```
插入测试数据：
```
> db.sparkCS.sparkCL.insert({my_name:"testuser1",my_age:20,my_address:"chengdu1,sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.006908s.
> db.sparkCS.sparkCL.insert({my_name:"testuser2",my_age:22,my_address:"chengdu2,sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.002743s.
> db.sparkCS.sparkCL.insert({my_name:"testuser3",my_age:23,my_address:"chengdu3,sichuan"})
{
  "InsertedNum": 1,
  "DuplicatedNum": 0
}
Takes 0.001164s.
```
在sdb中查询数据：
```
> db.sparkCS.sparkCL.find()
{
  "_id": {
    "$oid": "5e169d46a28dc4ea1b1a71fe"
  },
  "my_name": "testuser1",
  "my_age": 20,
  "my_address": "chengdu1,sichuan"
}
{
  "_id": {
    "$oid": "5e169d52a28dc4ea1b1a71ff"
  },
  "my_name": "testuser2",
  "my_age": 22,
  "my_address": "chengdu2,sichuan"
}
{
  "_id": {
    "$oid": "5e169d5ba28dc4ea1b1a7200"
  },
  "my_name": "testuser3",
  "my_age": 23,
  "my_address": "chengdu3,sichuan"
}
Return 3 row(s).

quit
```
### 4.5在beeline中，通过sparkSQL实例操作数据。

启动beeline，然后操作数据。
```
cd /opt/spark-2.1.3-bin-hadoop2.7/bin
/opt/spark-2.1.3-bin-hadoop2.7/bin/beeline -u jdbc:hive2://sdb:10000 -n sdbadmin -p sdbadmin
Connecting to jdbc:hive2://sdb:10000
log4j:WARN No appenders could be found for logger (org.apache.hive.jdbc.Utils).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Connected to: Spark SQL (version 2.1.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://sdb:10000> 
```
或者进入beeline之后再连接数据库：
```
beeline> !connect jdbc:hive2://sdb:10000
```
关联sdb中的集合空间和集合，为spark中的一张表：
```
0: jdbc:hive2://sdb:10000> CREATE table sparkCL ( my_name string, my_age int,my_address string) using com.sequoiadb.spark OPTIONS ( host 'sdb:11810', collectionspace 'sparkCS', collection 'sparkCL');
+---------+--+
| Result  |
+---------+--+
+---------+--+
No rows selected (1.378 seconds)
0: jdbc:hive2://sdb:10000> select * from sparkCL;
+------------+---------+-------------------+--+
|  my_name   | my_age  |    my_address     |
+------------+---------+-------------------+--+
| testuser1  | 20      | chengdu1,sichuan  |
| testuser2  | 22      | chengdu2,sichuan  |
| testuser3  | 23      | chengdu3,sichuan  |
+------------+---------+-------------------+--+
3 rows selected (4.646 seconds)
0: jdbc:hive2://sdb:10000> 
```

通过spark 插入数据：
```
0: jdbc:hive2://sdb:10000> insert into sparkCL values("testuser4",24,"chengdu4,sichuan");
+---------+--+
| Result  |
+---------+--+
+---------+--+
No rows selected (0.442 seconds)
0: jdbc:hive2://sdb:10000> select * from sparkCL;
+------------+---------+-------------------+--+
|  my_name   | my_age  |    my_address     |
+------------+---------+-------------------+--+
| testuser1  | 20      | chengdu1,sichuan  |
| testuser2  | 22      | chengdu2,sichuan  |
| testuser3  | 23      | chengdu3,sichuan  |
| testuser4  | 24      | chengdu4,sichuan  |
+------------+---------+-------------------+--+
4 rows selected (0.395 seconds)
0: jdbc:hive2://sdb:10000> 

0: jdbc:hive2://sdb:10000> select * from sparkCL where my_name="testuser1";
+------------+---------+-------------------+--+
|  my_name   | my_age  |    my_address     |
+------------+---------+-------------------+--+
| testuser1  | 20      | chengdu1,sichuan  |
+------------+---------+-------------------+--+
1 row selected (0.385 seconds)
0: jdbc:hive2://sdb:10000> !quit
```


### 4.6 在sdb中查看数据.
使用sdb命令行工具查看数据：
```
sdbadmin@sdb:/opt/spark-2.1.3-bin-hadoop2.7/conf$ sdb
Welcome to SequoiaDB shell!
help() for help, Ctrl+c or quit to exit
> db=new Sdb()
localhost:11810
Takes 0.001274s.
> db.sparkCS.sparkCL.find()
{
  "_id": {
    "$oid": "5e169d46a28dc4ea1b1a71fe"
  },
  "my_name": "testuser1",
  "my_age": 20,
  "my_address": "chengdu1,sichuan"
}
{
  "_id": {
    "$oid": "5e169d52a28dc4ea1b1a71ff"
  },
  "my_name": "testuser2",
  "my_age": 22,
  "my_address": "chengdu2,sichuan"
}
{
  "_id": {
    "$oid": "5e169d5ba28dc4ea1b1a7200"
  },
  "my_name": "testuser3",
  "my_age": 23,
  "my_address": "chengdu3,sichuan"
}
{
  "_id": {
    "$oid": "5e16a755e4b07352e3230430"
  },
  "my_name": "testuser4",
  "my_age": 24,
  "my_address": "chengdu4,sichuan"
}
Return 4 row(s).
Takes 0.001660s.
> quit
```
### 4.7 停止SparkSQL实例
```
cd /opt/spark-2.1.3-bin-hadoop2.7/sbin
```
停止thriftserver
``` 
./stop-thriftserver.sh 
```
停止spark
```
./stop-all.sh
```
退出Container
```
$exit
#exit
```
不要删除Container：sdbtestfu 用于后面的多实例课程。