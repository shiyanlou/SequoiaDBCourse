---
show: step
version: 14.0 
---

## 课程介绍

本课程将介绍使用 JavaScript 脚本开发语言在 SequoiaDB Shell 中进行创建数据域、集合空间、集合、索引等操作，随后使用 JAVA 驱动对创建的集合进行写入和查询操作。


#### JSON 实例简介
SequoiaDB 巨杉数据库为用户提供了 JSON 实例, 通过此实例可以与 SequoiaDB 巨杉数据库进行交互操作。JSON 实例支持多种方式执行集群管理、运行实例检查、数据增删改查等操作。


#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1480/1207281/96cb907f16094f2f959938fe26df8546-0)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本。

## 切换用户及查看数据库版本

#### 切换到 sdbadmin 用户

部署 SequoiaDB 巨杉数据库的操作系统用户为 sdbadmin。

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

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量少于上图中的数量，请稍等初始化完成并重试该步骤。

## 创建域、集合空间和集合
在 SequoiaDB Shell 中创建集合空间和集合用于后续章节进行数据操作。

1）通过 Linux 命令行进入 SequoiaDB Shell；

```shell
sdb
```

2）通过 javascript 语言连接协调节点，获取数据库连接；

```javascript
var db = new Sdb ("localhost",11810) ;
```

3）创建 company_domain 逻辑域；

```javascript
db.createDomain ("company_domain", ["group1", "group2", "group3"], { AutoSplit : true }) ;
```

4）创建 company 集合空间；

```javascript
db.createCS ("company", { Domain : "company_domain" }) ;
```

5）创建 employee 集合；

```javascript
db.company.createCL ("employee", {"ShardingKey" : { "_id" : 1} , "ShardingType" : "hash" , "ReplSize" : -1 , "Compressed" : true , "CompressionType" : "lzw" , "AutoSplit" : true , "EnsureShardingIndex" : false }) ;
```


>Note:
>
> - 集合（Collection）是数据库中存放文档的逻辑对象。任何一条文档必须属于一个且仅一个集合。
> - 集合空间（CollectionSpace）是数据库中存放集合的物理对象。任何一个集合必须属于一个且仅一个集合空间。
> - 域（Domain）是由若干个复制组（ReplicaGroup）组成的逻辑单元。每个域都可以根据定义好的策略自动管理所属数据，如数据切片和数据隔离等。
>


## 集合数据操作
通过 SequoiaDB Shell 进行集合数据的 CRUD 操作。

#### 集合中写入数据
在 JSON 实例集合 company 中写入数据：
```javascript
db.company.employee.insert ({ "empno" : 10001 , "ename" : "Georgi" , "age" : 48 }) ;
db.company.employee.insert ({ "empno" : 10002 , "ename" : "Bezalel" , "age" : 21 }) ;
db.company.employee.insert ({ "empno" : 10003 , "ename" : "Parto" , "age" : 33 }) ;
db.company.employee.insert ({ "empno" : 10004 , "ename" : "Chirstian" , "age" : 40 }) ;
db.company.employee.insert ({ "empno" : 10005 , "ename" : "Kyoichi" , "age" : 23 }) ;
db.company.employee.insert ({ "empno" : 10006 , "ename" : "Anneke" , "age" : 19 }) ;
```

#### 查询集合中的数据
查询集合 employees 中age 大于20，小于30的数据：
```javascript
db.company.employee.find ( { "age" : { "$gt" : 20 , "$lt" : 30 } } ) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/95b0770b9305772d6c795fe29a0b02d6)

#### 更新集合中的数据
1）集合 employees 中的数据，将 empno 为10001的记录 age 更改为34；

```javascript
db.company.employee.update ( { "$set" : { "age" : 34 } } , { "empno" : 10001 }) ;
```

2）查询数据结果确认 empno 为10001的记录更新是否成功；

```javascript
db.company.employee.find ( { "empno" : 10001 } ) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/39b91f46f3ce79d27b342f224e4a8535)

#### 删除集合中的数据
1）删除集合 employees 中的数据，将 empno 为10006的记录删除；

