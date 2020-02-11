---
show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他MySQL 语法操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。

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

![图片描述](images/710-sdbversion.png)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist 
```

操作截图：

![图片描述](images/710-sdblist.png)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤

检查 MySQL 实例
```
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图：

![图片描述](images/710-listmysqlinst.png)

## 创建数据库及数据表
进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，为接下来验证 MySQL 语法特性做准备。

#### 登录 MySQL 实例 Shell
```
mysql -h 127.0.0.1 -P 3306 -u root -p
```

#### 创建数据库
在MySQL实例中创建新数据库company，并切换至company库：
```
create database company;
use company;
```

#### 创建分区表
1）在 MySQL 实例 company 数据库中创建分区表 employee：
```
CREATE TABLE employee (
	empno INT,
	ename VARCHAR(128),
	age INT,
	PRIMARY KEY (empno)
)ENGINE=sequoiadb COMMENT="雇员表, sequoiadb:{ table_options: { ShardingKey: { 'empno': 1 }, ShardingType:'hash','Compressed':true,'CompressionType':'lzw','AutoSplit':true,'EnsureShardingIndex':false } }";
```

>Note:
>
>[MySQL 实例参数配置](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1566551297-edition_id-0)

2）查看 MySQL 实例分区表结构：
```
show create table employee;
```

操作截图：

![图片描述](images/710-showcreatetable.png)


#### 分区表中插入数据
在分区表 employee 中插入数据：
```
INSERT INTO employee VALUES (10001,'Georgi',48);
INSERT INTO employee VALUES (10002,'Bezalel',21);
INSERT INTO employee VALUES (10003,'Parto',33);
INSERT INTO employee VALUES (10004,'Chirstian',40);
INSERT INTO employee VALUES (10005,'Kyoichi',23);
INSERT INTO employee VALUES (10006,'Anneke',19);
```

#### 查询分区表中的数据
查询分区表 employee 中 age 大于20，小于30的数据：
```
select * from employee where age > 20 and age < 30;
```

操作截图：

![图片描述](images/710-select.png)


#### 更新分区表中的数据
1）更新分区表 employee 中的数据，将 empno 为10001的记录 age 更改为34：
```
update employee set age=34 where empno=10001;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功：
```
select * from employee;
```

操作截图：

![图片描述](images/710-update.png)

#### 删除分区表中的数据
1）删除分区表 employees 中的数据，将 empno 为10006的记录删除：
```
delete from employee where empno=10006;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除：
```
select * from employee;
```

操作截图：

![图片描述](images/710-delete.png)

#### 分区表中索引使用
1）在分区表 employee 的 ename 字段上创建索引：
```
alter table employee add index idx_ename(ename);
```

2）显示分区表 employee 查询语句执行计划：
```
explain select * from employee where ename = 'Georgi';
```

操作截图：

![图片描述](images/710-createandshowindex.png)


## Java 语言操作 MySQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaDB-MySQL 实例中的数据，为相关开发人员提供参考。

#### 连接 MySQL 实例
```
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
```
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
```
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
```
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
```
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

通过本课程，我们验证了 SequoiaDB 巨杉数据库所支持的 MySQL 语法，并对底层数据存储分布进行了直接验证。可以看出：
- SequoiaSQL-MySQL 实例 100% 兼容标准的 MySQL 语法；
- SequoiaDB 巨杉数据库底层存储为分布式架构，数据可均匀分布在多个分区中；
- Java语言操作 SequoiaSQL-MySQL 实例中的数据与操作原生 MySQL 中的数据无任何差异，可做到无缝切换；
