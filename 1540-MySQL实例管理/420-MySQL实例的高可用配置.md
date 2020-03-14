---
show: step
version: 23.0
enable_checker: true
---

# MySQL 实例的高可用配置

## 课程介绍

SequoiaSQL-MySQL 的架构使集群中的多个 MySQL 实例均为主机模式，都可独立对外提供读写服务。由于各实例的元数据都只存储在该实例本身，于是 SequoiaSQL-MySQL 提供了元数据同步工具，用来保证 MySQL 服务的高可用。当一个 MySQL 实例退出后，连接该实例的应用可以切换到其它实例，获得对等的读写服务。  

本课程将带领您在2台机器中展示 MySQL 进行实例高可用的配置，其中1台已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境，另外一台只安装了 MySQL 实例组件。

#### 请点击右侧选择使用的实验环境


#### MySQL 元数据同步工具架构
MySQL 元数据同步工具的基本原理是 MySQL 服务进程通过审计插件输出审计日志，元数据同步工具从审计日志中提取 SQL 语句，连接到其它 MySQL 实例执行，以达到元数据同步的目的。包含元数据同步工具的集群架构如下:


![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/e938c31f0190facca69b64369fc1a5eb)

在上图中，meta_sync 即同步工具进程，每一个 MySQL 实例都有一个对应的同步工具在运行。它独立于 MySQL 服务进程运行，对 MySQL 的审计日志文件 server_audit.log 进行分析处理。由于用户的业务数据存储于底层的 SequoiaDB 数据库集群中，因此只要 MySQL 层的元数据在各实例间完成同步，连接 MySQL 实例的客户端就可以访问到一致的数据，这就为 MySQL 服务提供了高可用能力。

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 巨杉数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。


## 切换用户及查看数据库版本

切换至部署 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 实例的操作系统用户并查看数据库引擎版本。

#### 切换到 sdbadmin 用户

```shell
su - sdbadmin
```
>Note:
>
>用户 sdbadmin 的密码为 `sdbadmin`

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本：

```shell
sequoiadb --version
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1538/1207281/6cccf5951f048e01b4789f3c08483bb0-0)

## 查看服务状态

检查课程涉及的 SequoiaDB 巨杉数据库和 SequoiaSQL-MySQL 程序是否运行正常。

#### 检查 SequoiaDB 巨杉数据库节点列表

查看 SequoiaDB 巨杉数据库引擎节点列表：

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/8d30e12b58b3ac0f0f9dd762b244fa84-0)

>Note:
>
>如果显示的节点数量与预期不符，请稍等节点初始化完成并重试该步骤。

#### 查看本机 MySQL 实例

1）切换到 SequoiaSQL-MySQL 安装目录；

```shell
cd /opt/sequoiasql/mysql
```

2）查看实例名；
```shell
bin/sdb_sql_ctl listinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/f1b3322d7bca2ff70e418faa293e9e8b-0)

3）查看对应实例是否启动；

```shell
bin/sdb_sql_ctl status
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/56e7eee4a5915f2b90e272376f69a987-0)

>Note:
>
>显示 PID 值不为空说明实例已经运行中。

#### 查看远程机器 MySQL 实例

1）登录 sdbserver2 远程服务器；

```shell
ssh sdbadmin@sdbserver2
```

>Note:
>
>用户 sdbadmin 的密码为 `sdbadmin`


2）查看实例名；

```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/a83e6dd386de1f04b959e6004b19800f-0)

>Note:
>
>命令未发现存在的实例名，说明仅安装 SequoiaSQL-MySQL 实例组件。

3）退出 sdbserver2 远程服务器；

```shell
exit
```


## 创建元数据同步用户

进行元数据同步的 MySQL 实例均要设置相同的用户名和密码，故需连接两个 MySQL 实例进行相应的设置，本步骤展示对本机的 MySQL 实例进行设置。

#### 创建数据库同步的用户

1）登录 MySQL Shell，连接本机的 MySQL 实例；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

2）创建用于同步元数据的 MySQL 用户；

```sql
CREATE USER 'sdbadmin'@'%' IDENTIFIED BY 'sdbadmin'; 
```

3）授予所有权限，用户名与密码在所有实例上保持一致；

```sql
GRANT ALL ON *.* TO 'sdbadmin'@'%' WITH GRANT OPTION; 
FLUSH PRIVILEGES;
```

>Note:
>
> 此处使用的密码 `sdbadmin` 仅为示例，请根据需要自行的需要设置安全的密码。

