---
title: MPP (Massively Parallel Processing)
toc: true
tags:
  - big data
date: 2020-11-10 10:12:56
---


# Concept

[5分钟了解MPP数据库](https://zhuanlan.zhihu.com/p/148621151)

MPP (Massively Parallel Processing)，即大规模并行处理。简单来说，MPP是将任务并行的分散到多个服务器和节点上，在每个节点上计算完成后，将各自部分的结果汇总在一起得到最终的结果(与Hadoop相似，但主要针对大规模关系型数据的分析计算)。

## MPP架构特征

- 任务并行执行;
- 数据分布式存储(本地化);
- 分布式计算;
- 私有资源;
- 横向扩展;
- Shared Nothing架构。

# MPPDB v.s. Hadoop

[知乎-为什么说HADOOP扩展性优于MPP架构的关系型数据库？](https://www.zhihu.com/question/22799482/answer/81615602)

hadoop 和 MPPDB **最大的区别在于：对数据管理理念的不同。** 

1. HDFS/Hadoop 对于数据管理是**粗放型管理**，以一个文件系统的模式，让用户根据文件夹层级，把文件直接塞到池子里。处理也**以批处理为主，就是拼命 scan**。如果想在一大堆数据里找符合条件的数据，hadoop 就是粗暴的把所有文件从头到尾 scan 一遍，因为对于这些文件他没有索引、分类等，他管的少，知道的也少，用的时候每次就要全 scan。
2. 数据库的本质在于数据管理，对外提供在线访问、增删改查等一系列操作。数据库的**内存管理比较精细**，有一套很完善的数据管理和分布体系。如果想在一大堆数据里找符合条件的数据，他可以根据分区信息先落到某个机器，再根据多维分区落到某个文件，再在文件里通过索引数据页的树形结构查询，可以直接定位到记录。
3. 因为这样的基本理念不同，使得 hadoop 的扩展只需要简单的增加机器，内部平衡和迁移 data block；而数据库的扩充则涉及到数据拓扑结构的变更、或者不同机器间数据的迁移，当变化迁移的时候，依然需要维护分区、索引等，这种复杂度就比粗放的 HDFS 要高很多了。目前两者的存储模型不同。hadoop 用的是 HDFS，而 MPP 需要自己切分扩展。HDFS 扩展是通过元数据做的，name node 存元数据，增加一个节点，修改元数据就行，所以 HDFS 的扩展能力受到管理元数据的机器（name node） 的性能限制，一般来说可以到 10k 的规模。但 MPP 采用没有中心节点的存储模型，比如 hash，每次增加节点，都需要 rehash，规模增加到几百台的时候，扩展能力就有下降下来了。

通常来讲，MPP数据库有对SQL的完整兼容和一些事务的处理能力，Hadoop在处理非结构化和半结构化数据上具备优势，所以MPP适合多维度数据自助分析、数据集市等；Hadoop适合海量数据存储查询、批量数据ETL、非结构化数据分析(日志分析、文本分析)等海量**数据批处理**应用要求。但通过 sql 还是 map-reduce 来查，其实只是一种查询形式。目前也有很多 sql on hadoop 的方案，例如 impala （sql on hadoop，其实是一个 MPP engine，所以它的查询性能会更好，提供更低的延迟和更少的处理时间）、spark Sql 等。现在Spark的重点都在Spark SQL，因为它已经不仅仅是SQL了，而是新的 "spark core"。（详见最后链接中Reynold Xin对此的解释）

> Spark SQL is not just about SQL. It turns out the primitives required for general data processing (eg ETL) are not that different from the relational operators, and that is what Spark SQL is. Spark SQL is the new Spark core with the Catalyst optimizer and the Tungsten execution engine, which powers the DataFrame, Dataset, and last but not least SQL.



![img](https://pic2.zhimg.com/80/v2-2195887a063e35952ffb9c94d3e18755_720w.jpg)

# 常用的MPP数据库有哪些

- 自我管理的数据仓库
  - HPE vertica
  - MemSql
  - Teradata
- 按需 MPP 数据库
  - aws redshift
  - azure sql 数据仓库
  - google bigQuery
- **GreenPlum**
- Sybase IQ
- TD Aster Data

