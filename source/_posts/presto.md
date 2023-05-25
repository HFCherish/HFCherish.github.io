---
title: presto
toc: true
tags:
  - big data
date: 2021-05-10 11:10:15
---


# 背景

Facebook的数据仓库存储在少量大型Hadoop/HDFS集群。Hive是Facebook在几年前专为Hadoop打造的一款数据仓库工具。在以前，Facebook的科学家和分析师一直依靠Hive来做数据分析。但Hive使用MapReduce作为底层计算框架，是专为批处理设计的。但随着数据越来越多，使用Hive进行一个简单的数据查询可能要花费几分到几小时，显然不能满足交互式查询的需求。Facebook也调研了其他比Hive更快的工具，但它们要么在功能有所限制要么就太简单，以至于无法操作Facebook庞大的数据仓库。

2012年开始试用的一些外部项目都不合适，他们决定自己开发，这就是Presto。2012年秋季开始开发，目前该项目已经在超过 1000名Facebook雇员中使用，运行超过30000个查询，每日数据在1PB级别。Facebook称Presto的性能比Hive要好上10倍多。2013年Facebook正式宣布开源Presto。

# 定位

[presto 官网](https://prestodb.io/)

> Presto is an open source distributed SQL query engine for running interactive analytic queries against data sources of all sizes ranging from gigabytes to petabytes.

Presto是一个分布式的查询引擎，本身并不存储数据，但是可以接入多种数据源，并且支持跨数据源的级联查询。Presto是一个OLAP的工具，擅长对海量数据进行复杂的分析；但是对于OLTP场景，并不是Presto所擅长，所以不要把Presto当做数据库来使用。

和大家熟悉的Mysql相比：首先Mysql是一个数据库，具有存储和计算分析能力，而Presto只有计算分析能力；其次数据量方面，Mysql作为传统单点关系型数据库不能满足当前大数据量的需求，于是有各种大数据的存储和分析工具产生，Presto就是这样一个可以满足大数据量分析计算需求的一个工具。

Hive是一个基于HDFS(分布式文件系统)的一个数据库，具有存储和分析计算能力， 支持大数据量的存储和查询。Hive 作为数据源，结合Presto分布式查询引擎，这样大数据量的查询计算速度就会快很多

# 数据源

Presto需要从其他数据源获取数据来进行运算分析，它可以连接多种数据源，包括Hive、RDBMS（Mysql、Oracle、Tidb等）、Kafka、MongoDB、Redis等

一条Presto查询可以将多个数据源的数据进行合并分析。
比如：select * from a join b where a.id=b.id;，其中表a可以来自Hive，表b可以来自Mysql。

# 数据模型

[preto concepts](https://prestodb.io/docs/current/overview/concepts.html)

Presto使用Catalog、Schema和Table这3层结构来管理数据。

---- Catalog:就是数据源。Hive是数据源，Mysql也是数据源，Hive 和Mysql都是数据源类型，可以连接多个Hive和多个Mysql，每个连接都有一个名字。一个Catalog可以包含多个Schema，大家可以通过show catalogs 命令看到Presto连接的所有数据源。
---- Schema：相当于一个数据库实例，一个Schema包含多张数据表。show schemas from 'catalog_name'可列出catalog_name下的所有schema。
---- Table：数据表，与一般意义上的数据库表相同。show tables from 'catalog_name.schema_name'可查看'catalog_name.schema_name'下的所有表。

在Presto中定位一张表，一般是catalog为根，例如：一张表的全称为 hive.test_data.test，标识 hive(catalog)下的 test_data(schema)中test表。

        kadmin.local addprinc -randkey ${USER}@FTMS.COM
        kadmin.local xst -k keytabs/${USER}.keytab ${USER}@FTMS.COM

# OLAP 引擎对比

为什么presto查询速度比Hive快？

* presto是常驻任务，接受请求立即执行，全内存并行计算；hive需要用yarn做资源调度，接受查询需要先申请资源，启动进程，并且中间结果会经过磁盘。

[开源OLAP引擎测评报告(SparkSql、Presto、Impala、HAWQ、ClickHouse、GreenPlum)](https://www.cnblogs.com/bonelee/p/12878451.html)

多表

![img](http://geek.analysys.cn/static/upload/dailidong/2018-12-11/08519d86-cd1e-420d-a2cc-80e1b5fef8c1.jpeg)

单表：

![img](http://geek.analysys.cn/static/upload/dailidong/2018-12-11/bf0bd90a-1209-4be7-b3a2-fb28fe29811a.jpeg)

# install

## ambari 集成

https://prestodb.io/ambari-presto-service/

https://www.codenong.com/js87725e1b2144/

https://www.jianshu.com/p/291e3ee3f981

https://blog.csdn.net/qq_29248935/article/details/98870687

https://cloud.tencent.com/developer/article/1158362

# presto known limitations

[limitations](https://docs.treasuredata.com/display/public/PD/Presto+Known+Limitations)

[Presto内存管理与相关参数设置](https://zhuanlan.zhihu.com/p/89381163)
