---
show: step
version: 1.0
enable_checker: true
---
# MySQL 实例的高可用配置



## 课程介绍

SequoiaSQL-MySQL 的架构使集群中的多个 MySQL 实例均为主机模式，均可对外提供读写服务。由于各实例的元数据均只存储在该实例本身，SequoiaSQL-MySQL 提供了元数据同步工具，用来保证 MySQL 服务的高可用。当一个 MySQL 实例退出后，连接该实例的应用可以切换到其它实例，获得对等的读写服务。  

本课程将带领您在2台机器中展示 MySQL 进行实例高可用的配置，其中1台已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境，另外一台已经安装了 MySQL 实例。

#### 请点击右侧选择使用的实验环境

#### MySQL 元数据同步工具架构

MySQL 元数据同步工具的基本原理是 MySQL 服务进程通过审计插件输出审计日志，元数据同步工具从审计日志中提取 SQL 语句，连接到其它 MySQL 实例执行，以达到元数据同步的目的。包含元数据同步工具的集群架构如下。

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/e938c31f0190facca69b64369fc1a5eb)

在上图中，meta_sync 即同步工具进程，每一个 MySQL 实例都有一个对应的同步工具在运行。它独立于 MySQL 服务进程运行，对 MySQL 的审计日志文件 server_audit.log 进行分析处理。由于用户的业务数据存储于底层的 SequoiaDB 数据库集群中，因此只要 MySQL 层的元数据在各实例间完成同步，连接 MySQL 实例的客户端就可以访问到一致的数据，这就为 MySQL 服务提供了高可用能力。


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
>用户 sdbadmin 的密码为 sdbadmin。

#### 查看巨杉数据库版本

查看 SequoiaDB 巨杉数据库引擎版本。

```shell
sequoiadb --version
```
操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/b4082b0d6d6bdf89d229aa713a53759d)

## 查看节点启动列表

查看 SequoiaDB 巨杉数据库引擎节点列表。

```shell
sdblist 
```

操作截图：

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/9737a96cfb2788c4361b2b6fb0abac88-0)

>Note:
>
>如果显示的节点数量与预期不符，请稍等节点初始化完成并重试该步骤。


## 创建元数据同步用户

进行元数据同步的 MySQL 实例均要设置用户名和密码，故需连接两个 MySQL 实例进行设置，本步骤展示对本机实例进行设置。

#### 创建数据库同步的用户

1）登录 MySQL Shell，连接本机的 MySQL 实例；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root
```

> 设置完成 127.0.0.1 的 MySQL 实例，需要对 172.17.0.2 的 MySQL 实例也进行设置。

2）创建用于同步元数据的 MySQL 用户；

```sql
CREATE USER 'sdbadmin'@'%' IDENTIFIED BY 'sdbadmin' ; 
```

并授予所有权限，用户名与密码在所有实例上保持一致；

```sql
GRANT all on *.* TO 'sdbadmin'@'%' with grant option ; 
FLUSH PRIVILEGES ;
```

> 注意：此处使用的密码 'sdbadmin' 仅为示例，请根据需要自行设置安全的密码。

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/bb2aeb2b052920db0eb468daf9dda54f)


#### 检查及验证同步的用户

1）查询数据库用户的权限；

```sql
SHOW GRANTS FOR sdbadmin ;
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/fcff6a32b56524b705e743e2e9a1ca0f)

2）退出 MySQL Shell， 使用 sdbadmin 用户重新登陆；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u sdbadmin -p
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/b667a6cc7f74c4b19d832efe32054996)



## 审计插件部署

进行元数据同步的 MySQL 实例均要部署审计插件，本步骤展示对本机实例进行设置。操作完成后需要 ssh 到 172.17.0.2 进行相同操作。

#### 审计插件准备

检查 MySQL 安装目录下 tools/lib 目录的审计插件；

```shell
ls /opt/sequoiasql/mysql/tools/lib/server_audit.so
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/20a621177c2cefbc611fde9555cfb9fd)

检查 MySQL 安装目录下 lib/plugin 目录的审计插件；

```shell
ls /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/601ca75e4c4bb64478cc9c27fd359619)

> 如截图显示，则代表未给 MySQL 配置数据库审计日志

