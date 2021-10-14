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
