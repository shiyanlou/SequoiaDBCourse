---
show: step
version: 7.0
---

## 课程介绍


本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 PostgreSQL 实例的环境中，使用 SQL 语句访问 SequoiaDB 数据库，完成对数据的增、删、查、改操作以及其它 PostgreSQL 语法操作。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括： 1 个 SequoiaSQL-PostgreSQL 数据库实例节点、1 个引擎协调节点，1 个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ef825173c9cd86053b61306ca6df9c65)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-PostgreSQL 实例均为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库和 SequoiaSQL-PostgreSQL 实例的操作系统用户为 sdbadmin。

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

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/6cccf5951f048e01b4789f3c08483bb0-0)

## 检查就绪状态

#### 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/810c1187bb311b8a506bdb6731e1f73f-0)

>Note:
>
>如果显示的节点数量与预期不符，请稍等初始化完成并重试该步骤。
> 
>C: 编目节点，S：协调节点，D：数据节点


#### 检查实例状态

查看 PostgreSQL 实例状态：

```shell
/opt/sequoiasql/postgresql/bin/sdb_sql_ctl status
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/ee64c7881af3a8f329bfb50848ed56e2)

## 创建数据库
1）在 SequoiaSQL-PostgreSQL 实例中并创建 company 数据库，为接下来验证 PostgreSQL 语法特性做准备；

以 sdbadmin 用户登录，在 PostgreSQL 实例创建数据库 company；
```shell
/opt/sequoiasql/postgresql/bin/sdb_sql_ctl createdb company myinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/41b0823324fdd49a4c108c44bbf919df)

2）查看数据库；
```shell
/opt/sequoiasql/postgresql/bin/psql -l
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/03c2315513eaca111dcec04b1b1d2871)

## 配置 PostgreSQL 实例
#### 加载 SequoiaDB 连接驱动
1）登录到 PostgreSQL 实例 Shell；
```shell
/opt/sequoiasql/postgresql/bin/psql -p 5432 company
```

2）加载SequoiaDB连接驱动；
```sql
CREATE EXTENSION sdb_fdw ;
```

#### 配置与 SequoiaDB 连接参数
在 PostgreSQL 实例中配置 SequoiaDB 连接参数：

1）创建 sdb_server 这个连接器
```sql
CREATE SERVER sdb_server FOREIGN DATA WRAPPER sdb_fdw OPTIONS (address '127.0.0.1', service '11810', user '', password '', preferedinstance 'A', transaction 'on' ) ;
```

>Note:
>
> - 如果没有配置数据库密码验证，可以忽略user与password字段；
> - 如果需要提供多个协调节点地址，options 中的 address 字段可以按格式 'ip1:port1,ip2:port2,ip3:port3'填写。此时，service 字段可填写任意一个非空字符串；
> - preferedinstance 设置 SequoiaDB 的连接属性。多个属性以逗号分隔，如：preferedinstance '1,2,A'；
> - transaction 设置 SequoiaDB 是否开启事务，默认为off。开启为on ；
> - 其他详细参数请参考 [PostgreSQL-连接说明](http://doc.sequoiadb.com/cn/SequoiaDB-cat_id-1432190715-edition_id-304) 

2）退出 PostgreSQL 客户端
```sql
\q
```

## 创建关联集合空间、集合
首先在 SequoiaDB 巨杉数据库引擎中创建集合空间 company 和集合 employee，然后在 PostgreSQL 实例中关联 SequoiaDB 数据库引擎中创建的集合空间和集合。

#### SequoiaDB 数据库引擎中创建集合

1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```
2）通过 JavaScript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```

3）创建 company_domains 逻辑域；

```javascript
db.createDomain ("company_domains", ["group1", "group2", "group3"], { AutoSplit : true } ) ;
```
4）创建 company 集合空间；
```javascript
db.createCS ("company", { Domain : "company_domains" } ) ;
```

5）创建 employee 集合；
```javascript
db.company.createCL ("employee", { "ShardingKey" : { "_id" : 1} , "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```
6）退出 SequoiaDB Shell；

```javascript
quit ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/283c4851aaf4bb04fea3f47408243b03)

#### 实例与数据引擎中集合关联

1）登录到 PostgreSQL 实例 Shell；

```shell
/opt/sequoiasql/postgresql/bin/psql -p 5432 company
```

2）创建 PostgreSQL 实例中的外表并与 SequoiaDB 中的集合空间和集合关联；

```sql
CREATE FOREIGN TABLE employee (
     empno INT,
     ename VARCHAR(128),
     age INT
  ) 
  SERVER sdb_server OPTIONS ( collectionspace 'company', collection 'employee', decimal 'on' ) ;
```

3）检查 PostgreSQL 中创建的表；
```sql
\d
```
操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/f69ea401a8a33279cd6e6eaf6882cb74-0)

>Note:
>
> - 集合空间与集合必须已经存在于 SequoiaDB，否则查询出错；
> - 如果需要对接 SequoiaDB 的 decimal 字段，则需要在 options 中指定 decimal 'on' 。
> - 默认情况下，表的字段映射到 SequoiaDB 中为小写字符，如果强制指定字段为大写字符，需要将字段名用双引号引起来；
> - 如果字段名或者字段类型有 PostgreSql 的关键字也是创建不成功表的，需要把关键字用双引号引起来。
> - 其他详细参数请参考 [PostgreSQL-连接说明](http://doc.sequoiadb.com/cn/SequoiaDB-cat_id-1432190715-edition_id-304) 



## 实例操作数据库引擎集合的数据
使用 PostgreSQL 实例操作关联表中的数据。

#### 通过关联表插入数据
在 PostgreSQL 实例中向外表 employee 中插入数据:

```sql
INSERT INTO employee VALUES (10001, 'Georgi', 48) ;
INSERT INTO employee VALUES (10002, 'Bezalel', 21) ;
INSERT INTO employee VALUES (10003, 'Parto', 33) ;
INSERT INTO employee VALUES (10004, 'Chirstian', 40) ;
INSERT INTO employee VALUES (10005, 'Kyoichi', 23) ;
INSERT INTO employee VALUES (10006, 'Anneke', 19) ;
```

#### 查询 employee 表中的数据
查询 PostgreSQL 实例外表 employee 中 age 大于20，小于30的数据：
```sql
SELECT * FROM employee WHERE age > 20 AND age < 30 ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/7e80bc92b094955b27b09cf310249119)


#### 更新关联表中的数据

1）更新 PostgreSQL 实例外表 employee 中的数据，将 empno 为10001的记录 age 更改为34；

```sql
UPDATE employee SET age=34 WHERE empno = 10001 ;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功：

```sql
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/d822b2d32288d17baf8fc6c99a75c514)