将审计插件 server_audit.so 文件复制到 MySQL 安装目录中的 lib/plugin 目录下

```shell
cp /opt/sequoiasql/mysql/tools/lib/server_audit.so /opt/sequoiasql/mysql/lib/plugin/
```

赋予 MySQL 运行用户的可执行权限

```shell
chmod a+x /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

检查 MySQL 安装目录下 lib/plugin 目录的审计插件是否存在

```shell
ls -lat /opt/sequoiasql/mysql/lib/plugin/server_audit.so
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/43aad87b3132f462fc54d0182a0081af)


#### 审计插件配置

修改 MySQL 实例的配置文件（SequoiaSQL-MySQL 实例的配置文件为数据路径下的 auto.cnf），在 mysqld 部分添加以下内容，审计日志文件路径请根据实际情况进行配置，并手动完成创建，如以下示例中的 auditlog 目录。

```shell
vi /opt/sequoiasql/mysql/database/3306/auto.cnf 
```

配置文件末尾补充以下内容：
```
### add server_audit.so config ###
# 加载审计插件 
plugin-load=server_audit=server_audit.so 
# 审计记录的审计，建议只记录需要同步的DCL和DDL操作 server_audit_events=CONNECT,QUERY_DDL,QUERY_DCL 
# 开启审计 
server_audit_logging=ON 
# 审计日志路径及文件名 
server_audit_file_path=/opt/sequoiasql/mysql/database/auditlog/server_audit.log 
# 强制切分审计日志文件 
server_audit_file_rotate_now=OFF 
# 审计日志文件大小10MB，超过该大小进行切割，单位为byte 
server_audit_file_rotate_size=10485760 
# 审计日志保留个数，超过后会丢弃最旧的 
server_audit_file_rotations=999 
# 输出类型为文件 
server_audit_output_type=file 
# 限制每行查询日志的大小为100kb，若表比较复杂，对应的操作语句比较长，建议增大该值 
server_audit_query_log_limit=102400 
```

创建审计日志存放的文件夹;

```shell
mkdir /opt/sequoiasql/mysql/database/auditlog/
```

#### 重启 MySQL 实例并检查审计日志

检查 MySQL 实例;

```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/cc1e61a3314ee6c7b49740df0fdcfff9)



重启 MySQL 实例;

```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl restart myinst
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/6f99f32ac517288da00cfafac03bc76f)


检查 MySQL 实例进程;

```shell
/opt/sequoiasql/mysql/bin/sdb_sql_ctl listinst
```

操作截图:

![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/7639a5c0c083603c33c7fda2fb5861bd)


检查审计日志文件目录，确保生成了审计日志文件server_audit.log

```shell
ls -alt /opt/sequoiasql/mysql/database/auditlog/
```

