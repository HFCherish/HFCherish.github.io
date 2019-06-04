---
layout: 'yarn,'
title: hive introduction
tags:
  - hadoop
  - hive
  - data warehouse
toc: true
date: 2019-01-03 14:50:02
---


[apache hive](https://cwiki.apache.org/confluence/display/Hive/Home#Home-HiveDocumentation) 是一个 data warehouse 应用。支持分布式存储的大数据读、写和管理，并且支持使用标准的 SQL 语法查询。

hive 并没有固定的数据存储方式。自带的是 csv（comma-separated value）和 tsv (tab-separated values) connectors，也可以使用 connector for other formats。

## database v.s. warehouse

参见 [the difference between database and data warehouse](https://panoply.io/data-warehouse-guide/the-difference-between-a-database-and-a-data-warehouse/)

### database：

存储具体的业务数据，完善支持 concurrent transaction 操作（CRUD）。

database contains highly detailed data as well as a detailed relational views. Tables are normalized to achieve efficient storage, concurrent transaction processing, as well as return quick query results.

* **主要用于 OLTP (online trancaction processing)**。
* **use a normalized structure**. 即通常会组织成 table、row、column，冗余信息很少（比如三张表 product、color、product-color），所以节省空间。在查询时就需要通过复杂的 join 来实现，所以分析性的查询会比较耗时
* **no historical data**. 主要处理 transaction 数据，只保存现在的数据，进行的查询和分析也是基于现有数据。即它的分析是 static one-time reports
* **optimization 主要是优化写速度、读速度**。复杂分析因为涉及很多 join，其性能提升也是一个主要的问题。
* **经常需要满足关系型数据库的 ACID 原则**（atomicity, consistency, isolation, and durability）。所以它需要支持并发操作下的数据完整性。对 concurrent transaction 的支持要求比较高。

### data warehouse

将企业中的各种数据收集起来，重新组织，对这些数据做高效 ***分析***

> A [data warehouse](https://panoply.io/data-warehouse-guide) is a system that pulls together data from many different sources within an organization for reporting and analysis. The reports created from complex queries within a data warehouse are used to make business decisions.
>
> The primary focus of a data warehouse is to provide a correlation between data from existing systems, i.e., product inventory stored in one system, purchase orders for a specific customer, stored in another system. Data warehouses are used for online analytical processing (OLAP), which uses complex queries to analyze rather than process transactions.

* **主要用于 OLAP (online analysis processing)**. 它收集企业内各个数据源的数据，建立数据关联，对这些数据做复杂的查询分析，以辅佐业务决策。
* **use a denormalized structure**. 它收集多个相关数据源的数据，将这些 table [denormailize](https://searchdatamanagement.techtarget.com/definition/denormalization)、transform，获得 summarized data、multidimentional views，并基于这些数据实现快速分析和查询。它不在乎冗余，相反，很多时候正是通过冗余重新组织数据，使得查询更方便。
* **store historical data**. data warehouse 主要是用于分析的，所以通常会存储历史数据，以实现对历史数据和现有数据的对比分析。
* **optimization 主要是查询响应速度**。它对大数据做分析，响应速度是主要的衡量标准。
* **一般不支持高并发操作**。支持一定并发，但支持程度远不如 database

# installation

See [hadoop: setting up a single-node cluster](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html), [GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

Hive relies on hadoop. And we need a db (eg. mysql) to store hive metadata. So the prerequisites are:

* **hadoop installed**
* **mysql installed**: to store hive metadata
* **java installed**: ??
* **ssh installed and sshd running**: when running hadoop scripts and managing remote hadoop daemons, it use ssh to authenticate.

### [install hadoop on mac](https://hfcherish.github.io/2019/01/07/hadoop/)

### [install hive on mac](https://www.cnblogs.com/micrari/p/7067968.html)

> After init mysql, you may find that you can't connect mysql using '-uhive -pxxx'. Then try to grant privileges to `'hive'@'%'` instead of `'hive'@'localhost'`. Use wildcard `%` to match all hosts.

After installation, can try the [simple example](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SimpleExampleUseCases) to see how to conduct analysis on hive.

### set env

To use hadoop and hive conveniently, set the bin in Path. Just add the follow config into `~/.zshrc`, and then source it `source ~/.zshrc`.

```sh
export HADOOP_HOME=/usr/local/Cellar/hadoop/3.1.1/libexec
export HIVE_HOME=/usr/local/Cellar/hive/3.1.1/libexec
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export PATH=$PATH:$HIVE_HOME/bin
```



### [running using beeline](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveServer2andBeeline.1)

beeline is a new hive client to replace the deprecated HiveCli. With beeline, you can execute write, load, query, etc. on hive.

To connect simply, type the following:

```
$ hiveserver2
$ beeline -u jdbc:hive2://
```

To create, alter database/table/column/etc. on hive, see [Hive Data Definition Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL).

To get the query commands, see [LanguageManual Select](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select)

To load data from file, insert, delete, merge, update data, see [DML (data manipulation language)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML)

Other non-sql commands to use in HiveQL or beeline, see [LanguageManual Commands](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Commands). The [Hive Resources](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources) related commands are non-sql commands.

# analysis on hive

When you start a sql function (eg. `select count(*) from xxx`), it in fact  starts an map-reduce job based on hadoop to search among all datanodes. Such functions are simple analysis implemented by hive.

>Hive compiler generates map-reduce jobs for most queries. These jobs are then submitted to the Map-Reduce cluster indicated by the variable:

```
  mapred.job.tracker
```

For complex analysis, you may need to write custom mappers (map data) & reducers (collect data) scripts. Use[`TRANSFORM`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Transform) keyword in hive to achieve this.

For example, the `weekday_mapper.py` to convert `unixtime` to `weekday`:

```python
import sys
import datetime

for line in sys.stdin:
  line = line.strip()
  userid, movieid, rating, unixtime = line.split('\t')
  weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
  print '\t'.join([userid, movieid, rating, str(weekday)])
```

And then use the script:

```sql
CREATE TABLE u_data_new (
  userid INT,
  movieid INT,
  rating INT,
  weekday INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t';

add FILE weekday_mapper.py;

INSERT OVERWRITE TABLE u_data_new
SELECT
  TRANSFORM (userid, movieid, rating, unixtime)
  USING 'python weekday_mapper.py'
  AS (userid, movieid, rating, weekday)
FROM u_data;

SELECT weekday, COUNT(*)
FROM u_data_new
GROUP BY weekday;
```

# storage on hive

Hive relies on Hadoop. The data in hive is saved in hdfs in fact. And the metadata is saved in mysql (the db can be configured). When check `localhost:9870`, you can see a new folder `/user/hive/warehouse`. All tables in hive are dirs in `/user/hive/warehouse`.

# [architecture](https://cwiki.apache.org/confluence/display/Hive/Design)

![hive architecture](https://cwiki.apache.org/confluence/download/attachments/27362072/system_architecture.png?version=1&modificationDate=1414560669000&api=v2)