```javascript
db.company.employee.remove ( { "empno" : 10006 } ) ;
```

2）查询数据结果确认 empno 为10006的记录是否成功删除；

```javascript
db.company.employee.find ({},{"empno":""}) ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/b7d47eedfcfbf827afc606f55af5565e)


## 索引使用
在 SequoiaDB 巨杉数据库中，索引是一种特殊的数据对象。索引本身不做为保存用户数据的容器，而是作为一种特殊的元数据，提高数据访问的效率。

1）在集合 employee 的 ename 字段上创建索引；
```javascript
db.company.employee.createIndex ("idx_ename", { ename : 1 }, false) ;
```

2）查看集合 employee 上创建的索引；
```javascript
db.company.employee.listIndexes () ;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/668e701adf5c780653096b32391a9f4c)

3）显示集合 employees 查询语句执行计划；

```javascript
db.company.employee.find ( { "ename" : "Georgi" } ).explain() ;
```

操作截图：
![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/0afc05df8deddc2ac5b285768c0b372e)

4）退出 SequoiaDB Shell；

```javascript
quit ;
```

##  创建 JAVA 工程项目

使用 Java 语言操作 JSON 实例中的数据，为相关开发人员提供参考，更多开发语言的操作请参考官网提供的驱动文档。

1）创建 JAVA 工程目录；

```shell
mkdir -p /home/sdbadmin/json/lib
```

2）进入工程目录；
```shell
cd /home/sdbadmin/json
```

3）拷贝 JSON 实例驱动包；
```shell
cp /opt/sequoiadb/java/sequoiadb-driver-3.4.jar ./lib
```

4）复制以下代码到实验环境终端执行，生成 Datasource.java 文件；

