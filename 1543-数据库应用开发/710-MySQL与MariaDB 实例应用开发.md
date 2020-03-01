---
show: step
version: 3.0
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他 MySQL 语法操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 SequoiaSQL-MySQL 数据库实例节点、1 个引擎协调节点、1 个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/7668dd33f9b1bee6a5d4209bcb529023)

详细了解 SequoiaDB 巨杉数据库系统架构：
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
>用户 sdbadmin 的密码为 sdbadmin

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看节点启动列表

1）查看 SequoiaDB 巨杉数据库引擎节点列表；

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量少于上图中的数量，请稍等初始化完成并重试该步骤。

2）检查 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl status
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/582f2d310ee38910e856e13415f47855-0)

## 创建数据库及数据表
进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，为接下来验证 MySQL 语法特性做准备。

#### 登录 MySQL 实例 Shell
```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

#### 创建数据库
在MySQL实例中创建新数据库 company，并切换至 company 库；
```sql
CREATE DATABASE company ;
USE company ;
```

#### 创建分区表
1）在 MySQL 实例 company 数据库中创建分区表 employee：
```sql
CREATE TABLE employee (
	empno INT,
	ename VARCHAR(128),
	age INT,
	PRIMARY KEY (empno)
) ENGINE = sequoiadb COMMENT = "雇员表, sequoiadb:{ table_options : { ShardingKey : { 'empno' : 1 } , ShardingType : 'hash' , 'Compressed' : true , 'CompressionType' : 'lzw' , 'AutoSplit' : true , 'EnsureShardingIndex' : false } }" ;
```

>Note:
>
>[MySQL 实例参数配置](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1566551297-edition_id-0)

2）查看 MySQL 实例分区表结构；

```sql
SHOW CREATE TABLE employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/b922380b36312dbcd841945d875c949e)

## 分区表中数据操作
通过 SequoiaSQL-MySQL 实例进行数据插入、查询、更新、删除操作。

#### 分区表中插入数据
在分区表 employee 中插入数据：

```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
INSERT INTO employee VALUES (10005, 'Kyoichi', 23) ;
INSERT INTO employee VALUES (10006, 'Anneke', 19) ;
```

#### 查询分区表中的数据
查询分区表 employee 中 age 大于20，小于30的数据；

```sql
SELECT * FROM employee WHERE age > 20 AND age < 30 ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/b08f930e4384a2c8be7dd7e8bcdd0f05)


#### 更新分区表中的数据
1）更新分区表 employee 中的数据，将 empno 为10001的记录 age 更改为34；

```sql
UPDATE employee SET age=34 WHERE empno=10001 ;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功；

```sql
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/dcf250f3fc2c0cbfb37af7d2e904e04a)

#### 删除分区表中的数据
1）删除分区表 employees 中的数据，将 empno 为 10006 的记录删除；

```sql
DELETE FROM employee WHERE empno=10006 ;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除；

```sql
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/8bff0f74eabcba85cd02e10de89065cd)

## 索引使用
通过 SequoiaSQL-MySQL 实例进行表上索引的创建及查看执行计划。

#### 分区表中索引使用
1）在分区表 employee 的 ename 字段上创建索引；

```sql
ALTER TABLE employee ADD INDEX idx_ename(ename) ;
```

2）显示分区表 employee 查询语句执行计划；

```sql
EXPLAIN SELECT * FROM employee WHERE ename = 'Georgi' ;
```
3）退出 MySQL 客户端

```sql
\q
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/353bd3baa886dc98f98503427f1139b0)


## Java 语言操作 MySQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaSQL-MySQL 实例中的数据，为相关开发人员提供参考。源码已经放置在 /home/sdbadmin/source/mysql 目录下。

1）进入源码放置目录；

```shell
cd /home/sdbadmin/source/mysql
```

2）查看 java 文件，一共5个 java 文件；

```shell
ls -trl
```

>Note：
>
> - MySQLConnection.java   连接数据库类
> - Insert.java            写入数据类
> - Select.java            查询数据类
> - Update.java            更新数据类
> - Delete.java            删除数据类

3）首先对连接的 java 文件进行编译；

```shell
javac -d . MySQLConnection.java
```

