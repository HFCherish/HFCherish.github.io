---
title: starrocks
toc: true
date: 2021-10-19 14:54:49
tags:
  - big data
---

[starrocks](https://docs.starrocks.com/zh-cn/main/quick_start/Deploy#%E4%BD%BF%E7%94%A8mysql%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AE%BF%E9%97%AEstarrocks) 以前叫 dorisdb，现在改名字了。

定位：极速全场景 MPP 数据库

# 概念

FE: FrontEnd. 是StarRocks的前端节点，负责管理元数据，管理客户端连接，进行查询规划，查询调度等工作。

BE: BackEnd. 是StarRocks的后端节点，负责数据存储，计算执行，以及compaction，副本管理等工作。

Tablet：StarRocks中表的逻辑分片，也是StarRocks中**副本管理的基本单位**，每个表根据分区和分桶机制被划分成多个Tablet存储在不同BE节点上。tablet 应该也是查询 scan 优化的基本单位

# 架构

[starrocks 系统架构](https://docs.starrocks.com/zh-cn/main/quick_start/Architecture)

它使用 mysql client 来访问集群，做查询。

1. 用户通过 mysql client 执行 sql 语句
2. 这个 sql 语句会被 sql layer 解析、改写，生成逻辑执行计划
3. Planner 将逻辑执行计划转化为物理执行计划，并分发给 executor 来执行
4. executor 会读本地的列式存储引擎，来获取数据，并通过索引和谓词下沉快速过滤数据
5. 另外：
   1. FE 的 catalog 管理元数据，通过 mysql client 执行 ddl 语句时，这些元数据最终也会通过 catalog 服务持久化到物理存储上。
   2. bdgje 应该是用来管理 BE 的上下线、心跳、tablet 副本数量等。

![architecture](https://docs.starrocks.com/static/6301aa9a95fca6f9bd037b15cd57d48b/ef3e1/2.1-1.png)

## 数据流和控制流 

### 查询

[查询](https://docs.starrocks.com/zh-cn/main/quick_start/Data_flow_and_control_flow#%E6%9F%A5%E8%AF%A2)

其中 执行计划在BE上的实际执行过程比较复杂, 采用向量化执行方式，比如一个算子产生4096个结果，输出到下一个算子参与计算，而非batch方式或者one-tuple-at-a-time。？？？？？

此外，FE 会首先选择一个 BE 作为 coordinator，且这个 coordinator 最后会负责把查询的结果汇总后给 FE 返回

![query_plan](https://docs.starrocks.com/static/07d9c3ce41e41433d97546d055f4683c/c1b63/2.4.1-1.png)

### 数据导入

[数据导入](https://docs.starrocks.com/zh-cn/main/quick_start/Data_flow_and_control_flow#%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%85%A5)

导入请求中有 label，作为导入任务的唯一标识，label 用于避免数据重复导入。

一次数据导入会启动一个全局 transaction

BE收到请求后, 向FE master节点上报, 执行loadTxnBegin, 创建全局事务。 因为导入过程中, 需要同时更新base表和物化索引的多个bucket, 为了保证数据导入的一致性, 用事务控制本次导入的原子性.

当BE coordinator节点完成此次数据导入, 向FE master节点执行loadTxnCommit, 提交全局事务, 发送本次数据导入的 执行情况, FE master确认所有涉及的tablet的多数副本都成功完成, 则发布本次数据导入使数据对外可见, 否则, **导入失败, 数据不可见, 后台负责清理掉不一致的数据**（这个应该指的是 BE 会自动回滚）. 这个应该是通过 catalog 实现的，每次查询都是查的 catalog，而 catalog 只有任务完成后才会更新，catalog 里记录了 table, partition, index, tablet 等信息。没有这些信息，查询的时候就查不到这些表或分区，但是数据其实可能已经部分存储到硬盘了。这就涉及到下边的[元数据更新](#metadata-update)

数据导入会作用到所有关联的 MVs 表中。在 Base 表及所有的 MVs 表均完成后，导入才算完成，数据才能被看到。StarRocks保证**Base 表和 MVs 表之间的数据是一致的**。查询 Base 表和查询 MVs 表不会存在数据差异。

![load](https://docs.starrocks.com/static/f6aa09a24c0d34f58c33059267157053/c1b63/2.4.2-1.png)

现有的[数据导入方式](https://docs.starrocks.com/zh-cn/main/loading/Loading_intro)：

其中 spark load 方式适合大体量 hive 数据导入，以及需要对数据做复杂清洗的场景（broker load 方式也支持，比较简单）。这种方式通过外部资源（非 starrocks 集群资源）Spark对数据进行预处理生成中间文件，StarRocks读取中间文件导入，所以 starrocks 接入的其实时 spark 处理后的结果

![数据导入概览](https://docs.starrocks.com/static/be49131194eb5ebef2f4f4eba3b6c12a/2e694/4.1.1.png)

#### broker 导入

1. 部署 broker，这个 broker 进程用于访问远端存储，从远端存储拉数据



### 元数据更新<a name='metadata-update' />

[元数据更新](https://docs.starrocks.com/zh-cn/main/quick_start/Data_flow_and_control_flow#%E6%9B%B4%E6%94%B9%E5%85%83%E6%95%B0%E6%8D%AE)

ddl 操作会先在 BE 执行，保证物理表修改完成后，才会修改 catalog、editlog（类似于 binlog 之类的东西），最后把 fe master 的 editlog 同步到 fe followers，并基于 editlog 回放 catalog。

- 用户使用MySQL client执行SQL的DDL命令, 向FE的master节点发起请求; 比如: 创建表.
- FE检查请求合法性, 然后向BE发起同步命令, 使操作在BE上生效; 比如: FE确定表的列类型是否合法, 计算tablet的副本的放置位置, 向BE发起请求, 创建tablet副本.
- BE执行成功, 则修改FE内存的**Catalog**. 比如: 将table, partition, index, tablet的副本信息保存在Catalog中.
- FE追加本次操作到**EditLog**并且持久化.
- FE通过复制协议将EditLog的新增操作项同步到FE的follower节点.
- FE的follower节点收到新追加的操作项后, 在自己的Catalog上按顺序播放, 使得自己状态追上FE master节点.

上述过程任何环节都可能失败：

1. BE 任务失败，这个时候有可能部分成功了，那应该是有回滚操作的，类似数据导入部分的后台清理？
2. catalog / editlog 修改失败。此时 BE 任务已经成功了，BE 的这部分修改需要回滚，BE 自动回滚？？？
3. catalog / editlog 同步到 FE 其他节点。

这里很有可能导致不一致啊，第3个场景，也可能基于 editlog 播放 catalog 失败。而且从设计方案讲有不一致：

1. FE master 是先修改 catalog，再修改 editlog
2. FE follower 是先同步 editlog，再基于 editlog 回放 catalog。

![meta_change](https://docs.starrocks.com/static/26675d8c6494afa4839154dfad2887df/c1b63/2.4.3-1.png)



# 表设计

## 数据模型

starrocks 采用列式存储（一列数据会经过分块编码压缩等操作, 然后持久化于非易失设备），一张表的列可以分为维度列(也成为key列)和指标列(value列), 维度列用于分组和排序, 指标列可通过聚合函数SUM, COUNT, MIN, MAX, REPLACE, HLL_UNION, BITMAP_UNION等累加起来.

[create_table](https://docs.starrocks.com/zh-cn/main/sql-reference/sql-statements/data-definition/CREATE%20TABLE#create-table)

维度列也是**排序列**。starrocks 存储数据时，会把数据按指定列（有顺序的，类似 mysql 复合索引中的字段顺序）排序，有如下限制：

* 排序列的定义必须出现在建表语句中其他列的定义之前
* 排序列的顺序是由create table语句中的列顺序决定的。DUPLICATE/UNIQUE/AGGREGATE KEY中顺序需要和create table语句保持一致

starrocks 不仅仅是一个 olap 数据库，它定位的是全场景 MPP 数据库，所以它可以处理多种数据场景：

* **明细模型**：对应明细表。存储不经常变化的明细数据，以追加写为主，存储数据历史（数据主键有重复）, e.g. 原始日志、原始操作记录等。

* **聚合模型**：对应聚合表，就是我们常说的大宽表啦。这种表没有历史
* **更新模型**：对应更新表。存储经常变化的明细数据，通过追加的方式存储增量，相同主键的数据有多个版本，取版本最新的为最新状态。

> ```plaintext
> 数据按照指定的key列进行排序，且根据不同的key_type具有不同特性。
> key_type支持以下类型：
> AGGREGATE KEY:key列相同的记录，value列按照指定的聚合类型进行聚合，
> 适合报表、多维分析等业务场景。
> UNIQUE KEY:key列相同的记录，value列按导入顺序进行覆盖，
> 适合按key列进行增删改查的点查询业务。
> DUPLICATE KEY:key列相同的记录，同时存在于StarRocks中，
> 适合存储明细数据或者数据无聚合特性的业务场景。
> 默认为DUPLICATE KEY，key列为列定义中前36个字节, 如果前36个字节的列数小于3，将使用前三列。
> 
> 注意：
> 除AGGREGATE KEY外，其他key_type在建表时，value列不需要指定聚合类型。
> ```

```sql
-- 通过不同的 key_type 区分不同的数据模型（duplicate_key, aggregate_key, unique_key)
CREATE TABLE site_access_duplicate
(
site_id INT DEFAULT '10',
city_code SMALLINT,
user_name VARCHAR(32) DEFAULT '',
pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(site_id, city_code)	-- 这个采用明细模型，这个主键被作为 sort key
DISTRIBUTED BY HASH(site_id) BUCKETS 10;

CREATE TABLE site_access_aggregate
(
site_id INT DEFAULT '10',
city_code SMALLINT,
user_name VARCHAR(32) DEFAULT '',
pv BIGINT SUM DEFAULT '0'	-- 聚合列，SUM 指定了 aggregate type
)
AGGREGATE KEY(site_id, city_code)	-- 这个采用聚合模型，这些维度列被作为 sort key
DISTRIBUTED BY HASH(site_id) BUCKETS 10;

CREATE TABLE site_access_unique
(
site_id INT DEFAULT '10',
city_code SMALLINT,
user_name VARCHAR(32) DEFAULT '',
pv BIGINT DEFAULT '0'
)
UNIQUE KEY(site_id, city_code)	-- 这个采用更新模型，这些主键被作为 sort key
DISTRIBUTED BY HASH(site_id) BUCKETS 10;
```

### duplicate model

[duplicate model with example](https://doris.apache.org/master/en/getting-started/data-model-rollup.html#duplicate-model)

> In some multidimensional analysis scenarios, data has neither primary keys nor aggregation requirements. **Data is stored entirely in accordance with the data in the imported file**, without any aggregation.
>
> The **DUPLICATE KEY** specified in the table building statement is **only used to specify which columns the underlying data is sorted according to**. (The more appropriate name should be "Sorted Column", where the name "DUPLICATE KEY" is used to specify the data model used. On the choice of DUPLICATE KEY, we **recommend that the first 2-4 columns** be selected appropriately.
>
> This data model is suitable for storing raw data without aggregation requirements and primary key uniqueness constraints.

## 分区和分桶

StarRocks支持支持单分区和复合分区两种建表方式。和 [hive bucket & hive partition](https://hfcherish.github.io/2019/01/03/hive-introduction/) 类似，都是提高数据读写性能的手段，特别是缩减 scan 的数据量。

在复合分区中：

- 第一级称为Partition，即分区。用户可以指定某一维度列作为分区列（当前只支持整型和时间类型的列），并指定每个分区的取值范围。虽然分区列只能是整型/时间类型，但分区名是 string，e.g. p20200101。一张表的分区名可以有多种取值方式（见下边的 range partition）
- 第二级称为Distribution，即分桶。用户可以指定某几个维度列（或不指定，即所有KEY列）以及桶数对数据进行HASH分布。不同分区的分桶数目可以不同。

如何分区：

- 一般从数据的管理角度来选择分区键，选用**时间**或者区域作为分区键。（但是上边也说了，只支持整型和时间，所以区域必须是个整型区域码才行）
- 使用动态分区可以定期自动创建分区，比如每天创建出新的分区。

如何分桶：

- 选择**高基数的列**来作为分桶键（如果有唯一ID就用这个列来作为 分桶键 即可），这样保证数据在各个bucket中尽可能均衡，如果碰到数据倾斜严重的，数据可以使用多列作为分桶键（但一般不要太多）。
- 分桶的数量影响查询的并行度，最佳实践是计算一下数据存储量，将每个tablet设置成 100MB ~ 1GB 之间。（类似于以前的 partition 大小吧，128M）
- 在机器比较少的情况下，如果想充分利用机器资源可以考虑使用：bucket_count =  `BE数量 * cpu core / 2`（？？？？？）。例如有100GB数据的一张表，有4台BE，每台64C，只有一个分区，那么可以采用 bucket数量 `4 * 64 /2 = 144`，这样每个tablet的数据也在694MB，同时也能充分利用CPU资源。

```sql

mysql >
-- 建立单分区表
CREATE TABLE table1
(
    siteid INT DEFAULT '10',
    citycode SMALLINT,
    username VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, citycode, username) -- 指定维度列
DISTRIBUTED BY HASH(siteid) BUCKETS 10	-- 指定分桶规则
PROPERTIES("replication_num" = "1");	-- 指定副本数目

-- 建立复合分区表
CREATE TABLE table2
(
event_day DATE,
siteid INT DEFAULT '10',
citycode SMALLINT,
username VARCHAR(32) DEFAULT '',
pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, siteid, citycode, username)	-- 指定维度列
PARTITION BY RANGE(event_day)	-- 指定 range 分区，这里分区列是日期类型，分区名 p1,p2,p3 是 string
(
PARTITION p1 VALUES LESS THAN ('2017-06-30'),
PARTITION p2 VALUES LESS THAN ('2017-07-31'),
PARTITION p3 VALUES LESS THAN ('2017-08-31')
)
DISTRIBUTED BY HASH(siteid) BUCKETS 10 		-- 指定分桶规则
PROPERTIES("replication_num" = "1");

-- 新增 range 分区，并指定该分区的分桶数（不指定，默认和原分区相同）
ALTER TABLE table2
ADD PARTITION p4 VALUES LESS THAN ("2020-04-31")
DISTRIBUTED BY HASH(site_id) BUCKETS 20;
```

### Range partition

starrocks 的分区只有 range partition 这一种做法，具体参照 [create table](https://docs.starrocks.com/zh-cn/main/sql-reference/sql-statements/data-definition/CREATE%20TABLE#create-table) 中的定义

分区其实有三个概念：

* 分区列：基于哪个列做的分区（只能是整型/日期类型列）
* 分区名：给一个分区取的名字，就是常说的分区，比如 20200101-20200131 的分区叫 p202001，是个 string，一张表的分区名可以有多种取名类型
* 分区值：每个分区名，可以对应多个分区列的值

```sql
CREATE TABLE site_access (
    datekey DATE,
    site_id INT,
    city_code SMALLINT,
    user_name VARCHAR(32),
    pv BIGINT DEFAULT '0'
)
ENGINE=olap
DUPLICATE KEY(datekey, site_id, city_code, user_name)
PARTITION BY RANGE (datekey) (
    START ("2019-01-01") END ("2021-01-01") EVERY (INTERVAL 1 YEAR),
    START ("2021-01-01") END ("2021-05-01") EVERY (INTERVAL 1 MONTH),
    START ("2021-05-01") END ("2021-05-04") EVERY (INTERVAL 1 DAY)
)
DISTRIBUTED BY HASH(site_id) BUCKETS 10
PROPERTIES (
    "replication_num" = "1"
);

-- 分区列： datekey
-- 产生如下分区：
-- PARTITION p2019 VALUES [('2019-01-01'), ('2020-01-01')),
-- PARTITION p2020 VALUES [('2020-01-01'), ('2021-01-01')),
-- PARTITION p202101 VALUES [('2021-01-01'), ('2021-02-01')),
-- PARTITION p202102 VALUES [('2021-02-01'), ('2021-03-01')),
-- PARTITION p202103 VALUES [('2021-03-01'), ('2021-04-01')),
-- PARTITION p202104 VALUES [('2021-04-01'), ('2021-05-01')),
-- PARTITION p20210501 VALUES [('2021-05-01'), ('2021-05-02')),
-- PARTITION p20210502 VALUES [('2021-05-02'), ('2021-05-03')),
-- PARTITION p20210503 VALUES [('2021-05-03'), ('2021-05-04'))
```



### 动态分区

starrocks 的动态分区和 hive 的有差别，它似乎主要是为了管理分区的 ttl 的。

```sql
CREATE TABLE site_access(
event_day DATE,
site_id INT DEFAULT '10',
city_code VARCHAR(100),
user_name VARCHAR(32) DEFAULT '',
pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
PARTITION p20200321 VALUES LESS THAN ("2020-03-22"),
PARTITION p20200322 VALUES LESS THAN ("2020-03-23"),
PARTITION p20200323 VALUES LESS THAN ("2020-03-24"),
PARTITION p20200324 VALUES LESS THAN ("2020-03-25")
)
DISTRIBUTED BY HASH(event_day, site_id) BUCKETS 32
PROPERTIES(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",	-- 按天建分区
    "dynamic_partition.start" = "-3",		-- 分区起始值为今天往前 3 天
    "dynamic_partition.end" = "3",			-- 分区结束值为今天往后 3 天
    "dynamic_partition.prefix" = "p",		-- 分区名的前缀
    "dynamic_partition.buckets" = "32"
);
```

# 性能

[性能优化](https://docs.starrocks.com/zh-cn/main/administration/Profiling)

## shortkey 索引

[prefix index](https://doris.apache.org/master/en/getting-started/data-model-rollup.html#rollup)

所有的 key 列就是 order key ----> 这就是建议将维度列都做排序，维度列的顺序影响稀疏索引的建立。

1. 先按 key 去排序数据，存储
2. 然后基于 key 顺序建立稀疏索引

> Unlike traditional database design, Doris does not support indexing on any column.
>
> In Aggregate, Uniq and Duplicate three data models. The underlying data storage is sorted and stored according to the columns specified in AGGREGATE KEY, UNIQ KEY and DUPLICATE KEY in their respective table-building statements.
>
> The prefix index, which is based on sorting, **implements an index method** to query data quickly according to a given prefix column.

StarRocks**对数据进行有序存储, 在数据有序的基础上为其建立稀疏索引**,索引粒度为block(1024行)。

- 稀疏索引选取schema中固定长度的前缀作为索引内容, 目前StarRocks选取36个字节的前缀作为索引。
- 建表时建议将查询中常见的过滤字段放在schema的前面, 区分度越大，频次越高的查询字段越往前放。
- 这其中有一个特殊的地方,就是varchar类型的字段,varchar类型字段只能作为稀疏索引的最后一个字段，索引会在varchar处截断, 因此varchar如果出现在前面，可能索引的长度不足36个字节。

StarRocks在Sort Key的基础上引入稀疏的shortkey index，Sort Index的内容会比数据量少1024倍，因此会全量缓存在内存中，实际查找的过程中可以有效加速查询。

但是 sort key 的取值、列可能很多，这个时候为将所有 sort key 加入到 shortkey index 就会占用大量内存，所有有限制：

- shortkey 的列只能是排序键的前缀;
- shortkey 列数不超过3;
- 字节数不超过36字节;
- 不包含FLOAT/DOUBLE类型的列;
- VARCHAR类型列只能出现一次, 并且是末尾位置;
- 当shortkey index的末尾列为CHAR或者VARCHAR类型时, shortkey的长度会超过36字节;
- 当用户在建表语句中指定PROPERTIES {short_key = "integer"}时, 可突破上述限制;

## Materialized View

[物化视图](https://docs.starrocks.com/zh-cn/main/table_design/Materialized_view)，和 hive 的 materialized view 类似，目前只能创建单表的，查询时，用户不需要指定 MV 表，starrocks 会根据 sql 智能选择最佳的 MV 表。

MV表的选择规则如下：

1. 选择包含所有查询列的MV表
2. 按照过滤和排序的Column筛选最符合的MV表
3. 按照Join的Column筛选最符合的MV表
4. 行数最小的MV表
5. 列数最小的MV表

什么时候使用 MV：

- Analyze requirements to cover both detailed data query and fixed-dimensional query.
- The query only involves a small part of the columns or rows in the table.
- The query contains some time-consuming processing operations, such as long-time aggregation operations.
- The query needs to match different prefix indexes

## Rollup

ROLLUP in multidimensional analysis means "scroll up", which means that data is aggregated further at a specified granularity

Rollup本质上可以理解为原始表(base table)的一个物化索引。建立rollup时可只选取base table中的部分列作为schema，schema中的字段顺序也可与base table不同。**物化**是因为其数据在物理上独立存储，而**索引**的意思是，Rollup可以调整列顺序以增加前缀索引的命中率，也可以减少key列以增加数据的聚合度。

[materialized view vs Rollup](https://doris.apache.org/master/en/administrator-guide/materialized_view.html#materialized-view-vs-rollup)

和 materialized view 的区别是，**rollup 针对明细模型不能做预聚合**。**MV 相当于 rollup 的 superset**，包含了所有 rollup 的能力，但能支持更多的聚合 function，也能对明细模型做预聚合。所有用 `alter table add rollup` 能做的，都能通过 `create materialized view` 来做到。

## 查询并行度parallel_fragment_exec_instance_num

[parallel_fragment_exec_instance_num](https://docs.starrocks.com/zh-cn/main/quick_start/Test_faq#%E5%A6%82%E4%BD%95%E5%90%88%E7%90%86%E5%9C%B0%E8%AE%BE%E7%BD%AE%E5%B9%B6%E8%A1%8C%E5%BA%A6) 有点类似于 sqlServer 的 MaxDOP：

```
set global parallel_fragment_exec_instance_num = 16
```

* 设置为 1，好像就是变串行，高 QPS 的场景下，可以减少不同查询之间的资源竞争，反而可以提升 QPS

# Q&A

- [ ] 数据导入和元数据更新的过程中，都有很多一致性问题，当任何一步失败后，是系统有自动的回滚操作，还是需要手动回滚？
- [ ] 分桶数量的推荐计算方式中，为啥每个 bucket 有俩 core，是不是一个 bucket 的数据会被查询引擎进一步拆分成类似于 spark partition 的东西？

有点奇怪，这里讲到一个 `parallel_fragment_exec_instance_num`，指[查询的并行度](https://docs.starrocks.com/zh-cn/main/quick_start/Test_faq#%E5%A6%82%E4%BD%95%E5%90%88%E7%90%86%E5%9C%B0%E8%AE%BE%E7%BD%AE%E5%B9%B6%E8%A1%8C%E5%BA%A6)，这里的说法，就是 tablet 是执行的最小单位了……？？？

> parallel_fragment_exec_instance_num 受到每个BE的tablet数量的影响，比如 一张表的bucket数量是 32, 有3个分区，分布在4个BE上，那么每个BE的tablet数量是 32 * 3 / 4 = 24, 那么单机的并行度无法超过24，即使你set parallel_fragment_exec_instance_num = 32 ，但是实际执行的时候并行度还是会变成24。

- [ ] starrocks 还挺重视索引的，所以我们建立数据导入任务时，是否需要针对每张表考虑索引？
- [x] 明细表、聚合表、更新表，在建表语句上有啥区别吗？或者有啥具体的技术支持吗？

A: 有区别，就是不同的 key_type

- [ ] 动态分区指定了分区上下界，但是同时指定 range partition？？？
- [x] spark load 难道就是直接把 spark etl 后的结果导入过来，那不还是 hive 吗？所谓的中间结果是指啥呢？

A: 并不是，它的预处理和其他是一样的，并不是 spark etl，只是通过 spark etl 来做数据导入，是它自动生成了一个 spark 任务，提交到 spark 集群取执行，并把中间结果直接拿过来。这个 spark 任务是它自动生成的，所以它的中间结果应该就是 etl 的输出，只是这个输出可以被直接使用。

- [ ] rollup 是怎么查询的？