```shell
cat > /home/sdbadmin/json/Datasource.java << EOF

package com.sequoiadb.samples;

import java.util.ArrayList;
import org.bson.BSONObject;
import org.bson.BasicBSONObject;
import com.sequoiadb.base.CollectionSpace;
import com.sequoiadb.base.ConfigOptions;
import com.sequoiadb.base.DBCollection;
import com.sequoiadb.base.DBCursor;
import com.sequoiadb.base.Sequoiadb;
import com.sequoiadb.datasource.ConnectStrategy;
import com.sequoiadb.datasource.DatasourceOptions;
import com.sequoiadb.datasource.SequoiadbDatasource;
import com.sequoiadb.exception.BaseException;

public class Datasource {

    public static void main(String[] args) throws InterruptedException {
        ArrayList<String> addrs = new ArrayList<String>();
        String user = "";
        String password = "";
        ConfigOptions nwOpt = new ConfigOptions();
        DatasourceOptions dsOpt = new DatasourceOptions();
        SequoiadbDatasource ds = null;

        // 提供coord节点地址
        addrs.add("sdbserver1:11810");

        // 设置网络参数
        nwOpt.setConnectTimeout(500); // 建连超时时间为500ms。
        nwOpt.setMaxAutoConnectRetryTime(0); // 建连失败后重试时间为0ms。

        // 设置连接池参数
        dsOpt.setMaxCount(500); // 连接池最多能提供500个连接。
        dsOpt.setDeltaIncCount(20); // 每次增加20个连接。
        dsOpt.setMaxIdleCount(20); // 连接池空闲时，保留20个连接。
        dsOpt.setKeepAliveTimeout(0); // 池中空闲连接存活时间。单位:毫秒。
                                        // 0表示不关心连接隔多长时间没有收发消息。
        dsOpt.setCheckInterval(60 * 1000); // 每隔60秒将连接池中多于
                                            // MaxIdleCount限定的空闲连接关闭，
                                            // 并将存活时间过长（连接已停止收发
                                            // 超过keepAliveTimeout时间）的连接关闭。
        dsOpt.setSyncCoordInterval(0); // 向catalog同步coord地址的周期。单位:毫秒。
                                        // 0表示不同步。
        dsOpt.setValidateConnection(false); // 连接出池时，是否检测连接的可用性，默认不检测。
        dsOpt.setConnectStrategy(ConnectStrategy.BALANCE); // 默认使用coord地址负载均衡的策略获取连接。

        // 建立连接池
        ds = new SequoiadbDatasource(addrs, user, password, nwOpt, dsOpt);

        // 使用连接池运行任务
        runTask(ds);

        // 任务结束后，关闭连接池
        ds.close();
    }

    static void runTask(SequoiadbDatasource ds) throws InterruptedException {
        String clFullName = "company.employee";
        // 准备任务

        Thread insertTask = new Thread(new InsertTask(ds, clFullName));
        Thread queryTask = new Thread(new QueryTask(ds, clFullName));

        // 往集合插记录
        insertTask.start();
        Thread.sleep(3000);

        // 从集合中查记录
        queryTask.start();

        // 等待任务结束
        insertTask.join();
        queryTask.join();
    }
}

class InsertTask implements Runnable {
    private SequoiadbDatasource ds;
    private String csName;
    private String clName;

    public InsertTask(SequoiadbDatasource ds, String clFullName) {
        this.ds = ds;
        this.csName = clFullName.split("\\\.")[0];
        this.clName = clFullName.split("\\\.")[1];
    }

    @Override
    public void run() {
        Sequoiadb db = null;
        CollectionSpace cs = null;
        DBCollection cl = null;
        BSONObject record = null;
        // 从连接池获取连接
        try {
            db = ds.getConnection();
        } catch (BaseException e) {
            e.printStackTrace();
            System.exit(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.exit(1);
        }

        // 使用连接获取集合对象
        cs = db.getCollectionSpace(csName);
        cl = cs.getCollection(clName);
        // 使用集合对象插入记录
        record = genRecord();
        cl.insert(record);
        // 将连接归还连接池
        ds.releaseConnection(db);
        System.out.println("Suceess to insert record: " + record.toString());
    }

    private BSONObject genRecord() {
        BSONObject obj = new BasicBSONObject();
        obj.put("empno", 10007);
        obj.put("ename", "JACK");
        obj.put("age", 30);
        return obj;
    }
}

class QueryTask implements Runnable {
    private SequoiadbDatasource ds;
    private String csName;
    private String clName;

    public QueryTask(SequoiadbDatasource ds, String clFullName) {
        this.ds = ds;
        this.csName = clFullName.split("\\\.")[0];
        this.clName = clFullName.split("\\\.")[1];
    }

    @Override
    public void run() {
        Sequoiadb db = null;
        CollectionSpace cs = null;
        DBCollection cl = null;
        DBCursor cursor = null;
        // 从连接池获取连接
        try {
            db = ds.getConnection();
        } catch (BaseException e) {
            e.printStackTrace();
            System.exit(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.exit(1);
        }
        // 使用连接获取集合对象
        cs = db.getCollectionSpace(csName);
        cl = cs.getCollection(clName);
        // 使用集合对象查询
        cursor = cl.query();
        try {
            while (cursor.hasNext()) {
                System.out.println("The inserted record is: " + cursor.getNext());
            }
        } finally {
            cursor.close();
        }
        // 将连接对象归还连接池
        ds.releaseConnection(db);
    }
}
EOF
```

5）查询是否已经生成 Datasource.java 文件；

```shell
ls -trl /home/sdbadmin/json/Datasource.java
```

## 编译运行代码

上一小节已经创建了 JAVA 工程和代码并且拷贝了 JAVA 驱动，接下来我们对代码进行编译运行。

1）编译 Datasource.java 文件；

```shell
javac -cp .:./lib/sequoiadb-driver-3.4.jar -d . Datasource.java
```

2）运行 Datasource 类代码，查询数据；
```shell
java -cp .:./lib/sequoiadb-driver-3.4.jar com.sequoiadb.samples.Datasource
```

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/1bc8826733633657839e812a3297d4d2-0)

## 总结

本课程通过 javascript 语法对 SequoiaDB 巨杉数据库 JSON 实例进行了创建集合空间、集合、索引以及数据的 CRUD 基本操作，通过 JAVA 开发语言展示了连接池和集合的写入和查询操作。





