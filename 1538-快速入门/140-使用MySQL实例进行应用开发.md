---
show: step
version: 1.0
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

## 切换用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户为 sdbadmin。
```
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

## 使用 MySQL shell 进行操作

1）登录 MySQL shell；
```
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）查看 MySQL 存储引擎，确认默认存储引擎为 SequoiaDB；
```sql
show engines ;
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

5）创建包含唯一索引的 manager 表；
```sql
CREATE TABLE manager (empno INT, department TEXT, UNIQUE INDEX id_idx(empno)) ;
```

## 基本数据操作

SequoiaDB 巨杉数据库的 MySQL 实例支持完整的 CRUD 数据基本操作。

1）验证基本的数据写入操作；
```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
INSERT INTO manager VALUES (1, "Sales Department") ;
```

2）验证单表查询与排序；
```sql
SELECT * FROM employee ORDER BY empno ASC LIMIT 1 ;
```

3）验证两表关联功能；
```sql
SELECT * FROM employee, manager WHERE employee.empno = manager.empno ;
```

4）验证数据更新能力；
```sql
UPDATE employee SET ename = "Bob" WHERE empno = 1 ;
```

5）查看数据结果；
```sql
SELECT * FROM employee ;
```

6）验证数据删除能力；
```sql
DELETE FROM employee WHERE empno = 2 ;
```

7）查看数据结果并确认 empno 为 2 的记录被删除；
```sql
SELECT * FROM employee ;
```

## 事务操作
SequoiaDB 巨杉数据库的 MySQL 数据库实例支持完整的事务操作能力，本小节将验证其基本的回滚与提交能力。

#### 验证回滚能力
1）开始事务；
```sql
BEGIN ;
```

2）写入数据；
```sql
INSERT INTO manager VALUES (2, "Product Department") ;
```

3）回滚事务操作；
```sql
ROLLBACK ;
```

4）查询写入数据，确保刚才写入的数据被撤销；
```sql
SELECT * FROM manager WHERE empno = 2 ;
```

#### 验证提交能力
1）开始事务；
```sql
BEGIN ;
```

2）写入数据；
```sql
INSERT INTO manager VALUES (2, "Product Department") ;
```

3）提交事务；
```sql
COMMIT ;
```

4）查询写入数据，确保之前写入的数据被正确提交；
```sql
SELECT * FROM manager WHERE empno = 2 ;
```

## 查看执行计划
SequoiaDB 巨杉数据库支持标准的 MySQL 执行计划。
```sql
EXPLAIN SELECT * FROM manager WHERE empno = 2 ;
```

## 创建存储过程
SequoiaDB 巨杉数据库支持标准的 MySQL 存储过程。本小节将验证其对标准 MySQL 存储过程的支持。

1）创建存储过程；
```sql
DELIMITER //
CREATE PROCEDURE delete_match ()
BEGIN
 DELETE FROM manager WHERE empno = 2 ;
END//
DELIMITER ;
```

2）调用存储过程；
```sql
CALL delete_match () ;
```

3）查询数据并确保 empno 为 2 的记录被删除；
```sql
SELECT * FROM manager WHERE empno = 2 ;
```

## 创建视图
SequoiaDB 巨杉数据库支持标准的 MySQL 视图功能。本小节将验证其对标准 MySQL 视图能力的支持。

1）创建视图；
```sql
CREATE VIEW manager_view AS
SELECT
 e.ename, m.department
FROM
 employee AS e, manager AS m
WHERE 
 e.empno = m.empno ;