4）查询数据库用户的权限；

```sql
SHOW GRANTS FOR sdbadmin;
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/fcff6a32b56524b705e743e2e9a1ca0f)

5）退出 MySQL Shell ；

```sql
\q
```

6）使用 `sdbadmin` 用户重新登陆 MySQL ，如果进入 MySQL Shell 说明密码设置成功；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -usdbadmin -psdbadmin
```

7）退出 MySQL Shell ；

```sql
\q
```

## 审计插件部署

进行元数据同步的 MySQL 实例均要部署审计插件，本步骤展示对本机实例进行设置。

#### 审计插件准备

1）检查 MySQL 安装目录下 tools/lib 目录的审计插件；

```shell
ls /opt/sequoiasql/mysql/tools/lib/server_audit.so
```


2）检查 MySQL 安装目录下 lib/plugin 目录的审计插件；

```shell
ls /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/135182bca08cfd54eea443ac5813bc34-0)
> Note:
>
> 文件不存在，需要给 SequoiaSQL-MySQL 配置数据库审计日志。

3）将审计插件 server_audit.so 文件复制到 MySQL 安装目录中的 lib/plugin 目录下；
```shell
cp /opt/sequoiasql/mysql/tools/lib/server_audit.so /opt/sequoiasql/mysql/lib/plugin/
```

4）赋予 MySQL 运行用户的可执行权限；
```shell
chmod a+x /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

5）检查 MySQL 安装目录下 lib/plugin 目录的审计插件是否存在；
```shell
ls -lat /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/00024367ea70eb1775ecafee415a41f2-0)


#### 审计插件配置

1）修改 MySQL 实例的配置文件；

```shell
echo 'plugin-load=server_audit=server_audit.so' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_logging=ON' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_file_path=/opt/sequoiasql/mysql/database/auditlog/server_audit.log' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_file_rotate_now=OFF' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_file_rotate_size=10485760' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_file_rotations=999' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_output_type=file' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
echo 'server_audit_query_log_limit=102400' >> /opt/sequoiasql/mysql/database/3306/auto.cnf 
```

>Note:
>- add server_audit.so config ：加载审计插件；
>- plugin-load=server_audit=server_audit.so ：审计记录的审计，建议只记录需要同步的DCL和DDL操作 server_audit_events=CONNECT，QUERY_DDL,QUERY_DCL ；
>- server_audit_logging=ON ：开启审计；
>- server_audit_file_path=/opt/sequoiasql/mysql/database/auditlog/server_audit.log ：审计日志路径及文件名 ；
>- server_audit_file_rotate_now=OFF ：强制切分审计日志文件 ；
>- server_audit_file_rotate_size=10485760 ：审计日志文件大小10MB，超过该大小进行切割，单位为byte ；
>- server_audit_file_rotations=999 ：审计日志保留个数，超过后会丢弃最旧的 ；
>- server_audit_output_type=file ： 输出类型为文件 ；
>- server_audit_query_log_limit=102400 ：限制每行查询日志的大小为100kb，若表比较复杂，对应的操作语句比较长，建议增大该值 ；



2）创建审计日志存放的文件夹；
```shell
mkdir /opt/sequoiasql/mysql/database/auditlog/
```

#### 重启 MySQL 实例并检查审计日志

1）检查 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/5af8080b632bae7b1387d858429c93f0-0)



2）重启 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl restart myinst
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/6f99f32ac517288da00cfafac03bc76f)


3）检查 MySQL 实例进程；
```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1542/1207281/5af8080b632bae7b1387d858429c93f0-0)


