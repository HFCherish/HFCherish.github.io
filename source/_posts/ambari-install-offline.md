---
title: HDP install (offline using ambari)
toc: true
tags:
  - big data
date: 2020-11-23 15:36:44
---

[reference](https://wbaseurlww.cnblogs.com/shook/p/12409759.html)

[官方安装指导](https://docs.cloudera.com/HDPDocuments/Ambari-latest/bk_ambari-installation/content/set_up_password-less_ssh.html)

# Preparation

除非说明，默认以下操作都是在所有节点上执行

## 修改 host

```sh
[root@master ~]# vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.105.137 master
192.168.105.191 slave1
192.168.105.13 slave2
```

## 修改 network config

```sh
[root@master ~]# vi /etc/sysconfig/network
# Created by anaconda
NETWORKING=yes
HOSTNAME=master

[root@master ~]# hostnamectl set-hostname master
[root@master ~]# hostname
master

# ping 各个节点，查看是否可连通
[root@master ~]# ping slave1
PING slave1 (192.168.105.191) 56(84) bytes of data.
```

## 同步时间 ntp

## 关闭防火墙

## 关闭 Selinux 和 THP

## ~~修改文件打开最大限制~~

## SSH 无密码登录（主节点）

## Reboot

```sh
$ shutdown -r now
```

# 制作本地源（离线安装）

## 文件目录访问（http 服务方式）

## 制作本地源（主节点）

### 安装本地源制作相关工具

### 修改文件里面的源地址

```sh
[root@master ambari]# vi ambari/centos7/2.7.4.0-118/ambari.repo
#VERSION_NUMBER=2.7.4.0-118
[ambari-2.7.4.0]
#json.url = http://public-repo-1.hortonworks.com/HDP/hdp_urlinfo.json
name=ambari Version - ambari-2.7.4.0
baseurl=http://192.168.105.137/ambari/ambari/centos7/2.7.4.0-118
gpgcheck=1
gpgkey=http://192.168.105.137/ambari/ambari/centos7/2.7.4.0-118/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
[root@master ambari]# cp ambari/centos7/2.7.4.0-118/ambari.repo /etc/yum.repos.d/
[root@master ambari]# vi HDP/centos7/3.1.4.0-315/hdp.repo
#VERSION_NUMBER=3.1.4.0-315
[HDP-3.1.4.0]
name=HDP Version - HDP-3.1.4.0
baseurl=http://192.168.105.137/ambari/HDP/centos7/3.1.4.0-315
gpgcheck=1
gpgkey=http://192.168.105.137/ambari/HDP/centos7/3.1.4.0-315/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://192.168.105.137/ambari/HDP-UTILS/centos7/1.1.0.22
gpgcheck=1
gpgkey=http://192.168.105.137/ambari/HDP-UTILS/centowwws7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
[root@master ambari]# cp HDP/centos7/3.1.4.0-315/hdp.repo /etc/yum.repos.d/
```



### 将创建好的源文件（.repo）拷贝到子节点

# 安装 ambari-server

## 安装 ambari-server

先安装，然后开始配置

```sh
$ yum -y install ambari-server
```

## 设置并启动 ambari-server（主节点）

[how to find JAVA_HOME](https://www.baeldung.com/find-java-home)

```sh
$ java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home' 
```

### 使用默认 postgresql

Setup server 时，有几个交互式配置：

1. 是否自定义用户账户：

   1. 选 n。即默认设置了 Ambari GUI 的登录用户为 admin/admin。并且指定 Ambari Server 的运行用户为 root。

   > If you want to create a different user to run the Ambari Server, or to assign a previously created user, select **`y`** at the `Customize user account for ambari-server daemon` prompt, then provide a user name.

2. JDK：

   1. 选 2. 因为默认会安装并使用 oracle jdk，但是（1）不能联网下载（2）好像 oracle jdk 不会下载依赖包。所以自己安装好，在这指定 path 即可

3. 数据库：

   1. 按默认配置创建 postgres 数据库


```sh
[root@master yum.repos.d]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
Enter choice (1): 2
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/java/jdk1.8.0_202
Validating JDK on Ambari Server...done.
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? n
Configuring database...
Default properties detected. Using built-in database.
Configuring ambari database...
Checking PostgreSQL...
Running initdb: This may take up to a minute.
Initializing database ... OK


About to start PostgreSQL
Configuring local database...
Configuring PostgreSQL...
Restarting PostgreSQL
Creating schema and user...
done.
Creating tables...
done.
Extracting system views...
ambari-admin-2.7.4.0-118.jar
...........
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```

### 使用 mysql

[离线安装 mysql](https://programmer.group/linux-centos-7-mysql-5.7-offline-installation.html)

#### 离线安装

```sh
# 创建用户组
$ groupadd hdp
# 创建用户
$ mkdir /home/mysql
$ useradd -g hdp hive -d /home/mysql/hive
$ passwd hive
yourPassword
# 创建临时目录
$ mkdir /home/mysql/hive/3306/data
$ mkdir /home/mysql/hive/3306/log
$ mkdir /home/mysql/hive/3306/tmp
$ chown -R hive:hdp /home/mysql/hive

# 解压
$ mv mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz /usr/local
$ cd /usr/local
$ tar -xzvf mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz
# Establish soft links for future upgrades
$ ln -s mysql-5.7.27-linux-glibc2.12-x86_64 mysql
# Modify users and user groups of all files under mysql folder
$ chown -R mysql:mysql mysql/

# 创建配置文件
$ cd /etc
$ vi my.cnf

# 安装 mysql
$ cd /usr/local/mysql/bin
$ ./mysqld --initialize --user=hive
```

#### 安装 driver, 并配置与 ambari-server 的 jdbc 连接

[mysql-connector-driver 下载](https://dev.mysql.com/downloads/connector/j/) （选 RedHat 8 那个操作系统）

```sh
# 解压
$ rpm -ivh mysql80-community-release-el7-3.noarch.rpm
$ cp /usr/share/java/mysql-connector-java.jar /var/lib/ambari-server/resources/mysql-jdbc-driver.jar
# 配置 driver path
$ vi /etc/ambari-server/conf/ambari.properties
添加server.jdbc.driver.path=/usr/share/java/mysql-connector-java.jar
```

#### 配置 mysql

```sh
# 设置开机启动，并启动 mysql
$ cd /usr/local/mysql
# Copy the startup script to the resource directory and modify mysql.server. It's better to modify mysqld as well. These two files are best synchronized.
$ cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
# Increase the execution privileges of mysqld service control scripts
$ chmod +x /etc/rc.d/init.d/mysqld
# Add mysqld service to system service
$ chkconfig --add mysqld
# Check whether the mysqld service is in effect
$ chkconfig --list mysqld 
# mysql start
$ service mysqld start
# View mysql status
$ service mysqld status
# Check mysql related processes
$ ps aux|grep mysql

# 配置环境变量
$ vi /etc/profile
export PATH=$PATH:/usr/local/mysql/bin
$ source /etc/profile

# 更新 root 密码
$ mysql -uroot -p
$ mysql> set password for root@localhost=password("root");
# 配置 remote access to the mysql
$ mysql> use mysql
$ mysql> update user set host='%' where user='root';
$ mysql> select host,user from user;
$ mysql> grant all privileges on *.* to 'root'@'%' identified by 'yourPassword';
$ mysql> flush privileges;
```

#### 创建 database

```mysql
CREATE DATABASE ambari;  
use ambari;  
CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari';  
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';  
CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'ambar';  
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';  
CREATE USER 'ambari'@'master' IDENTIFIED BY 'ambari';  
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'master';  
FLUSH PRIVILEGES;  
source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql  

CREATE DATABASE hive;  
use hive;  
CREATE USER 'hive'@'%' IDENTIFIED BY 'hive';  
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%';  
CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive';  
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'localhost';  
CREATE USER 'hive'@'master' IDENTIFIED BY 'hive';  
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'master';  
FLUSH PRIVILEGES;  
CREATE DATABASE oozie;  
use oozie;  
CREATE USER 'oozie'@'%' IDENTIFIED BY 'oozie';  
GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'%';  
CREATE USER 'oozie'@'localhost' IDENTIFIED BY 'oozie';  
GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'localhost';  
CREATE USER 'oozie'@'master' IDENTIFIED BY 'oozie';  
GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'master';  
FLUSH PRIVILEGES;
```



#### Mysql conf

上边 cnf 的内容

```shell
[client]                                        # Client settings, the default connection parameters for the client
port = 3306                                    # Default connection port
socket = /home/mysql/hive/3306/tmp/mysql.sock                        # For socket sockets for local connections, the mysqld daemon generates this file

[mysqld]                                        # Server Basic Settings
# Foundation setup
user = hive
bind-address = 0.0.0.0                         # Allow any ip host to access this database
server-id = 1                                  # The unique number of Mysql service Each MySQL service Id needs to be unique
port = 3306                                    # MySQL listening port
basedir = /usr/local/mysql                      # MySQL installation root directory
datadir = /home/mysql/hive/3306/data                      # MySQL Data File Location
tmpdir  = /home/mysql/hive/3306/tmp                                  # Temporary directories, such as load data infile, will be used
socket = /home/mysql/hive/3306/tmp/mysql.sock        # Specify a socket file for local communication between MySQL client program and server
pid-file = /home/mysql/hive/3306/log/mysql.pid      # The directory where the pid file is located
skip_name_resolve = 1                          # Only use IP address to check the client's login, not the host name.
character-set-server = utf8mb4                  # Database default character set, mainstream character set support some special emoticons (special emoticons occupy 4 bytes)
transaction_isolation = READ-COMMITTED          # Transaction isolation level, which is repeatable by default. MySQL is repeatable by default.
collation-server = utf8mb4_general_ci          # The character set of database corresponds to some sort rules, etc. Be careful to correspond to character-set-server.
init_connect='SET NAMES utf8mb4'                # Set up the character set when client connects mysql to prevent scrambling
lower_case_table_names = 1                      # Is it case sensitive to sql statements, 1 means insensitive
max_connections = 400                          # maximum connection
max_connect_errors = 1000                      # Maximum number of false connections
explicit_defaults_for_timestamp = true          # TIMESTAMP allows NULL values if no declaration NOT NULL is displayed
max_allowed_packet = 128M                      # The size of the SQL packet sent, if there is a BLOB object suggested to be modified to 1G
interactive_timeout = 1800                      # MySQL connection will be forcibly closed after it has been idle for more than a certain period of time (in seconds)
wait_timeout = 1800                            # The default value of MySQL wait_timeout is 8 hours. The interactive_timeout parameter needs to be configured concurrently to take effect.
tmp_table_size = 16M                            # The maximum value of interior memory temporary table is set to 128M; for example, group by, order by with large amount of data may be used as temporary table; if this value is exceeded, it will be written to disk, and the IO pressure of the system will increase.
max_heap_table_size = 128M                      # Defines the size of memory tables that users can create
query_cache_size = 0                            # Disable mysql's cached query result set function; later test to determine whether to turn on or not based on business conditions; in most cases, close the following two items
query_cache_type = 0

# Memory settings allocated by user processes, and each session will allocate memory size for parameter settings
read_buffer_size = 2M                          # MySQL read buffer size. Requests for sequential table scans allocate a read buffer for which MySQL allocates a memory buffer.
read_rnd_buffer_size = 8M                      # Random Read Buffer Size of MySQL
sort_buffer_size = 8M                          # Buffer size used for MySQL execution sort
binlog_cache_size = 1M                          # A transaction produces a log that is recorded in Cache when it is not committed, and persists the log to disk when it needs to be committed. Default binlog_cache_size 32K

back_log = 130                                  # How many requests can be stored on the stack in a short time before MySQL temporarily stops responding to new requests; the official recommendation is back_log = 50 + (max_connections/5), with a cap of 900

# log setting
log_error = /home/mysql/hive/3306/log/error.log                          # Database Error Log File
slow_query_log = 1                              # Slow Query sql Log Settings
long_query_time = 1                            # Slow query time; Slow query over 1 second
slow_query_log_file = /home/mysql/hive/3306/log/slow.log                  # Slow Query Log Files
log_queries_not_using_indexes = 1              # Check sql that is not used in the index
log_throttle_queries_not_using_indexes = 5      # Represents the number of SQL statements per minute that are allowed to be logged to a slow log and are not indexed. The default value is 0, indicating that there is no limit.
min_examined_row_limit = 100                    # The number of rows retrieved must reach this value before they can be recorded as slow queries. SQL that returns fewer than the rows specified by this parameter is not recorded in the slow query log.
expire_logs_days = 5                            # MySQL binlog log log file saved expiration time, automatically deleted after expiration

# Master-slave replication settings
log-bin = mysql-bin                            # Open mysql binlog function
binlog_format = ROW                            # The way a binlog records content, recording each row being manipulated
binlog_row_image = minimal                      # For binlog_format = ROW mode, reduce the content of the log and record only the affected columns

# Innodb settings
innodb_open_files = 500                        # Restrict the data of tables Innodb can open. If there are too many tables in the library, add this. This value defaults to 300
innodb_buffer_pool_size = 64M                  # InnoDB uses a buffer pool to store indexes and raw data, usually 60% to 70% of physical storage; the larger the settings here, the less disk I/O you need to access the data in the table.
innodb_log_buffer_size = 2M                    # This parameter determines the size of memory used to write log files in M. Buffers are larger to improve performance, but unexpected failures can result in data loss. MySQL developers recommend settings between 1 and 8M
innodb_flush_method = O_DIRECT                  # O_DIRECT reduces the conflict between the cache of the operating system level VFS and the buffer cache of Innodb itself.
innodb_write_io_threads = 4                    # CPU multi-core processing capability settings are adjusted according to read-write ratio
innodb_read_io_threads = 4
innodb_lock_wait_timeout = 120                  # InnoDB transactions can wait for a locked timeout second before being rolled back. InnoDB automatically detects transaction deadlocks and rolls back transactions in its own lock table. InnoDB notices the lock settings with the LOCK TABLES statement. The default value is 50 seconds.
innodb_log_file_size = 32M                      # This parameter determines the size of the data log file. Larger settings can improve performance, but also increase the time required to recover the failed database.
```



## 停止 ambari-server

```sh
[root@master ~] ambari-server stop    #停止命令

[root@master ~]# # ambari-server reset   #重置命令
[root@master ~]# # ambari-server setup   #重新设置 
[root@master ~]# # ambari-server start   #启动命令
```

# 配置集群

## 基本信息

集群名字：ftms_hdp_qat

选择版本：

* hdp 3.1
* redhat7

## 配置服务

选择服务

选择 master/slave for 各服务

![image-20201125154733621](/images/ambari-20201125154733621.png)

### 配置 hive/ozzie/ranger database

### 使用 postgresql

```sh

# 先执行下边这句话，再继续配置
$ ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```

### 使用 mysql

使用 mysql（生产环境推荐使用），且 mysql 在其他地方也在用，而 postgresql 和 mysql 的语法是有区别的。

```sh
# 先执行下边这句话，再继续配置
$ ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```

### 配置 directories

采用默认的


### configurations

采用默认的

# 异常调试

## 查看 ambari 的配置

```sh
# 查看 ambari-server 的配置
$ vi /etc/ambari-server/conf/ambari.properties

# 查看 hdp 各服务的配置
$ cd /usr/hdp/3.1.4.0-315/hbase/conf
# 运行服务
$ /usr/hdp/3.1.4.0-315/hbase/bin/hbase shell
```

## 查看错误日志

```sh
# 查看 ambari-server 启动的错误日志
$ tail /var/log/ambari-server/ambari-server.log -n 10 -f | less

# 查看 hdp 各服务的日志
$ cd /usr/hdp/3.1.4.0-315/hbase/logs/hbase/hbase-hbase-regionserver-slave2.log
# 或者在 var 下看也一样，不知道是放了软 link 还是什么，日志好像是一样的
$ cd /var/log/hbase/hbase-hbase-regionserver-slave2.log

# 查看 ranger service 的配置
$ /etc/ranger/
```

## 重新安装

```sh
$ ambari-server stop
$ ambari-server reset
$ ambari-server setup
```

## 安装找不到包

### 找不到 hdp.repo

在实际安装时，ambari 会生成一个新的 ambari-hdp-1.repo，其中也定义了 hdp 的 baseurl 之类，这里对 hdp 定义的 name 可能是 `[HDP-3.1-repo-1]` , 而前边准备本地库时，定义的 hdp 的 name 是 `[HDP-3.1.4.0]`，这两个名字必须保持一致，否则 ambari 就找不到包（这是 ambari 的一个 bug）。

> 解决方案：
>
> 将 `hdp.repo` 中的 [HDP-3.1.4.0] 改为 [HDP-3.1-repo-1]，并重新 scp 到各个节点

### postfix找不到libmysqlclient.so.18

>还有一种简单的方法（没试过）：
>
>```sh
>$ yum reinstall mysql-libs -y
>```

```sh
# 先删除现在安装的 postfix
$ systemctl disable postfix
$ systemctl stop postfix
$ yum remove postfix

# 给 libmysqlclient.so.18 加上软链
$ find / -name '*libmysqlclient.so.18*'
/usr/lib64/mysql/libmysqlclient.so.18
$ ln -s /usr/lib64/mysql/libmysqlclient.so.18 /usr/lib/libmysqlclient.so.18

# 重新安装 postfix 并启动
$ yum install postfix
$ systemctl enable postfix
$ systemctl start postfix
$ systemctl status postfix.service
```

### 找不到 libtirpc-devel

如果出错，可能各个节点都需要做这件事

```sh
# 不能联网
# 先下载 https://centos.pkgs.org/7/centos-x86_64/libtirpc-devel-0.2.4-0.16.el7.x86_64.rpm.html 包
$ yum-config-manager --enable rhui-REGION-rhel-server-optional
$ yum install libtirpc-devel-0.2.4-0.16.el7.x86_64.rpm

# 如果可以联网。这个可以从 CentOs-Base.repo 里下，但不能联网就没办法了
$ cd /etc/yum.repos.d
$ cp backup/CentOs-Base.repo .
```

## ranger-admin start fail

start ranger-admin fail.

> error detail:
> This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)

Solution ([stackflow](https://stackoverflow.com/questions/26015160/deterministic-no-sql-or-reads-sql-data-in-its-declaration-and-binary-logging-i)):

```sh
# Execute the following in the MySQL console:
SET GLOBAL log_bin_trust_function_creators = 1;
```

## atlas server 启动失败

### Ranger authorization 失败

> 执行 `cat /var/lib/ambari-agent/tmp/atlas_hbase_setup.rb | hbase shell -n` 命令时，失败报 404
>
> Error detail:
>
> ```sh
> ERROR Java::OrgApacheHadoopHbaseIpc::RemoteWithExtrasException: org.apache.hadoop.hbase.coprocessor.CoprocessorException: HTTP 404 Error: HTTP 404
> 	at org.apache.ranger.authorization.hbase.RangerAuthorizationCoprocessor.grant(RangerAuthorizationCoprocessor.java:1261)
> 	at org.apache.ranger.authorization.hbase.RangerAuthorizationCoprocessor.grant(RangerAuthorizationCoprocessor.java:1072)
> 	at org.apache.hadoop.hbase.protobuf.generated.AccessControlProtos$AccessControlService$1.grant(AccessControlProtos.java:10023)
> 	at org.apache.hadoop.hbase.protobuf.generated.AccessControlProtos$AccessControlService.callMethod(AccessControlProtos.java:10187)
> 	at org.apache.hadoop.hbase.regionserver.HRegion.execService(HRegion.java:8135)
> 	at org.apache.hadoop.hbase.regionserver.RSRpcServices.execServiceOnRegion(RSRpcServices.java:2426)
> 	at org.apache.hadoop.hbase.regionserver.RSRpcServices.execService(RSRpcServices.java:2408)
> 	at org.apache.hadoop.hbase.shaded.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:42010)
> 	at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:413)
> 	at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:131)
> 	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:324)
> 	at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:304)
> 
> ```

这个原因是 atlas 通过 ranger 访问 hbase 的时候没有权限。可能是由于安装先后顺序的原因，也有 atlas + ranger 本身需要手动配置的原因，导致 ranger 中没有配置 atlas 对 hbase、kafka 的访问权限。因此需要做做几件事：

1. 在 ambari 的 ranger config 中，启动 hbase ranger plugin，并重启相关服务

2. 在 ambari 的 ranger config 中，启动 kafka ranger  plugin，并重启相关服务 ===> 暂时没做

3. 在 ranger 添加 hbase 的 service（参照 [Configure a Resource-based Service: HBase](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/authorization-ranger/content/resource_service_configure_an_hbase_service.html))
   1. 这里 service 的名称必须和 `/usr/hdp/3.1.4.0-315/ranger-hbase-plugin/install.properties` 里配置 `REPOSITORY_NAME` 保持一致（但是似乎并不需要？）
   2. username，password 说是 The end system username that can be used for connection.（目前来看，我现在是随便写的 admin 的账号）
   3. Zookeeper 和 hbase.authentication 的配置是和  `/usr/hdp/3.1.4.0-315/hbase/conf/hbase-site.xml` 中的配置保持一致
   
   ![image-20201130112641591](/images/ambari-20201130112641591.png)
   
   ![image-20201130112704671](/images/ambari-20201130112704671.png)
   
4. 添加 atlas 对 hbase tables、kafka topic 的访问 policies（可参看：[tag-based security with atlas + ranger](https://github.com/emaxwell-hw/Atlas-Ranger-Tag-Security/blob/master/README.md)）===》 暂时没做 kafka

   1. **创建 hbase 的 policies 时，必须给 all - table, column-family, column 加上 `hbase` 这个 user**，否则可能会遇到 403。这个原因是启动 metadata server 时，会执行 `cat /var/lib/ambari-agent/tmp/atlas_hbase_setup.rb | hbase shell -n` 这么一条命令，执行时，会切换到 `hbase` 这个用户，如果这里不加权限，这条命令就会执行失败

   ![image-20201130112445208](/images/ambari-20201130112445208.png)

ranger 的访问链接: http://ranger-server-host:6080/index.html (可以从 ambari ranger config 中看到)

### 生成 jar 包失败（报 no such file or directory)

> 在执行 `source /usr/hdp/current/atlas-server/conf/atlas-env.sh ; /usr/hdp/current/atlas-server/bin/atlas_start.py` 时报错，
>
> ```sh
>   File "/usr/hdp/current/atlas-server/bin/atlas_start.py", line 163, in 
>     returncode = main()
>   File "/usr/hdp/current/atlas-server/bin/atlas_start.py", line 73, in main
>     mc.expandWebApp(atlas_home)
>   File "/usr/hdp/3.1.4.0-315/atlas/bin/atlas_config.py", line 160, in expandWebApp
>     jar(atlasWarPath)
>   File "/usr/hdp/3.1.4.0-315/atlas/bin/atlas_config.py", line 213, in jar
>     process = runProcess(commandline)
>   File "/usr/hdp/3.1.4.0-315/atlas/bin/atlas_config.py", line 249, in runProcess
>     p = subprocess.Popen(commandline, stdout=stdoutFile, stderr=stderrFile, shell=shell)
>   File "/usr/lib64/python2.7/subprocess.py", line 711, in __init__
>     errread, errwrite)
>   File "/usr/lib64/python2.7/subprocess.py", line 1327, in _execute_child
>     raise child_exception
> OSError: [Errno 2] No such file or directory
> ```

通过在 `atlas_config.py` 中添加 log，发现最后是在执行 `/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b03-1.el7.x86_64/jre/bin/jar -xf /usr/hdp/3.1.4.0-315/atlas/server/webapp/atlas.war` 时报错，找不到的是 jar 命令. 原因是 jar 是 jdk 中的命令，而使用的默认 openjdk 其实只安装了 jre.

> ```sh
> # 查看所有的 openjdk 列表
> $ yum list | grep jdk
> 
> # 安装 jdk
> $ yum install java-1.8.0-openjdk.x86_64
> 
> # copy jar 到指定目录
> $ cp /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b03-1.el7.x86_64/bin/jar /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b03-1.el7.x86_64/jre/bin/
> ```

## Host Disk Usage alert

安装过程下载了挺多乱七八糟的东西，导致硬盘报警了

```sh
# 查看目前各文件系统的硬盘使用情况，如果设置的 80% 报警，则只要有某个文件系统的使用哪个超了，就会报警
$ df -h
文件系统                       容量  已用  可用 已用% 挂载点
devtmpfs                        32G     0   32G    0% /dev
tmpfs                           32G     0   32G    0% /dev/shm
tmpfs                           32G  824M   31G    3% /run
tmpfs                           32G     0   32G    0% /sys/fs/cgroup
/dev/mapper/centos-root         50G   31G   20G   61% /
/dev/sda1                     1014M  154M  861M   16% /boot
/dev/mapper/vg_data2-lv_data2  200G   55M  200G    1% /data02
/dev/mapper/centos-home         47G  444M   47G    1% /home
/dev/mapper/vg_data1-lv_data1  200G  4.9G  196G    3% /data01
tmpfs                          6.3G     0  6.3G    0% /run/user/0

# 查看现在系统中的文件占用情况，找大文件去清
$ du -hs /*
$ du -hs /root/*
$ du -hs /var/log/*
$ du -h /var/* -d 1 | sort -n -r
```



# 验证各服务可用

## Hive

### 创建文件夹（可选？？？）

[hive 验证可用](https://www.tutorialspoint.com/hive/hive_quick_guide.htm)

### 配置 ranger service & policies

[ranger-hive-service](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/authorization-ranger/content/resource_service_configure_a_hive_service.html)

service 命名

url: get from ambari-hive, or when you connect hive, it will show the whole connect string

policy 中 `all-database,table,column` 加上 `hive` user 



```
# 重启 spark 服务
Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

[hive managed & external tables](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ManagedandExternalTables)

acid table not supported now （acid new feature in hive，many others does not support it)

[spark-acids thoughts](https://github.com/Gowthamsb12/BigData-Blogs/blob/master/Spark_ACID)

![image-20201211154708672](/images/ambari-table-create-in-hive.png)

![image-20201211154728273](/images/ambari-table-create-in-spark.png)

I figured it out. Just set: `mapred.input.dir.recursive` and `hive.mapred.supports.subdirectories` to `true`. (Hive-site.xml)



 /warehouse/tablespace/managed/hive/ftms_ods.db/test_user6/delta_0000001_0000001_0000

/warehouse/tablespace/managed/hive/ftms_ods.db/test_user7/part-00000-2a86feec-be9a-413d-a696-8ff115d14075-c000.snappy.orc

## 包冲突

xbean-asm5-shaded-4.4.jar

xmlbeans-3.1.0.jar

xercesImpl-2.9.1.jar

xerces2-xsd11-2.11.1.jar

Xmlapis.jar

## Xml

删除 /user/ftms/lib/xercesImpl-2.9.1.jar 和 /user/ftms/lib/xml-apis-1.3.04.jar

```
ApplicationMaster: Unregistering ApplicationMaster with FAILED (diag message: User class threw exception: javax.xml.parsers.FactoryConfigurationError: Provider for class javax.xml.parsers.DocumentBuilderFactory cannot be created
	at javax.xml.parsers.FactoryFinder.findServiceProvider(FactoryFinder.java:311)
	at javax.xml.parsers.FactoryFinder.find(FactoryFinder.java:267)
	at javax.xml.parsers.DocumentBuilderFactory.newInstance(DocumentBuilderFactory.java:120)
	at org.apache.hadoop.conf.Configuration.asXmlDocument(Configuration.java:3442)
	at org.apache.hadoop.conf.Configuration.writeXml(Configuration.java:3417)
	at org.apache.hadoop.conf.Configuration.writeXml(Configuration.java:3388)
	at org.apache.hadoop.conf.Configuration.writeXml(Configuration.java:3384)
	at org.apache.hadoop.hive.conf.HiveConf.getConfVarInputStream(HiveConf.java:2410)
	at org.apache.hadoop.hive.conf.HiveConf.initialize(HiveConf.java:2703)
	at org.apache.hadoop.hive.conf.HiveConf.<init>(HiveConf.java:2657)
	at com.ftms.datapipeline.common.HiveMetaStoreUtil$.hiveConf(HiveMetaStoreUtil.scala:18)
	at com.ftms.datapipeline.common.HiveMetaStoreUtil$.createHiveMetaStoreClient(HiveMetaStoreUtil.scala:23)
	at com.ftms.datapipeline.common.HiveMetaStoreUtil$.getHiveMetaStoreClient(HiveMetaStoreUtil.scala:34)
	at com.ftms.datapipeline.common.HiveMetaStoreUtil$.getHiveTablePartitionCols(HiveMetaStoreUtil.scala:78)
	at com.ftms.datapipeline.common.HiveMetaStoreUtil$.getHiveTablePartitionColNames(HiveMetaStoreUtil.scala:73)
	at com.ftms.datapipeline.common.HiveDataSource$.buildInsertSql(HiveDataSource.scala:7)
	at com.ftms.datapipeline.common.HiveDataSource$.save(HiveDataSource.scala:42)
	at com.ftms.datapipeline.tasks.dwd.DCompany$.main(DCompany.scala:197)
	at com.ftms.datapipeline.tasks.dwd.DCompany.main(DCompany.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$4.run(ApplicationMaster.scala:721)
Caused by: java.lang.RuntimeException: Provider for class javax.xml.parsers.DocumentBuilderFactory cannot be created
	at javax.xml.parsers.FactoryFinder.findServiceProvider(FactoryFinder.java:308)
	... 23 more
Caused by: java.util.ServiceConfigurationError: javax.xml.parsers.DocumentBuilderFactory: Provider org.apache.xerces.jaxp.DocumentBuilderFactoryImpl not found
	at java.util.ServiceLoader.fail(ServiceLoader.java:239)
	at java.util.ServiceLoader.access$300(ServiceLoader.java:185)
	at java.util.ServiceLoader$LazyIterator.nextService(ServiceLoader.java:372)
	at java.util.ServiceLoader$LazyIterator.next(ServiceLoader.java:404)
	at java.util.ServiceLoader$1.next(ServiceLoader.java:480)
	at javax.xml.parsers.FactoryFinder$1.run(FactoryFinder.java:294)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.xml.parsers.FactoryFinder.findServiceProvider(FactoryFinder.java:289)
	... 23 more
```



## Spark

### 在 yarn-client 模式下运行 spark

会出现 `Library directory '...\assembly\target\scala-2.11\jars' does not exist; make sure Spark is built.`，这个大致原因是 yarn-client 模式下

[`spark.yarn.jar`和`spark.yarn.archive`的使用](https://www.jianshu.com/p/016fbd2a421b)

> Running Spark on YARN requires a binary distribution of Spark which is built with YARN support. Binary distributions can be downloaded from the [downloads page](https://spark.apache.org/downloads.html) of the project website. To build Spark yourself, refer to [Building Spark](https://spark.apache.org/docs/latest/building-spark.html).
>  To make Spark runtime jars accessible from YARN side, you can specify `spark.yarn.archive` or `spark.yarn.jars`. For details please refer to [Spark Properties](https://spark.apache.org/docs/latest/running-on-yarn.html#spark-properties). If neither `spark.yarn.archive` nor `spark.yarn.jars` is specified, Spark will create a zip file with all jars under `$SPARK_HOME/jars` and upload it to the distributed cache

# Install ClickHouse

```sh
# 安装 clickhouse
$ rpm -ivh clickhouse-server-common-19.4.3.11-1.el6.x86_64.rpm
$ rpm -ivh clickhouse-common-static-19.4.3.11-1.el6.x86_64.rpm
$ rpm -ivh clickhouse-server-19.4.3.11-1.el6.x86_64.rpm
$ rpm -ivh clickhouse-client-19.4.3.11-1.el6.x86_64.rpm

# 启动服务
$ service clickhouse-server start
Start clickhouse-server service: Path to data directory in /etc/clickhouse-server/config.xml: /var/lib/clickhouse/

# 通过 client 验证运行成功
$ clickhouse-client
ClickHouse client version 19.4.3.11.
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 19.4.3 revision 54416.

master :) select 1

SELECT 1

┌─1─┐
│ 1 │
└───┘

1 rows in set. Elapsed: 0.001 sec.
```

## client 启动失败

> error detail:
>
> ```
> ClickHouse client version 20.8.3.18.
> Connecting to localhost:9000 as user default.
> Code: 102. DB::NetException: Unexpected packet from server localhost:9000 (expected Hello or Exception, got Unknown packet)
> ```

这个错表示，clickhouse-client 收到返回了，但是返回的结果是非预期错误。这个错一般是由于端口占用。可以通过 `netstat -antp|grep LIST|grep 9000` 查询。

### Solution：

```sh
# 更新 clickHouse 的端口为 9011
$ vi /etc/clickhouse-server/config.xml
:%s/9000/9011

# 启动时带上端口号
$ clickhouse-client --port 9011
```

# 安装 python3

https://www.python.org/downloads/release/python-361/

[reference doc](https://www.jianshu.com/p/758b592387d1)

```sh
# 解压并安装 python3
$ tar -xf Python-3.?.?.tar.xz
$ cd Python-3.?.?
$ ./configure
$ make altinstall
$ python3.x -V

# 创建软链
$ which python3.6
$ ln -s /usr/local/bin/python3.6.1 /usr/bin/python3
$ python3

# 安装 pip
$ python3 -m ensurepip
$ pip3
```

> 如果安装 pip 时报错 zlib not found：
>
> ```sh
> # 安装 zlib 相关依赖包
> $ yum -y install zlib*
> 
> # 进入 python安装包,修改Module路径的setup文件，Modules/Setup.dist （或者 Modules/Setup） 文件
> $ vi Module/Setup
> #zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
> 去掉注释
>      zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
> ```

# 安装 airflow

为了方便管理，可以安装个 mpack，然后就可以从 ambari 安装、管理、监控 airflow

* [install airflow from ambari](https://miho120.medium.com/integrating-apache-airflow-with-apache-ambari-ccab2c90173)
* [git](https://github.com/miho120/ambari-airflow-mpack)

这个插件在安装/启动时，其实就是执行了 `/var/lib/ambari-agent/cache/common-services/AIRFLOW/1.10.0/package/scripts/airflow_scheduler_control.py` ，脚本中提供了安装、启动、停止。安装时，本质是执行了以下内容：

```sh
$ pip install apache-airflow[all]==1.9.0 apache-airflow[celery]==1.9.0
```

> 相关的安装、启动等脚本都在 `/var/lib/ambari-agent/cache/common-services/AIRFLOW/1.10.0/package/scripts` 目录下

但上述过程只能在有线环境执行，离线环境还是得自己下。

## install airflow offline

[airflow installation 官方](https://airflow.apache.org/docs/stable/installation.html)

```sh
# 先下载相关包
$ mkdir airflow-install
$ cd airflow-install
$ pip download 'apache-airflow[all]==1.10.12' \
--constraint  'https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.6.txt'

# 接下来更新上述 `airflow_scheduler_control.py` 中的安装脚本，选择从本地文件安装即可
$ vi /var/lib/ambari-agent/cache/common-services/AIRFLOW/1.10.0/package/scripts/airflow_scheduler_control.py
$ vi /var/lib/ambari-agent/cache/common-services/AIRFLOW/1.10.0/package/scripts/airflow_webserver_control.py
# 下边这个命令是上述脚本的内容，这里只是介绍一下执行的命令
$ pip install apache-airflow==1.10.12 --no-index -f ./

# 过程中可能会有些包缺，例如 docutils、pytest-runner 等，从 https://pypi.org/ 下载相应的 .whl 文件，放到该目录下
# 然后通过 --no-index -f ./ 或者 --no-index -f ./xxx.whl 来安装即可
$ pip install pytest-runner --no-index -f ./
```

实际操作时，下载完 airflow 后，就更新 airflow_scheduler_control.py 和 airflow_webserver_control.py （两个文件的 install method 是一模一样的，按同样的方式修改就行）


```python
################# 源文件
     11         def install(self, env):
     12                 import params
     13                 env.set_params(params)
     14                 print('*' * 30)
     15                 print(env)
     16                 print('*' * 30)
     17                 self.install_packages(env)
     18                 Logger.info(format("Installing Airflow Service"))
     19                 Execute(format("pip install --upgrade {airflow_pip_params} pip"))
     20                 Execute(format("pip install --upgrade {airflow_pip_params} setuptools"))
     21                 Execute(format("pip install --upgrade {airflow_pip_params} docutils pytest-runner Cython==0.28"))
     22                 Execute(format("export SLUGIFY_USES_TEXT_UNIDECODE=yes && pip install --upgrade {airflow_pip_params} --ignore-        installed apache-airflow[all]==1.10.0"))
     23                 Execute(format("export SLUGIFY_USES_TEXT_UNIDECODE=yes && pip install --upgrade {airflow_pip_params} --ignore-        installed apache-airflow[celery]==1.10.0"))
     24                 Execute(format("chmod 755 /bin/airflow /usr/bin/airflow"))
     25                 Execute(format("useradd {airflow_user}"), ignore_failures=True)
     26                 Execute(format("mkdir -p {airflow_home}"))
     27                 airflow_make_startup_script(env)
     28                 Execute(format("chown -R {airflow_user}:{airflow_group} {airflow_home}"))
     29                 Execute(format("export AIRFLOW_HOME={airflow_home} && airflow initdb"),
     30                         user=params.airflow_user
     31                 )

################### 修改后的
     11         def install(self, env):
     12                 import params
     13                 env.set_params(params)
    # 这里是去安装 pip，已经装好了，就不装了
     14 #               self.install_packages(env)
     15                 Logger.info(format("Installing Airflow Service"))
     16                 Execute(format("pip install --upgrade {airflow_pip_params} pip"))
     17                 Execute(format("pip install --upgrade {airflow_pip_params} setuptools"))
    # 从指定目录找安装包 --no-index -f ./xxx
     18                 Execute(format("pip install --upgrade {airflow_pip_params} docutils pytest-runner Cython --no-index -f /install/python_install/airflow-install/"))
    # 从指定目录找安装包 --no-index -f ./xxx，并更新版本为 1.10.12，且仅安装 minimal packages
     19                 Execute(format("export SLUGIFY_USES_TEXT_UNIDECODE=yes && pip install --upgrade {airflow_pip_params} --ignore-installed apache-airflow==1.10.12 --no-index -f /install/python_install/airflow-install/"))
    # 从指定目录找安装包 --no-index -f ./xxx，并更新版本为 1.10.12
     20                 Execute(format("export SLUGIFY_USES_TEXT_UNIDECODE=yes && pip install --upgrade {airflow_pip_params} --ignore-installed apache-airflow[celery]==1.10.12 --no-index -f /install/python_install/airflow-install/"))
    # 目前系统中默认是安装在 /usr/local/bin/airflow
     21                 Execute(format("chmod 755 /usr/local/bin/airflow"))
     22                 Execute(format("useradd {airflow_user}"), ignore_failures=True)
     23                 Execute(format("mkdir -p {airflow_home}"))
     24                 airflow_make_startup_script(env)
     25                 Execute(format("chown -R {airflow_user}:{airflow_group} {airflow_home}"))
     26                 Execute(format("export AIRFLOW_HOME={airflow_home} && airflow initdb"),
     27                         user=params.airflow_user
     28                 )
```

# Review

**Admin Name** : admin 

**Cluster Name** : ftms_hdp_qat 

**Total Hosts** : 3 (3 new) 

**Repositories**:

redhat7 (HDP-3.1): 
 http://10.66.18.11/ambari/HDP/centos7/3.1.4.0-315/

redhat7 (HDP-3.1-GPL): 
 http://10.66.18.11/ambari/HDP-GPL/centos7/3.1.4.0-315/

redhat7 (HDP-UTILS-1.1.0.22): 
 http://10.66.18.11/ambari/HDP-UTILS/centos7/1.1.0.22/

**Services:**

*HDFS* 

DataNode : 2 hosts 

NameNode : master 

NFSGateway : 0 host 

SNameNode : slave1 

*YARN + MapReduce2* 

Timeline Service V1.5 : slave1 

NodeManager : 1 host 

ResourceManager : master 

Timeline Service V2.0 Reader : master 

Registry DNS : master 

*Tez* 

Clients : 1 host 

*Hive* 

Metastore : slave1 

HiveServer2 : slave1 

Database : Existing MySQL / MariaDB Database 

*HBase* 

Master : master 

RegionServer : 1 host 

Phoenix Query Server : 0 host 

*Sqoop* 

Clients : 1 host 

*Oozie* 

Server : master 

Database : Existing MySQL / MariaDB Database 

*ZooKeeper* 

Server : 3 hosts 

*Infra Solr* 

Infra Solr Instance : master 

*Ambari Metrics* 

Metrics Collector : slave2 

Grafana : master 

*Atlas* 

Metadata Server : slave1 

*Kafka* 

Broker : master 

*Ranger* 

Admin : slave1 

Tagsync : 1 host 

Usersync : slave1 

*SmartSense* 

Activity Analyzer : master 

Activity Explorer : master 

HST Server : master 

*Spark2* 

Livy for Spark2 Server : 0 host 

History Server : master 

Thrift Server : 0 host 


