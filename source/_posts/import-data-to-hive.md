---
title: 数据导入 hive
toc: true
tags:
  - big data
  - hadoop
date: 2020-11-11 11:58:27
---

# ftp .csv 文件导入

可以先将文件弄到 HDFS，然后创建/更新 hive 表来关联到 HDFS 文件。

将文件弄到 HDFS有以下一些方法：

1. **ftp -> local -> hdfs:** 将文件先下载到本地，再通过 hdfs 命令拷贝到 hdfs 中
2. **ftp -> hdfs**: 直接连接 FTP，将文件拷到 hdfs 中，省却本地拷贝
3. **已有的数据采集工具**：使用实时数据流处理系统，来实现不同系统之间的流通

## 一、ftp -> local ->hdfs

几种方案：

1.  `hadoop fs -get ftp://uid:password@server_url/file_path temp_file | hadoop fs -moveFromLocal tmp_file hadoop_path/dest_file` 

2. 参照[这个实现](https://community.cloudera.com/t5/Support-Questions/How-read-ftp-server-files-and-load-into-hdfs-in-incremental/m-p/223519/highlight/true#M185384)用 python 包从 ftp 中读，然后用 hdfs 命令写到 hdfs

   ```python
   from urllib.request import urlopen
   from hdfs import InsecureClient
   
   # You can also use KerberosClient or custom client
   namenode_address = 'your namenode address'
   webhdfs_port = 'your webhdfs port' # default for Hadoop 2: 50070, Hadoop 3: 9870
   user = 'your user name'
   client = InsecureClient('http://' + namenode_address + ':' + webhdfs_port, user=user)
   
   ftp_address = 'your ftp address'
   hdfs_path = 'where you want to write'
   with urlopen(ftp_address) as response:
       content = response.read()
       # You can also use append=True
       # Further reference: https://hdfscli.readthedocs.io/en/latest/api.html#hdfs.client.Client.write
       with client.write(hdfs_path) as writer:
           writer.write(content
   ```

3. 参考 [ftp 提取文件到 hdfs](https://blog.csdn.net/yiluohan0307/article/details/79364525)

## 二、ftp -> hdfs

几种方案：(参考 [ftp 提取文件到 hdfs](https://blog.csdn.net/yiluohan0307/article/details/79364525))

1. 用 [FTP To HDFS](http://hadoop101.blogspot.com/?view=classic) 连接 ftp，把文件直接放到 hdfs

2. HDFS dfs -cp: 简单快速，但不显示进度，适用于小文件

   ```sh
   $ hdfs dfs –cp [ftp://username:password@hostname/ftp_path] [hdfs:///hdfs_path]
   ```

3. Hadoop distcp: 分布式提取，快，能显示拷贝进度，不支持流式写入（即拷贝的文件不能有其他程序在写入），适合大量文件或大文件的拷贝

   ```sh
   $ hadoop distcp [ftp://username:password@hostname/ftp_path] [hdfs:///hdfs_path]
   ```

## 三、已有的数据采集工具

### 文件导入

1. [apache NiFi](https://nifichina.github.io/general/GettingStarted.html#%E6%9C%89%E5%93%AA%E4%BA%9B%E7%B1%BB%E5%88%AB%E7%9A%84%E5%A4%84%E7%90%86%E5%99%A8) 来实现不同系统之间的流通，似乎拷贝完，会直接删除 ftp 上的文件

2. Apache Flume是一个分布式、可靠、高可用的日志收集系统，支持各种各样的数据来源。基于流式数据，适用于日志和事件类型的数据收集，重构后的Flume-NG版本中一个agent（数据传输流程）中的source（源）和sink（目标）之间通过channel进行链接，同一个源可以配置多个channel。多个agent还可以进行链接组合共同完成数据收集任务，使用起来非常灵活。

   [flume 采集 ftp 文件 上传到 hadoop](https://blog.csdn.net/qq_39160721/article/details/80255588) 使用 [spooldir source](https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#spooling-directory-source)（不确定是不是能用）, 也可以使用第三方 source 组件 [flume-ftp-source](https://github.com/keedio/flume-ftp-source)

   > Flume 也支持 sql source 的流式导入（使用 [flume-ng-sql-source](https://github.com/keedio/flume-ng-sql-source) 插件），并提供对数据进行简单处理，并写到各数据接收方的能力。因此它的实时性更好。

3. DataX：阿里的开源框架，本身社区不太活跃，但有很多 fork 再改的，似乎架构不错

4. Gobllin: Gobblin是用来整合各种数据源的通用型ETL框架，Gobblin的接口封装和概念抽象做的很好，作为一个ETL框架使用者，我们只需要实现我们自己的Source，Extractor，Conventer类，再加上一些数据源和目的地址之类的配置文件提交给Gobblin就行了。Gobblin相对于其他解决方案具有普遍性、高度可扩展性、可操作性。

5. kettle：一款开源的ETL工具

### 其他数据源（非 FTP 文件）

1. Apache Sqoop：RDBMS <--> HDFS
2. Aegisthus：针对 Cassandra 数据源
3. mongo-hadoop：针对 mongodb 数据源

# 数据导入需要关注的问题

1. **数据源都有哪些？**
   1. 结构化（sql）、半结构化（json, xml...)、非结构化（video、image、file...)
   2. 日志数据（csv)、业务数据
2. **是否可以直接连接数据库？**
   1. 针对关系型数据，如果可连接数据库，可以通过 sqoop 导入数据到 hive
      1. 增量式导入？？
   2. 针对关系型数据，如果不能连接数据库：
      1. **是否可以默认周期性导出符合特定标准的 .csv 文件？**
         1. 如果数据库导出 dump 文件，再将 dump 文件导入到 hadoop，则比较麻烦，以 oracle 为例，可能需要使用 COPYToBDA 来创建 hive table [Query Oracle Dump File on BDA Using Copy2BDA](https://weidongzhou.wordpress.com/2016/11/12/data-query-between-bda-and-exadata-part-4-query-oracle-dump-file-on-bda-using-copy2bda/) ，或者将 dump 文件先导入到一个 temp oracle 数据库中，再用 sqoop 导入到 hive
         2. 如果数据库周期性导出 .csv 文件，将这些 .csv 文件使用上述工具（flume 等）导入到 hive，需要关注增量式导出和导入
            1. 增量式导出：文件的组织结构、命名规范 ，.csv 内 record 要求包含 modified date, delete date（在增量式导入时，需要基于这些时间来合并表）
            2. 增量式导入：将新增的 .csv 文件作为 hive external table，然后通过中间 view 来合并基表和incremental 表，并更新基表、清空 incremental 表。[Incrementally Updating a Table](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.6.5/bk_data-access/content/incrementally-updating-hive-table-with-sqoop-and-ext-table.html)
3. 导入周期和实时性需求
   1. **哪些需要每天批量导入、哪些需要流式实时导入**
   2. **哪些需要全量导入、哪些需要增量式导入？**
4. **如何实现增量式导入？删除的数据是否有删除标识（软删除）？**
   1. 如果用 sqoop，参考 [sqoop 增量导入](https://hfcherish.github.io/2020/11/10/sqoop/)，不支持对删除数据的处理
   2. 如果用 flume
      1. 如果是 sql source，使用 [flume-ng-sql-source](https://github.com/keedio/flume-ng-sql-source), 对于 mysql 可以通过 query `agent.sources.sqlSource.custom.query` 来获取增量 source
      2. 如果是文件导入，则需要通过 [Incrementally Updating a Table](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.6.5/bk_data-access/content/incrementally-updating-hive-table-with-sqoop-and-ext-table.html) 来合并表
   3. Spark SQL

