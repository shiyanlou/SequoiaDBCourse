---
show: step
version: 10.0
enable_checker: true
---

# 使用MySQL实例进行应用开发

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他 MySQL 语法操作，最后展示如何使用 MySQL 的 JAVA 驱动对数据进行 CRUD 操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


## 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 `sdbadmin` 。
```shell
su - sdbadmin
```

>Note:
>
>用户 `sdbadmin` 的密码为 `sdbadmin`

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本
```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看服务状态

#### 查看 SequoiaDB 巨杉数据库引擎节点列表
```shell
sdblist 
```


操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤

#### 查看 MySQL 实例是否已经启动
```shell
 /opt/sequoiasql/mysql/bin/sdb_sql_ctl status
```

>Note:
>
>如果 PID 显示为空，请稍等初始化完成并重试该步骤

## 使用 MySQL shell 进行操作

1）登录 MySQL shell；
```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）查看 MySQL 存储引擎，确认默认存储引擎为 SequoiaDB；
```sql
SHOW ENGINES ;
```

3）创建数据库；
```sql
CREATE DATABASE company ;
USE company ;
```

4）创建包含自增主键字段的 employee 表；
```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```



## 基本数据操作

SequoiaDB 巨杉数据库的 MySQL 实例支持完整的 CRUD 数据基本操作。

1）验证基本的数据写入操作；
```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
INSERT INTO employee VALUES (10005, 'Kyoichi', 23) ;
INSERT INTO employee VALUES (10006, 'Anneke', 19) ;
```


2）退出 MySQL Shell；
```
\q
```
## Java 语言操作 MySQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaSQL-MySQL 实例中的数据，为相关开发人员提供参考。源码已经放置在 /home/sdbadmin/source 目录下。

1）进入源码放置目录；
```shell
cd /home/sdbadmin/source/mysql
```

2）查看 java 文件，一共5个文件；
```shell
ls -trl
```


>Note：
>
> - MySQLConnection.java   连接数据库类
> - Insert.java  写入数据类
> - Select.java 查询数据类
> - Update.java 更新数据类
> - Delete.java    删除数据类

3）对 java 文件进行编译；
```shell
javac -d . *.java
```


## 写入数据

本小节介绍如何使用 MySQL 的 JAVA 驱动连接 SequoiaSQL-MySQL 实例并写入数据。

1）运行 Insert 类的代码；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Insert
```

2）Insert 类代码如下：
```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Insert {
    private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
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
        sb.append("(").append(20001).append(",").append("'Quincy'").append(",").append(30).append("),");
        sb.append("(").append(20002).append(",").append("'Newton'").append(",").append(31).append("),");
        sb.append("(").append(20003).append(",").append("'Dan'").append(",").append(32).append("),");

        sb.deleteCharAt(sb.length() - 1);
        sql = sql + sb.toString();
        System.out.println(sql);
        psmt.addBatch(sql);
        psmt.executeBatch();
        connection.close();
    }
}
```





## 查询数据

本小节介绍如何使用 MySQL 的 JAVA 驱动连接 SequoiaSQL-MySQL 实例并查询数据。

1）运行 Select 类的代码；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```


2）Select 类代码如下：
```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Select {
    private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
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
        System.out.println("----------------------------------------------------------------------");
        System.out.println("empno  \t ename \t age ");
        System.out.println("----------------------------------------------------------------------");
        while(rs.next()){
            Integer empno = rs.getInt("empno");
            String ename = rs.getString("ename");
            String age = rs.getString("age");

            System.out.println(empno + "\t" + ename + "\t" + age );
        }
        connection.close();
    }
}
```


## 更新数据

本小节介绍如何使用 MySQL 的 JAVA 驱动连接 SequoiaSQL-MySQL 实例并更新数据。

1）运行 Update 类代码；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Update
```

2）查询确认10001雇员的年龄已经被更改为 49；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```

3）Update 类代码如下：
```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Update {
    private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
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



## 删除数据

本小节介绍如何使用 MySQL 的 JAVA 驱动连接 SequoiaSQL-MySQL 实例并删除数据。

1）运行 Delete 类的代码；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Delete
```

2）查询确认 empno 为 10006 的雇员信息已经被删除；
```shell
java -cp  .:../mysql-connector-java-5.1.48.jar com.sequoiadb.mysql.Select
```

3）Delete 类代码如下：
```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Delete {
    private static String url = "jdbc:mysql://127.0.0.1:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
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


## 总结
SequoiaSQL-MySQL 实例完全兼容 MySQL 语法，使用 MySQL 的驱动即可完成对 MySQL 实例的数据操作。应用从原生 MySQL 切换到 SequoiaDB 巨杉数据库可以做到平滑迁移，无需修改代码。