#### 在 MySQL 实例中插入数据
1）增加 empno 为 20001 、 20002 和 20003 这三条记录，Insert.java 源代码如下；
```java
package com.sequoiadb.mysql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Insert {
	private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&charac
terEncoding=utf-8&useSSL=false";
	private static String username = "root";
	private static String password = "";

	public static void main(String[] args) throws SQLException {
		insert();
	}

	public static void insert() throws SQLException {
        MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

        Connection connection = mysqlConnection.getConnection();
        String sql = "INSERT INTO employee VALUES";
        PreparedStatement psmt = connection.prepareStatement("");
        StringBuffer sb = new StringBuffer();
        sb.append("(").append(20001).append(",").append("'Quincy'").append(",").append(30).
append("),");
        sb.append("(").append(20002).append(",").append("'Newton'").append(",").append(31).
append("),");
        sb.append("(").append(20003).append(",").append("'Dan'").append(",").append(32).app
end("),");

        sb.deleteCharAt(sb.length() - 1);
        sql = sql + sb.toString();
        System.out.println(sql);
        psmt.addBatch(sql);
        psmt.executeBatch();
        connection.close();
    }
}
```
2）对 Insert.java 文件进行编译；

```shell
javac -d . Insert.java
```

3）运行 Insert 类的代码可以将数据插入；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Insert
```

4）插入操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/cfa44cf731fb7cabafaf1769c17dddf4-0)  

#### 从 MySQL 实例中查询数据
1）查询表中 empno 、 ename 和 age 这三个字段，Select.java 源代码如下；
```java
package com.sequoiadb.mysql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Select {
	private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&charac
terEncoding=utf-8&useSSL=false";
	private static String username = "root";
	private static String password = "";

	public static void main(String[] args) throws SQLException {
		select();
	}

	public static void select() throws SQLException {
        MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

        Connection connection = mysqlConnection.getConnection();
        String sql = "select * from employee";
        PreparedStatement psmt = connection.prepareStatement(sql);
        ResultSet rs = psmt.executeQuery();
        System.out.println("-------------------------------------------");
        System.out.println("empno \t ename \t age");
        System.out.println("-------------------------------------------");
        while(rs.next()){
            Integer empno = rs.getInt("empno");
            String ename = rs.getString("ename");
            String age = rs.getString("age");
            System.out.println(empno + "\t" + ename + "\t" + age);
        }
        connection.close();
    }
}


```
2）对 Select.java 文件进行编译；

```shell
javac -d . Select.java
```

3）运行 Select 类的代码；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```

4）查询截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/9a0807e3351365ca5af562a78af85137-0)

#### 在 MySQL 实例中更新数据  
1）将 empno 为 10001 的 age 修改为 49 ，Update.java 文件源代码如下；
```java
package com.sequoiadb.mysql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Update {
	private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&charac
terEncoding=utf-8&useSSL=false";
	private static String username = "root";
	private static String password = "";

	public static void main(String[] args) throws SQLException {
		update();
	}

	public static void update() throws SQLException {
		MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

		Connection connection = mysqlConnection.getConnection();
		String sql = "update employee set age = ? where empno = ?";
		PreparedStatement psmt = connection.prepareStatement(sql);
	    psmt.setInt(1, 49);
		psmt.setInt(2, 10001);
		psmt.execute();

		connection.close();
	}
}
```
2）对 Update.java 文件进行编译；

```shell
javac -d . Update.java
```

3）运行 Update 类代码；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Update
```

4）查询确认 empno 为 10001 的记录 age 值是否被更改为 49 ；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```

5）更新操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/8adea8f2804552f884138718fa3af781-0)


#### 在 MySQL 实例中删除数据
1）将 empno 值为 10006 的记录删除， Delete.java 源代码如下； 
```java
package com.sequoiadb.mysql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Delete {
    private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&charac
terEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        delete();
    }

    public static void delete() throws SQLException {
        MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

        Connection connection = mysqlConnection.getConnection();
        String sql = "delete from employee where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 10006);
        psmt.execute();
        connection.close();
    }
}
```
2）对 Delete.java 文件进行编译；

```shell
javac -d . Delete.java
```

3）运行 Delete 类的代码；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Delete
```

4）检查确认 empno 为 10006 的雇员信息已经被删除；

```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```

5）删除操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/fef78d3dc2644066798c8b6bd0aeb561-0)

## 总结

通过本课程，我们通过 MySQL 语法在 SequoiaSQL-MySQL 实例上创建数据库和数据表，并对数据表进行了 CRUD 的基本数据操作；同时展示了使用 JAVA 语言对数据表进行数据操作。可以看出：
- SequoiaSQL-MySQL 实例兼容标准的 MySQL 语法；
- Java语言操作 SequoiaSQL-MySQL 实例中的数据与操作原生 MySQL 中的数据无任何差异，可做到无缝切换；

