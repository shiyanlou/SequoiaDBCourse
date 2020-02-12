---
show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 PostgreSQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其他 PostgreSQL 语法操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-PostgreSQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ef825173c9cd86053b61306ca6df9c65)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-PostgreSQL 实例均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-PostgreSQL 实例的操作系统用户为 sdbadmin。
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

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1d1b4057ef81bc03b825926d3071183a)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表

```
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤

检查 PostgreSQL 实例
```
/opt/sequoiasql/postgresql/bin/sdb_sql_ctl listinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ee64c7881af3a8f329bfb50848ed56e2)

## 创建数据库
1）在 SequoiaSQL-PostgreSQL 实例中并创建 company 数据库实例，为接下来验证 MySQL 语法特性做准备。

以 sdbadmin 用户登录，在 PostgreSQL 实例创建数据库 company：
```
/opt/sequoiasql/postgresql/bin/sdb_sql_ctl createdb company myinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/41b0823324fdd49a4c108c44bbf919df)

2）查看数据库：
```
/opt/sequoiasql/postgresql/bin/psql -l
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/03c2315513eaca111dcec04b1b1d2871)

## 配置 PostgreSQL 实例
#### 加载 SequoiaDB 连接驱动
1）登录到 PostgreSQL 实例 Shell：
```
/opt/sequoiasql/postgresql/bin/psql -p 5432 company
```

2）加载SequoiaDB连接驱动
```sql
CREATE EXTENSION sdb_fdw ;
```

#### 配置与 SequoiaDB 连接参数
在 PostgreSQL 实例中配置 SequoiaDB 连接参数：
```sql
CREATE SERVER sdb_server 
    FOREIGN DATA WRAPPER sdb_fdw 
    OPTIONS (address '127.0.0.1', service '11810', user '', password '', preferedinstance 'A', transaction 'off' ) ;
```

>Note:
>
> - 如果没有配置数据库密码验证，可以忽略user与password字段。 
> - 如果需要提供多个协调节点地址，options 中的 address 字段可以按格式 'ip1:port1,ip2:port2,ip3:port3'填写。此时，service 字段可填写任意一个非空字符串。
> - preferedinstance 设置 SequoiaDB 的连接属性。多个属性以逗号分隔，如：preferedinstance '1,2,A'。详细配置请参考 preferedinstance 取值
> - preferedinstancemode 设置 preferedinstance 的选择模式
> - sessiontimeout 设置会话超时时间 如：sessiontimeout '100' 
> - transaction 设置 SequoiaDB 是否开启事务，默认为off。开启为on 
> - cipher 设置是否使用加密文件输入密码，默认为off。开启为on 
> - token 设置加密口令 
> - cipherfile 设置加密文件，默认为 ./passwd 

## 关联集合空间、集合
在 PostgreSQL 实例中关联 SequoiaDB 数据库中的集合空间、集合。

#### 在 PostgreSQL 实例中创建外部表
进入 SequoiaDB Shell，在 SequoiaDB 中创建集合空间 company，集合 employee：

1）通过 Linux 命令行进入 SequoiaDB Shell；
```
sdb
```
1）通过 javascript 语言连接协调节点，获取数据库连接；
```javascript
var db = new Sdb ("localhost",11810) ;
```

2）创建 company_domains 逻辑域；

```javascript
db.createDomain ("company_domains", ["group1", "group2", "group3"], { AutoSplit : true }) ;
```
3）创建 company 集合空间；
```javascript
db.createCS ("company",{Domain:"company_domains"}) ;
```

4）创建 employee 集合；
```javascript
db.company.createCL ("employee", {"ShardingKey" : { "_id" : 1} , "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/283c4851aaf4bb04fea3f47408243b03)

#### PostgreSQL 实例外部表与 SequoiaDB 集合空间、集合关联
将 PostgreSQL 实例中的外表并与 SequoiaDB 中的集合空间、集合关联：
```sql
CREATE FOREIGN TABLE employee (
     empno INT,
     ename VARCHAR(128),
     age INT
) SERVER sdb_server
OPTIONS ( collectionspace 'company', collection 'employee', decimal 'on' ) ;
```

>Note:
>
> - 集合空间与集合必须已经存在于 SequoiaDB，否则查询出错。
> - 如果需要对接 SequoiaDB 的 decimal 字段，则需要在 options 中指定 decimal 'on' 。
> - pushdownsort 设置是否下压排序条件到 SequoiaDB，默认为 on，关闭为 off。
> - pushdownlimit 设置是否下压 limit 和 offset 条件到 SequoiaDB，默认为on，关闭为off。
> - 开启 pushdownlimit 时，必须同时开启 pushdownsort ，否则可能会造成结果非预期的问题。
> - 默认情况下，表的字段映射到 SequoiaDB 中为小写字符，如果强制指定字段为大写字符，创建方式参考“注意事项1”。
> - 映射 SequoiaDB 的数组类型，创建方式参考“注意事项2”。

## 关联表数据操作
使用 PostgreSQL 实例操作关联表中的数据。

#### 通过关联表插入数据
在 PostgreSQL 实例外表 employee 中插入数据：
```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
INSERT INTO employee VALUES (10005, 'Kyoichi', 23) ;
INSERT INTO employee VALUES (10006, 'Anneke', 19) ;
```

#### 插入关联表中的数据
查询 PostgreSQL 实例外表 employee 中 age 大于20，小于30的数据：
```sql
SELECT * FROM employee WHERE age > 20 AND age < 30 ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/7e80bc92b094955b27b09cf310249119)


