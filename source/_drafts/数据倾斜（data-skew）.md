---
title: 数据倾斜（data skew）
toc: true
date: 2021-08-12 09:00:05
tags:
  - big data
---

[count(distinct) 优化](https://datavalley.github.io/2016/02/15/Hive%E4%B9%8BCOUNT-DISTINCT%E4%BC%98%E5%8C%96)

# 数据倾斜是什么

在了解数据倾斜以前，需要先了解下分布式并行运算，数据倾斜正是因为破坏了分布式并行运算的意图，使得大数据运算极慢，甚至运算不出来。

分布式计算引擎，以 spark 为例，一般将程序解析为 jobs -> stages -> tasks。task 是执行单元。这个里边有两个并行：

1. job 拆分的 stage 可能是可以并行的
2. stage 拆分的 task 是并行运算的。

## 倾斜的 stage

job 拆分的 stage 可以并行。比如如下 sql，a、b 表的计算 stage 是可以并行的。res 的表计算依赖 a、b 的计算。由于整个过程没有 action 的运算，这些步骤最后会形成一个大的 job。这个 job 会拆分成 a 的计算、b 的计算、res 的 join 等几个大的 stage。其中 a、b 的运算无依赖关系，是可以并行运行的。

此时，整个 job 的运行时长 = max(time(a), time(b)) + time(res)。找到运行时间特别长的 stage（skew 的 stage），想办法优化它。如何优化 stage，要继续往下看 stage 拆分的 task 运行情况。

```sql
-- target=temp.a
select * from x group by m
-- target=temp.b
select * from y group by n
...
-- target=temp.res
select * from a 
	left join b on a.id = b.id
	left join c on a.id = c.id
	...
```

## 倾斜的 task

stage 对数据做 partition，每个 task 处理一个 partition 的数据，task 并行运算。每个 task 由 executor 来执行，一个 node 上可以启动多个 executor。这三者一一对应

![image-20210812092334066](/Users/zhenzheng/code/hfcherish.github.io/source/images/image-20210812092334066.png)

如果 partition 的数据分布不均匀，即有的 partition 数据很多，此时就产生数据倾斜，对应的 task 压力很大，运行很慢。而