```

2）查询视图数据；
```sql
SELECT * FROM manager_view ;
```

## 在线添加删除主键和索引

本小节将验证 SequoiaDB 巨杉数据库的 MySQL 实例对在线添加与删除索引能力的支持。

1）创建索引；
```sql
ALTER TABLE employee ADD INDEX name_idx (ename) ;
```

2）查看表的索引；
```sql
SHOW INDEX FROM employee ;
```

3）删除创建索引；
```sql
DROP INDEX name_idx ON employee ;
```

4）观察索引情况；
```sql
SHOW INDEX FROM employee ;
```

## 在线修改列操作

SequoiaDB 巨杉数据库支持在线 DDL 操作，包括添加删除列、以及数据类型扩容等的操作。例如添加 DEFAULT NULL 的列，以及为数据类型扩容（如 VARCHAR(64) 改 VARCHAR(128)、INT 改 BIGINT）等轻量级操作，可以实现秒级返回。

1）扩容 employee 表的 ename 字段；
```sql
ALTER TABLE employee MODIFY COLUMN ename VARCHAR (130) ;
```
2）查看更改后的表结构；
```sql
DESC employee ;
```

## 验证 TRUNCATE 方法

验证 TRUNCATE 方法，清空 employee 表，方便后续小节对数据分布进行验证。

1）使用 TRUNCATE 方法清空 employee 表；
```sql
TRUNCATE TABLE employee ;
```

2）验证 employee 数据表已被清空；
```sql
select * from employee ;
```

3）退出 MySQL Shell；
```
\q
```
## Java 语言操作 MySQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaSQL-MySQL 实例中的数据，为相关开发人员提供参考。

#### 连接 MySQL 实例

```java
import java.sql.*;

public class MySQLConnection {

    private String url;
    private String username;
    private String password;
    private String driver = "com.mysql.jdbc.Driver";

    public MySQLConnection(String url, String username, String password){
        this.url = url;
        this.username = username;
        this. password = password;
    }

    public Connection getConnection(){
        Connection connection = null;
        try {
            Class.forName(driver);
            connection = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return connection;
    }
}
```

#### 在 MySQL 实例中插入数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Insert {
    private static String url = "jdbc:mysql://localhost:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "admin";
    public static void main(String[] args) throws SQLException {
        insert();
    }

    public static void insert() throws SQLException {
        MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

        Connection connection = mysqlConnection.getConnection();
        String sql = "INSERT INTO employee VALUES";
        PreparedStatement psmt = connection.prepareStatement("");
        StringBuffer sb = new StringBuffer();
        sb.append("(").append(10001).append(",").append("'Georgi'").append(",").append(48).append("),");
        sb.append("(").append(10002).append(",").append("'Bezalel'").append(",").append(21).append("),");
        sb.append("(").append(10003).append(",").append("'Parto'").append(",").append(33).append("),");
        sb.append("(").append(10004).append(",").append("'Chirstian'").append(",").append(40).append("),");
        sb.append("(").append(10005).append(",").append("'Kyoichi'").append(",").append(23).append("),");
        sb.append("(").append(10006).append(",").append("'Anneke'").append(",").append(19).append("),");

        sb.deleteCharAt(sb.length() - 1);
        sql = sql + sb.toString();
        System.out.println(sql);
        psmt.addBatch(sql);
        psmt.executeBatch();
        connection.close();
    }
}
```

#### 从 MySQL 实例中查询数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Select {
    private static String url = "jdbc:mysql://localhost:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "admin";

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
        System.out.println("empno \t ename \t age");
        System.out.println("----------------------------------------------------------------------");
        while(rs.next()){
            Integer empno = rs.getInt("emp_no");
            String ename = rs.getString("ename");
            Integer age = rs.getInt("age");

            System.out.println(empno + "\t" + ename + "\t" + age);
        }
        connection.close();
    }
}
```

#### 在 MySQL 实例中更新数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Update {
    private static String url = "jdbc:mysql://localhost:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "admin";

    public static void main(String[] args) throws SQLException {
        update();
    }

    public static void update() throws SQLException {
        MySQLConnection mysqlConnection = new MySQLConnection(url, username, password);

        Connection connection = mysqlConnection.getConnection();
        String sql = "update employee set age = ? where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 37);
        psmt.setInt(2, 10001);
        psmt.execute();

        connection.close();
    }
}
```

#### 在 MySQL 实例中删除数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Delete {
    private static String url = "jdbc:mysql://localhost:3306/company?useUnicode=true&characterEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "admin";

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
SequoiaSQL-MySQL 实例兼容 MySQL 语法，用户可通过 MySQL 的方式操作 SequoiaDB 数据库。