4）检查审计日志文件目录，确保生成了审计日志文件 server_audit.log；
```shell
ls -alt /opt/sequoiasql/mysql/database/auditlog/
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/dcc686392312e9273f2ac29cb65ce99f)


## 部署元数据同步工具

进行元数据同步的 MySQL 实例均要部署元数据同步工具，本步骤展示对本机实例进行设置。

#### 元数据同步工具配置

1）拷贝生成一份元数据同步工具的配置文件；
```shell
cp /opt/sequoiasql/mysql/tools/metaSync/config.sample /opt/sequoiasql/mysql/tools/metaSync/config
```

2）进行元数据同步工具配置文件修改；
```shell
sed -i 's/hosts = sdb1,sdb2,sdb3/hosts = sdbserver1,sdbserver2/g' /opt/sequoiasql/mysql/tools/metaSync/config
```

3）确认配置文件 hosts 参数是否为 sdbserver1,sdbserver2 ；

```shell
cat /opt/sequoiasql/mysql/tools/metaSync/config
```

4）拷贝一份元数据同步的日志配置文件同步工具使用 python 的 logging 模块输出日志，配置文件为 log.config。如果是全新安装，开始该文件是不存在的，需要从 log.config.sample 拷贝。配置项如下（日志目录会自动创建）；

```shell
cp /opt/sequoiasql/mysql/tools/metaSync/log.config.sample /opt/sequoiasql/mysql/tools/metaSync/log.config
```




#### 启动元数据同步工具

1）在完成所有配置后，在各实例所在主机的 `sdbadmin` 用户下，执行以下命令在后台启动同步工具；

```shell
python /opt/sequoiasql/mysql/tools/metaSync/meta_sync.py &
```

2）可以通过配置定时任务提供基本的同步工具监控，定期检查程序是否在运行，若进程退出了，会被自动拉起。配置命令如下（在 SequoiaSQL-MySQL 安装用户下配置）；

```shell
crontab -e
```

3）去到最后一行按 `i` 然后添加以下内容；
```
#每一分钟运行一次
*/1 * * * * /usr/bin/python /opt/sequoiasql/mysql/tools/metaSync/meta_sync.py >/dev/null 2>&1 &
```

4）添加后按 Esc 键输入 :wq 进行保存退出;

5）linux 系统后台进程作业配置检查，最后一行可以显示上一步骤输入内容；

```shell
crontab -l

```


## 安装配置 sdbserver2 元数据同步工具

以上步骤已经完成 sdbserver1 的 MySQL 元数据同步工具部署和配置，本章将对 sdbserver2 进行设置。 

#### 登录 sdbserver2 创建实例

1）登录 sdbserver2 远程服务器；

```shell
ssh sdbadmin@sdbserver2
```

>Note:
>
>用户 sdbadmin 的密码为 `sdbadmin`


2）进入 SequoiaSQL-MySQL 实例安装目录；

```shell
cd /opt/sequoiasql/mysql
```

3）创建数据库实例；

```shell
bin/sdb_sql_ctl addinst myinst -D database/3306/
```

4）查看实例；

```shell
bin/sdb_sql_ctl status
```

5）设置本实例连接巨杉数据库的协调节点；

```shell
sed -i 's/# sequoiadb_conn_addr=localhost:11810/sequoiadb_conn_addr=172.17.0.1:11810/g' database/3306/auto.cnf
```

6）重启 MySQL 实例；

```shell
bin/sdb_sql_ctl restart myinst
```


7）查看 MySQL 实例状态；

```shell
bin/sdb_sql_ctl status
```


操作截图：


![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/56e7eee4a5915f2b90e272376f69a987-0)



#### 安装配置 sdbserver2 元数据同步工具
需要在 sdbserver2 重复以下步骤 ，点击左下角章节图标，根据弹出内容选择对应章节。

- 创建元数据同步用户

- 审计插件部署

- 部署元数据同步工具


## 验证元数据同步情况

进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库，并创建数据表。

#### 登录 MySQL shell 

登录 MySQL 实例；
```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

#### 创建数据库

创建数据库；
```sql
CREATE DATABASE company;
USE company;
```

#### 创建数据表，并写入数据

1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee 
(
empno INT AUTO_INCREMENT PRIMARY KEY, 
ename VARCHAR(128), 
age INT
);
```

2）写入数据；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36);
INSERT INTO employee (ename, age) VALUES ("Alice", 18);
```

3）查询数据；

```sql
SELECT * FROM employee;
```

4）退出 MySQL Shell；
```sql
\q
```

5）退出 sdbserver2 机器；
```shell
exit
```

#### 验证 sdbserver1 的 MySQL 实例元数据同步情况

1）登录 MySQL Shell ；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -usdbadmin -psdbadmin
```

2）查看数据库，可以看到 company 数据库已经同步创建；

```sql
SHOW DATABASES;
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/c32c36f921b0e11c6d271e713248ce80-0
)


3） 切换数据库；

```sql
USE company;
```

4）查询 employee 表；

```sql
SELECT * FROM employee;
```
5）退出 MySQL Shell；
```sql
\q
```

## 总结
本课程我们学会了 MySQL 实例的高可用配置并进行了验证，包括创建数据库用户、审计插件部署和元数据同步工具部署。