#### 删除关联表中的数据
1）删除 PostgreSQL 实例外表 employee 中的数据，将 empno 为 10006 的记录删除；
```sql
DELETE FROM employee WHERE empno = 10006 ;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除；
```sql
SELECT * FROM employee ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/31aea8f7e0cc92d6a82283c433929d76)

3）退出 PostgreSQL 客户端

```sql
\q
```

## 索引使用
通过 PostgreSQL 实例查看执行计划及通过过滤条件查看 SequoiaDB 执行计划。

#### 索引使用
1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```

2）通过 JavaScript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```

3）进入 SequoiaDB Shell，在 SequoiaDB 数据库引擎集合 employee 的 ename 字段上创建索引；

```javascript
db.company.employee.createIndex ("idx_ename", { ename : 1 }, false) ;  
```

4）查看执行计划；

```javascript
db.company.employee.find ({ "ename": { "$et": "Georgi" } }).explain() ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/a9ea5e8421244d5010ff08fcc7019e15)

退出 SequoiaDB Shell；

```
quit ;
```

5）登录到 PostgreSQL 实例 Shell；
```shell
/opt/sequoiasql/postgresql/bin/psql -p 5432 company
```

6）在 PostgreSQL 实例查看表 employee 查询语句执行计划，看到此查询已经使用索引；

```sql
EXPLAIN SELECT * FROM employee WHERE ename = 'Georgi' ;
```

退出 PostgreSQL 客户端
```sql
\q
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/6c240851d0dbd8d67c6a5ecfbd2646bc)



## Java 语言操作 PostgreSQL 实例中的数据
本节内容主要用来演示 Java 语言操作 SequoiaSQL-PostgreSQL 实例中的数据，为相关开发人员提供参考。源码已经放置在 /home/sdbadmin/source/postgresql 目录下。

1）进入源码放置目录；
```shell
cd /home/sdbadmin/source/postgresql
```

2）查看 java 文件，一共5个 java 文件；
```shell
ls -trl
```

>Note：
>
> - PostgreSQLConnection.java   连接数据库类
> - Insert.java                 写入数据类
> - Select.java                 查询数据类
> - Update.java                 更新数据类
> - Delete.java                 删除数据类

3）首先对连接的 java 文件进行编译；

```shell
javac -d . PostgreSQLConnection.java
```