操作截图:
![图片描述](https://doc.shiyanlou.com/courses/1540/1207281/dcc686392312e9273f2ac29cb65ce99f)



## 部署元数据同步工具

进行元数据同步的 MySQL 实例均要部署元数据同步工具，本步骤展示对本机实例进行设置。操作完成后需要 ssh 到 172.17.0.2 进行相同操作。

#### 元数据同步工具配置

拷贝生成一份元数据同步工具的配置文件;

```shell
cp /opt/sequoiasql/mysql/tools/metaSync/config.sample /opt/sequoiasql/mysql/tools/metaSync/config
```

进行元数据同步工具配置文件修改;
```shell
vi /opt/sequoiasql/mysql/tools/metaSync/config
```
配置内容:
```
[mysql]
# mysql节点主机名，只能填主机名
hosts = sdb1,sdb2,sdb3
# mysql服务端口
port = 3306
# 密码类型，0代表密码为明文，1代表密码为密文，初次使用配置为 0，密码使用明文，工具启动后会自动加密并更新此处配置
mysql_password_type = 0
# mysql用户
mysql_user = sdbadmin
# mysql密码
mysql_password = sdbadmin
# mysql安装目录
install_dir = /opt/sequoiasql/mysql
# 审计日志存储目录
audit_log_directory = /opt/sequoiasql/mysql/database/auditlog
# 审计日志文件名
audit_log_file_name = server_audit.log

[execute]
# 同步间隔，取值范围：[1-3600]
interval_time = 5
# 出错时是否忽略，如为 false，会一直重试
ignore_error = true
# 出错的情况下，忽略前的重试次数，取值范围：[1-1000]
max_retry_times = 5
```

拷贝一份元数据同步的日志配置文件同步工具使用 python 的 logging 模块输出日志，配置文件为 log.config。如果是全新安装，开始该文件是不存在的，需要从 log.config.sample 拷贝。配置项如下（日志目录会自动创建）：
```shell
cp /opt/sequoiasql/mysql/tools/metaSync/log.config.sample /opt/sequoiasql/mysql/tools/metaSync/log.config
```
进行元数据同步工具日志配置文件修改
```shell
vi /opt/sequoiasql/mysql/tools/metaSync/log.config
```

配置内容:
```
[loggers]
keys=root,ddlLogger
[handlers]
keys=rotatingFileHandler
[formatters]
keys=loggerFormatter
[logger_root]
level=INFO
handlers=rotatingFileHandler
[logger_ddlLogger]
level=INFO
handlers=rotatingFileHandler
qualname=ddlLogger
propagate=0

[handler_rotatingFileHandler]
class=logging.handlers.RotatingFileHandler
# 日志级别，支持ERROR,INFO,DEBUG
level=INFO
# 日志格式
formatter=loggerFormatter
# 第1个参数为运行日志文件名,路径对应的目录必须已存在
# 第2个参数为写入模式，值为'a+',不建议修改
# 第3个参数为日志文件大小，单位为byte
# 第4个参数为备份日志文件，即日志文件总数为10+1
args=('logs/run.log', 'a+', 104857600, 10)

[formatter_loggerFormatter]
format=%(asctime)s [%(levelname)s] [%(filename)s:%(lineno)s] %(message)s
datefmt=
```

#### 启动元数据同步工具

在完成所有配置后，在各实例所在主机的 sdbadmin 用户下，执行以下命令在后台启动同步工具：

```shell
python /opt/sequoiasql/mysql/tools/metaSync/meta_sync.py &
```

可以通过配置定时任务提供基本的同步工具监控，定期检查程序是否在运行，若进程退出了，会被自动拉起。配置命令如下（在 SequoiaSQL-MySQL 安装用户下配置）：
```shell
crontab -e
```
crontab 编辑的内容如下
```shell
#每一分钟运行一次
*/1 * * * * /usr/bin/python /opt/sequoiasql/mysql/tools/metaSync/meta_sync.py >/dev/null 2>&1 &
```

#### 验证元数据同步工具
linux 系统后台进程作业配置检查

```shell
crontab -l
```
## 验证元数据同步情况

进入 MySQL shell ，连接 SequoiaSQL-MySQL 实例并创建 company 数据库实例，并创建数据表。

#### 登录 MySQL shell 

```shell
/opt/sequoiasql/mysql/bin/mysql -h 127.0.0.1 -P 3306 -u root -p
```

#### 创建数据库实例

```sql
CREATE DATABASE company ;
USE company ;
```

#### 创建数据表，并写入数据

1）创建包含自增主键字段的 employee 表；

```sql
CREATE TABLE employee (empno INT AUTO_INCREMENT PRIMARY KEY, ename VARCHAR(128), age INT) ;
```

2）写入数据；

```sql
INSERT INTO employee (ename, age) VALUES ("Jacky", 36) ;
INSERT INTO employee (ename, age) VALUES ("Alice", 18) ;
```

3）查询数据；

```sql
SELECT * FROM employee ;
```

4）退出 MySQL Shell；
```
\q
```

#### 验证172.17.0.2机器的 MySQL 实例

1）登录 MySQL Shell，连接172.17.0.2的 MySQL 实例；

```shell
/opt/sequoiasql/mysql/bin/mysql -h 172.17.0.2 -P 3306 -u root -p
```
2） 切换数据库；

```sql
USE company ;
```

3）查询 employee 表；

```sql
SELECT * FROM employee ;
```

## 总结
本课程我们学会了 MySQL 实例的高可用配置并进行了验证，包括创建数据库用户、审计插件部署和元数据同步工具部署。
