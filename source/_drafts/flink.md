---
title: flink
toc: true
date: 2021-10-14 10:57:03
tags:
  - big data
---

# flink architecture

![The processes involved in executing a Flink dataflow](https://nightlies.apache.org/flink/flink-docs-master/fig/processes.svg)

# flinkx

[flinkx](https://gitee.com/dtstack_dev_0/flinkx/blob/1.10_release/README.md) 是袋鼠云内部广泛使用的基于flink的分布式离线和实时的数据同步框架，实现了多种异构数据源之间高效的数据迁移

不同的数据源头被抽象成不同的Reader插件，不同的数据目标被抽象成不同的Writer插件。理论上，FlinkX框架可以支持任意数据源类型的数据同步工作。作为一套生态系统，每接入一套新数据源该新加入的数据源即可实现和现有的数据源互通。

FlinkX是一个基于Flink的批流统一的数据同步工具，既可以采集静态的数据，比如MySQL，HDFS等，也可以采集实时变化的数据，比如MySQL binlog，Kafka等。

![img](https://gitee.com/dtstack_dev_0/flinkx/raw/1.10_release/docs/images/template.png)

在底层实现上，FlinkX依赖Flink，数据同步任务会被翻译成StreamGraph在Flink上执行，基本原理如下图：

![img](https://gitee.com/dtstack_dev_0/flinkx/raw/1.10_release/docs/images/diagram.png)

## flinkx vs datax vs flinkcdc

[flinkx vs datax vs flinkcdc](https://www.icode9.com/content-4-1098731.html)

简单来说：

1. flinkx 和 flink cdc 是基于 flink 的数据同步工具，批流一体。只是 flinkx 是第三方工具，重，支持数据源多，而 flink-cdc 是 flink 官方的一组连接器，轻，使用方便，但是支持的数据源也少（mysql/kafka/postgres）
2. [datax](https://github.com/alibaba/DataX) 是离线数据同步工具，只有离线数据。flinkx 有点对标这个的意思，而且 flinkx 使用 yarn 做任务调度，和 hadoop 生态无缝结合。

> DataX 是阿里云 [DataWorks数据集成](https://www.aliyun.com/product/bigdata/ide) 的开源版本，在阿里巴巴集团内被广泛使用的离线数据同步工具/平台。DataX 实现了包括 MySQL、Oracle、OceanBase、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、Hologres、DRDS 等各种异构数据源之间高效的数据同步功能。

## 运行

[运行任务](https://gitee.com/dtstack_dev_0/flinkx/blob/1.10_release/docs/quickstart.md#%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1) 提供 local，standalone，yarn session， yarn perjob 等模式运行。运行的时候有这些核心[参数概念](https://gitee.com/dtstack_dev_0/flinkx/blob/1.10_release/docs/quickstart.md#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)：

1. mode：运行模式
2. job：配置的同步任务 json（reader，writer，setting）
3. pluginRoot：即各种打包后的同步插件。如果要支持新的 reader/writer，就是开发插件，打包后放在这个目录下。
4. pluginLoadMode: 指定上述 plugin 加载的方式，一种（`classpath`是各个 yarn node 都要包含这些插件，`shipfile` 则只需要 flinkx client 需要，yarn node 会从 flinkx client ship 过去）

## 同步任务

[同步任务配置](https://gitee.com/dtstack_dev_0/flinkx/blob/1.10_release/docs/generalconfig.md#%E6%8F%92%E4%BB%B6%E9%80%9A%E7%94%A8%E9%85%8D%E7%BD%AE)以 json 提供，一般需要指定 reader, writer, setting 等信息。

### 断点续传

[详解 flinkx 断点续传功能](https://juejin.cn/post/6959130447376826376)

setting 中有一项 `restore` 是配置断点续传，断点续传主要通过 checkpoint 机制实现，一般来讲，满足下述条件就可以支持实现断点续传功能：

1. reader 可以支持数据过滤（数据源需要有一个升序字段，支持 checkpoint 和筛选数据）
2. writer 支持 transaction，可以一次将 checkpoint 的 snapshot 的数据写入物理层。

# flinkx connector 自定义开发

flinkx 基于 flink，所以它也是通过一堆 connector 来实现扩展。

[flink customerized connector](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/dev/table/sourcessinks/)