#### 在 PostgreSQL 实例中插入数据：
1）增加 empno 为 30001 、 30002 和 30003 这三条记录，Insert.java 源代码如下；
```java
package com.sequoiadb.postgresql;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class Insert {
    private static String url = "jdbc:postgresql://127.0.0.1:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";
    public static void main(String[] args) throws SQLException {
        insert();
    }

    public static void insert() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, passwor
d);
        Connection connection = pgConnection.getConnection();
        String sql = "INSERT INTO employee VALUES";
        Statement stmt = connection.createStatement();
        StringBuffer sb = new StringBuffer();
        sb.append("(").append(30001).append(",").append("'Mike'").append(",").append(20).ap
pend("),");
        sb.append("(").append(30002).append(",").append("'Donna'").append(",").append(21).a
ppend("),");
        sb.append("(").append(30003).append(",").append("'Jack'").append(",").append(22).ap
pend("),");
        sb.deleteCharAt(sb.length() - 1);
        sql = sql + sb.toString();
        System.out.println(sql);
        stmt.executeUpdate(sql);
        connection.close();
    }
}
```

2）对 Insert.java 文件进行编译；

```shell
javac -d . Insert.java
```

3）运行 Insert 类的代码；

```shell
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Insert
```

4）操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/7e39b84aa42b4fe40a38045cbf2b2591-0)


#### 从 PostgreSQL 实例中查询数据：
1）查询返回 empno 、 ename 和 age 这三个字段 ，Select.java 源代码如下；
```java
package com.sequoiadb.postgresql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Select {
    private static String url = "jdbc:postgresql://127.0.0.1:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        select();
    }

    public static void select() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, passwor
d);
        Connection connection = pgConnection.getConnection();
        String sql = "select * from employee";
        PreparedStatement psmt = connection.prepareStatement(sql);
        ResultSet rs = psmt.executeQuery();
        System.out.println("---------------------------------------------------------------
-------");
        System.out.println("empno \t ename \t age");
        System.out.println("---------------------------------------------------------------
-------");
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
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Select
```

4）操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/5377b73da5f8ca47d060e96ce999b936-0)


#### 在 PostgreSQL 实例中更新数据：
1）将 empno 值为 10004 的 age 修改为 41 ， Update.java 源代码如下；
```java
package com.sequoiadb.postgresql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Update {
    private static String url = "jdbc:postgresql://127.0.0.1:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        update();
    }

    public static void update() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, passwor
d);
        Connection connection = pgConnection.getConnection();
        String sql = "update employee set age = ? where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 41);
        psmt.setInt(2, 10004);
        psmt.execute();

        connection.close();
    }
}
```
2）对 Update.java 文件进行编译；

```shell
javac -d . Update.java
```

3）运行 Update 类的代码；

```shell
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Update
```
4）更新后查看 empno 为 10004 的 age 值是否更新为 41；

```shell
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Select
```

5）操作截图 ：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3fe5936fbd18b68ec2e92fa365c623e9-0)

#### 在 PostgreSQL 实例中删除数据
1）将 empno 值为 10005 的记录删除 ,  Delete.java 源代码如下；
```java
package com.sequoiadb.postgresql;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Delete {
    private static String url = "jdbc:postgresql://127.0.0.1:5432/company";
    private static String username = "sdbadmin";
    private static String password = "";

    public static void main(String[] args) throws SQLException {
        delete();
    }

    public static void delete() throws SQLException {
        PostgreSQLConnection pgConnection = new PostgreSQLConnection(url, username, passwor
d);
        Connection connection = pgConnection.getConnection();
        String sql = "delete from employee where empno = ?";
        PreparedStatement psmt = connection.prepareStatement(sql);
        psmt.setInt(1, 10005);
        psmt.execute();
        connection.close();
    }
}
```

2）对 Delete.java 文件进行编译；

```shell
javac -d . Delete.java
```

3）运行 Update 类的代码；

```shell
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Delete
```

4）检查确认 empno 为 10005 的雇员信息已经被删除；

```shell
 java -cp .:../postgresql-9.3-1104-jdbc41.jar com.sequoiadb.postgresql.Select
```

5）操作截图；
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/4af36643e5d00856da956dd2e6d39adc-0)

## 总结

通过本课程，我们验证了 SequoiaDB 巨杉数据库所支持的 PostgreSQL 语法，并展示了使用 JAVA 语言对 SequoiaSQL-PostgreSQL 实例中的表进行 CRUD 操作。可以看出：
- SequoiaSQL-PostgreSQL 实例兼容标准的 PostgreSQL 语法；
- Java语言操作 SequoiaSQL-PostgreSQL 实例中的数据与操作原生 PostgreSQL 中的数据无任何差异，可做到无缝切换；
