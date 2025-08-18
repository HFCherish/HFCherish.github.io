---
title: sql window functions
toc: true
date: 2021-07-02 12:06:41
tags:
    - database
---

[sql 窗口函数](https://zhuanlan.zhihu.com/p/92654574)

[bigquery 分析函数概念](https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts?hl=zh-cn)

# 基本语法

分析函数针对一组行计算值，并为每行返回一个结果。这与聚合函数不同；聚合函数会为一组行返回一个结果。

分析函数包含一个 `OVER` 子句，该子句定义了涵盖所要计算行的行窗口。对于每一行，系统会使用选定的行窗口作为输入来计算分析函数结果，并可能进行聚合。

借助分析函数，您可以计算移动平均值、对各项进行排名、计算累计总和，以及执行其他分析。

```sql
<窗口函数> over (partition by <用于分组的列名>
                order by <用于排序的列名>)
```

<窗口函数>的位置，可以放以下两种函数：

1） 专用窗口函数，包括后面要讲到的rank, dense_rank, row_number等专用窗口函数。

2） 聚合函数，如sum. avg, count, max, min等

Partition by 可选

分析函数可以在查询的以下两处位置显示为标量表达式操作数：

- `SELECT` 列表。如果分析函数显示在 `SELECT` 列表中，则其参数列表和 `OVER` 子句不能引用同一 SELECT 列表中引入的别名。
- `ORDER BY` 子句。如果分析函数显示在查询的 `ORDER BY` 子句中，则其参数列表可以引用 `SELECT` 列表别名。

分析函数的参数列表或 `OVER` 子句不能引用其他分析函数，即使通过别名间接引用也不行。

分析函数在聚合后进行计算。例如，`GROUP BY` 子句和非分析聚合函数先进行计算。 由于聚合函数是在分析函数之前进行计算，因此聚合函数可用作分析函数的输入操作数。

窗口函数是对where或者group by子句处理后的结果进行操作，所以**窗口函数原则上只能写在select子句中**。
