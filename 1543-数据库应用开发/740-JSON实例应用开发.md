---
show: step
version: 2.2
---

## 课程介绍
SequoiaDB 巨杉数据库为用户提供了 JSON 实例, 支持多种方式执行集群管理、运行实例检查、数据增删改查等操作。本课程将带领您在已经部署 SequoiaDB 巨杉数据库环境中，使用 JAVA 开发语言连接 JSON 实例实现创建域、集合空间、集合和数据操作操作。

JSON 实例支持多种 SDK 驱动开发，请参考：
* [SequoiaDB 驱动](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)


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

查看 SequoiaDB 巨杉数据库引擎节点列表

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/3ebdc835c21b5685d858918d25a9f372)

>Note:
>
>如果显示的节点数量少于上图中的数量，请稍等初始化完成并重试该步骤。


##  创建 JAVA 工程项目

使用 Java 语言操作 JSON 实例中的数据，为相关开发人员提供参考。

1）创建 JAVA 工程目录；

```shell
mkdir -p /home/sdbadmin/json/lib
```

2）拷贝 JSON 实例驱动包；
```shell
cp /opt/sequoiadb/java/sequoiadb-driver-3.4.jar /home/sdbadmin/json/lib
```

3）查看 sequoiadb-driver-3.4.jar 驱动文件包, 开始创建数据源；

```shell
ls -trl /home/sdbadmin/json/lib
```


4）复制以下代码到实验环境终端执行，生成Datasource.java 文件；

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
import com.sequoiadb.base.Domain;
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
		addrs.add("192.168.75.129:11810");

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
		Thread createCLTask = new Thread(new CreateCLTask(ds, clFullName));
		Thread insertTask = new Thread(new InsertTask(ds, clFullName));
		Thread queryTask = new Thread(new QueryTask(ds, clFullName));

		// 创建集合
		createCLTask.start();
		createCLTask.join();

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

class CreateCLTask implements Runnable {
	private SequoiadbDatasource ds;
	private String csName;
	private String clName;

	public CreateCLTask(SequoiadbDatasource ds, String clFullName) {
		this.ds = ds;
		this.csName = clFullName.split("\\\.")[0];
		this.clName = clFullName.split("\\\.")[1];
	}

	@Override
	public void run() {
		Sequoiadb db = null;
		CollectionSpace cs = null;
		DBCollection cl = null;
		// 从连接池获取连接池
		try {
			db = ds.getConnection();
		} catch (BaseException e) {
			e.printStackTrace();
			System.exit(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
			System.exit(1);
		}
		// 使用连接创建集合
		if (db.isCollectionSpaceExist(csName))
			db.dropCollectionSpace(csName);

		BSONObject dmOptions = new BasicBSONObject();
		BSONObject csOptions = new BasicBSONObject();
		BSONObject clOptions = new BasicBSONObject();

		ArrayList<String> groupList = new ArrayList<String>();
		groupList.add("group1");
		groupList.add("group2");
		groupList.add("group3");

		dmOptions.put("Groups", groupList);
		dmOptions.put("AutoSplit", true);

		Domain domain = db.createDomain("company_domain", dmOptions);

		csOptions.put("Domain", "company_domain");

		clOptions.put("ShardingKey", new BasicBSONObject("empno", 1));
		clOptions.put("AutoSplit", true);// 本示例其他参数均使用默认值

		cs = db.createCollectionSpace(csName, csOptions);
		cl = cs.createCollection(clName, clOptions);
		// 将连接归还连接池
		ds.releaseConnection(db);
		System.out.println("Suceess to create collection " + csName + "." + clName);
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
		obj.put("empno", 1);
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

1）进入工程项目目录；

```shell
cd  /home/sdbadmin/json
```

2）编译 Datasource.java 文件；

```shell
javac -cp .:./lib/sequoiadb-driver-3.4.jar -d . Datasource.java
```

3）运行 Select 类代码，查询数据；
```shell
java -cp .:./lib/sequoiadb-driver-3.4.jar com.sequoiadb.samples.Datasource
```

![图片描述](https://doc.shiyanlou.com/courses/1543/1207281/170c9fb0f6c0cc01573ccd708c785b44-0)

## 总结

本课程使用 JAVA 语言对 SequoiaDB 巨杉数据库的 JSON 实例进行了创建数据域、集合空间、集合及数据的写入和查询基本操作。
                                                                                            
                                                                                            
                                                                                            
