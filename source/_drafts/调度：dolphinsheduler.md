---
title: 调度：dolphinsheduler
toc: true
date: 2021-10-14 10:49:41
tags:
  - big data
---

## dolphinscheduler

[dolphinscheduler](https://dolphinscheduler.apache.org/zh-cn/index.html) ：分布式易扩展的可视化工作流任务调度平台

> 差不多是 yarn + airflow + 监控 + ui，其实就是数据平台 / dataworks 的主体了。这样一个平台可以：
>
> 1. 定义工作流：dag
> 2. 定义 task：dag 中的每个节点，并且将常见的任务节点都已经抽象了，比如 sql 查询节点、udf 定义节点、存储程序节点、spark 节点、flink 节点、datax 节点等等
> 3. 资源调度管理：可以设置资源队列，即 yarn 中的队列，上述 dag、任务可以指定执行队列
> 4. 租户隔离
> 5. 各种任务、工作流监控、用户管理等监控

Apache DolphinScheduler是一个分布式去中心化，易扩展的可视化DAG工作流任务调度平台。致力于解决数据处理流程中错综复杂的依赖关系，使调度系统在数据处理流程中开箱即用

![img](https://dolphinscheduler.apache.org/img/archdiagram_zh.svg)
