---
title: flink
toc: true
date: 2021-10-14 10:57:03
tags:
  - big data
---

## flinkx

[flinkx](https://gitee.com/dtstack_dev_0/flinkx/blob/1.10_release/README.md) 是袋鼠云内部广泛使用的基于flink的分布式离线和实时的数据同步框架，实现了多种异构数据源之间高效的数据迁移

不同的数据源头被抽象成不同的Reader插件，不同的数据目标被抽象成不同的Writer插件。理论上，FlinkX框架可以支持任意数据源类型的数据同步工作。作为一套生态系统，每接入一套新数据源该新加入的数据源即可实现和现有的数据源互通。

FlinkX是一个基于Flink的批流统一的数据同步工具，既可以采集静态的数据，比如MySQL，HDFS等，也可以采集实时变化的数据，比如MySQL binlog，Kafka等。

![img](https://gitee.com/dtstack_dev_0/flinkx/raw/1.10_release/docs/images/template.png)

在底层实现上，FlinkX依赖Flink，数据同步任务会被翻译成StreamGraph在Flink上执行，基本原理如下图：

![img](https://gitee.com/dtstack_dev_0/flinkx/raw/1.10_release/docs/images/diagram.png)

### flinkx vs datax vs flinkcdc

[flinkx vs datax vs flinkcdc](https://www.icode9.com/content-4-1098731.html)

简单来说：

1. flinkx 和 flink cdc 是基于 flink 的数据同步工具，批流一体。只是 flinkx 是第三方工具，重，支持数据源多，而 flink-cdc 是 flink 官方的一组连接器，轻，使用方便，但是支持的数据源也少（mysql/kafka/postgres）
2. [datax](https://github.com/alibaba/DataX) 是离线数据同步工具，只有离线数据。flinkx 有点对标这个的意思，而且 flinkx 使用 yarn 做任务调度，和 hadoop 生态无缝结合。

> DataX 是阿里云 [DataWorks数据集成](https://www.aliyun.com/product/bigdata/ide) 的开源版本，在阿里巴巴集团内被广泛使用的离线数据同步工具/平台。DataX 实现了包括 MySQL、Oracle、OceanBase、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、Hologres、DRDS 等各种异构数据源之间高效的数据同步功能。