#### 更新关联表中的数据
更新 PostgreSQL 实例外表 employee 中的数据，将 empno 为10001的记录 age 更改为34：
```SQL
UPDATE employee SET age=34 WHERE empno=10001 ;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功：
```SQL
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/d822b2d32288d17baf8fc6c99a75c514)

#### 删除关联表中的数据
1）删除 PostgreSQL 实例外表 employee 中的数据，将 empno 为10006的记录删除：：
```SQL
DELETE FROM employee WHERE empno=10006 ;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除：
```SQL
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/31aea8f7e0cc92d6a82283c433929d76)

## 索引使用
通过 PostgreSQL 实例查看执行计划及通过过滤条件查看 SequoiaDB 执行计划。

#### 索引使用
1）进入 SequoiaDB Shell，在 SequoiaDB 集合 employee 的ename字段上创建索引：
```javascript
db.company.employee.createIndex ("idx_ename", { ename : 1 }, false) ;  
```

2）在 PostgreSQL 实例显示表 employee 查询语句执行计划：
```SQL
EXPLAIN SELECT * FROM employee WHERE ename = 'Georgi' ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/6c240851d0dbd8d67c6a5ecfbd2646bc)

3）进入 SequoiaDB Shell，根据上述输出中的Filter，在 SequoiaDB 中显示集合 employee 查询语句执行计划：
```javascript
db.company.employee.find ({ "ename": { "$et": "Georgi" } }).explain() ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/a9ea5e8421244d5010ff08fcc7019e15)

## Java 语言操作 PostgreSQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaDB-PostgreSQL 实例中的数据，为相关开发人员提供参考。

#### 连接 PostgreSQL 实例：
```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class PostgreSQLConnection {

    private String url;
    private String username;
    private String password;
    private String driver = "org.postgresql.Driver";

    public PostgreSQLConnection(String url, String username, String password){
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

#### 在 PostgreSQL 实例中插入数据：
```
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class Insert {
    private static String url = "jdbc:postgresql://localhost:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";
    public static void main(String[] args) throws SQLException {
        insert();
    }

    public static void insert() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, password);
        Connection connection = pgConnection.getConnection();
        String sql = "INSERT INTO employee VALUES";
        Statement stmt = connection.createStatement();
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
        stmt.executeUpdate(sql);
        connection.close();
    }
}
```

#### 从 PostgreSQL 实例中查询数据：
```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Select {
    private static String url = "jdbc:postgresql://localhost:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        select();
    }

    public static void select() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, password);
        Connection connection = pgConnection.getConnection();
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

#### 在 PostgreSQL 实例中更新数据：
```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Update {
    private static String url = "jdbc:postgresql://localhost:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        update();
    }

    public static void update() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, password);
        Connection connection = pgConnection.getConnection();
        String sql = "update employee set age = ? where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 37);
        psmt.setInt(2, 10001);
        psmt.execute();

        connection.close();
    }
}
```

#### 在 PostgreSQL 实例中删除数据：
```
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Delete {
    private static String url = "jdbc:postgresql://localhost:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        delete();
    }

    public static void delete() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, password);
        Connection connection = pgConnection.getConnection();
        String sql = "delete from employee where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 10006);
        psmt.execute();
        connection.close();
    }
}
```


## 总结


通过本课程，我们通过 MySQL 语法在 SequoiaSQL-MySQL 实例上创建数据库和数据表，并对数据表进行了 CRUD 的基本数据操作；同时展示了使用 JAVA 语言对数据表进行数据操作。可以看出：
- SequoiaSQL-PostgreSQL 实例兼容标准的 PostgreSQL 语法；
- Java语言操作 SequoiaSQL-PostgreSQL 实例中的数据与操作原生 PostgreSQL 中的数据无任何差异，可做到无缝切换；
