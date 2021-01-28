---
title: spark
toc: true
tags:
  - hadoop
  - distributed computing
  - big data
date: 2019-01-10 10:17:01
---

# Concept

[spark](https://spark.apache.org/docs/latest/) is a fast and general-purpose cluster computing system like Hadoop Map-reduce.  It runs on the clusters.

## Spark Ecosystem

**[The components of Apache Spark Ecosystem](http://data-flair.training/blogs/apache-spark-ecosystem-components/)**

![ecosystem](https://d2h0cx97tjks2p.cloudfront.net/blogs/wp-content/uploads/sites/2/2017/07/apache-spark-ecosystem-components.jpg)

* spark core: **cluster computing system**. Provide API to  write computing functions.
* [Spark SQL](https://spark.apache.org/docs/2.4.0/sql-programming-guide.html). SQL for data processing, like hive?
* [MLlib](https://spark.apache.org/docs/2.4.0/ml-guide.html) for machine learning.
* [GraphX](https://spark.apache.org/docs/2.4.0/graphx-programming-guide.html) for graph processing
* [Spark Streaming](https://spark.apache.org/docs/2.4.0/streaming-programming-guide.html).

### Spark Core

Spark Core is the fundamental unit of the whole Spark project. Its key features are:

- It is in charge of essential I/O functionalities.
- Provide API to defines and manipulate the RDDs. Significant in programming and observing the role of the **[Spark cluster](http://data-flair.training/blogs/install-apache-spark-multi-node-cluster/)**.
- Task dispatching, scheduling
- Fault recovery.
- It overcomes the snag of**[ MapReduce](http://data-flair.training/blogs/hadoop-mapreduce-introduction-tutorial-comprehensive-guide/)** by using in-memory computation.

Spark makes use of Special data structure known as **[RDD (Resilient Distributed Dataset)](http://data-flair.training/blogs/rdd-in-apache-spark/)**. Spark Core is distributed execution engine with all the functionality attached on its top. For example, MLlib, **[SparkSQL](http://data-flair.training/blogs/spark-sql-tutorial/)**, GraphX, **[Spark Streaming](http://data-flair.training/blogs/apache-spark-streaming-comprehensive-guide/)**. Thus, allows diverse workload on single platform. All the basic functionality of Apache Spark Like **[in-memory computation](http://data-flair.training/blogs/apache-spark-in-memory-computing/), [fault tolerance](http://data-flair.training/blogs/apache-spark-streaming-fault-tolerance/)**, memory management, monitoring, task scheduling is provided by Spark Core.
Apart from this Spark also provides the basic connectivity with the data sources. For example, **[HBase](http://data-flair.training/blogs/category/hbase/)**, Amazon S3, **[HDFS ](http://data-flair.training/blogs/comprehensive-hdfs-guide-introduction-architecture-data-read-write-tutorial/)**etc.

#### Action, Job, Stage, Task

**Actions** are RDD’s operation. *reduce, collect, takeSample, take, first, saveAsTextfile, saveAsSequenceFile, countByKey, foreach* are common actions in Apache spark.

In a Spark application, when you invoke an action on RDD, a **job** is created. Jobs are the main function that has to be done and is submitted to Spark. The jobs are divided into **stages** depending on how they can be separately carried out (mainly on shuffle boundaries). Then, these stages are divided into **tasks**. Tasks are the smallest unit of work that has to be done the executor.

 **An example:**

[What is Spark Job ?](https://stackoverflow.com/questions/28973112/what-is-spark-job)

> let's say you need to do the following:
>
> 1. Load a file with people names and addresses into RDD1
> 2. Load a file with people names and phones into RDD2
> 3. Join RDD1 and RDD2 by name, to get RDD3
> 4. Map on RDD3 to get a nice HTML presentation card for each person as RDD4
> 5. Save RDD4 to file.
> 6. Map RDD1 to extract zipcodes from the addresses to get RDD5
> 7. Aggregate on RDD5 to get a count of how many people live on each zipcode as RDD6
> 8. Collect RDD6 and prints these stats to the stdout.
>
> So,
>
> 1. The ***driver program\*** is this entire piece of code, running all 8 steps.
> 2. Producing the entire HTML card set on step 5 is a ***job\*** (clear because we are using the *save* action, not a transformation). Same with the *collect* on step 8
> 3. Other steps will be organized into ***stages\***, with each job being the result of a sequence of stages. For simple things a job can have a single stage, but the need to repartition data (for instance, the join on step 3) or anything that breaks the locality of the data usually causes more stages to appear. You can think of stages as computations that produce intermediate results, which can in fact be persisted. For instance, we can persist RDD1 since we'll be using it more than once, avoiding recomputation.
> 4. All 3 above basically talk about how the *logic* of a given algorithm will be broken. In contrast, a ***task\*** is a particular *piece of data* that will go through a given stage, on a given executor.

## SparkContext

[SparkContext](https://data-flair.training/blogs/learn-apache-spark-sparkcontext/) is the entry point of Spark functionality. The most important step of any Spark driver application is to generate SparkContext. **It allows your Spark Application to access Spark Cluster** with the help of Resource Manager. 

If you want to create SparkContext, first **SparkConf** should be made. The SparkConf has a configuration parameter that our Spark driver application will pass to SparkContext. Some of these parameter defines properties of Spark driver application. While some are used by Spark to allocate resources on the cluster, like the number, memory size, and cores used by executor running on the worker nodes.
In short, **it guides how to access the Spark cluster**. After the creation of a SparkContext object, we can invoke functions such as **textFile, sequenceFile, parallelize** etc.
Once the SparkContext is created, it can be used to **[create RDDs](http://data-flair.training/blogs/how-to-create-rdds-in-apache-spark/)**, broadcast variable, and accumulator, ingress Spark service and run jobs. All these things can be carried out until SparkContext is stopped.

```python
from __future__ import print_function

import sys
from operator import add

from pyspark.sql import SparkSession


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: wordcount <file>", file=sys.stderr)
        sys.exit(-1)

    # the builder here defines the sparkConf, and then create a sparkSession with an underlying SparkContext `spark.sparkContext`
    spark = SparkSession\
        .builder\
        .appName("PythonWordCount")\
        .getOrCreate()

    # here by `spark.read.text('some.txt')`, we use SparkContext create an DataFrame
    lines = spark.read.text(sys.argv[1]).rdd.map(lambda r: r[0])
    counts = lines.flatMap(lambda x: x.split(' ')) \
                  .map(lambda x: (x, 1)) \
                  .reduceByKey(add)
    # this is the spark action `collect`
    output = counts.collect()
    for (word, count) in output:
        print("%s: %i" % (word, count))

    # this in fact stop the sparkContext
    spark.stop()
```



![10 Important Functions of SparkContext in Apache Spark](https://d2h0cx97tjks2p.cloudfront.net/blogs/wp-content/uploads/sites/2/2017/08/functions-of-sparkcontext-in-apache-spark.jpg)

# How does it run

Spark core contains the main api, driver engine, scheduler... to support the cluster computing. The real computing is completed on the cluster. Spark can connect to many cluster managers(spark's own standalone cluster manager, mesos, yarn) to complete the jobs. Typically, the process is like this:

1. The user submits a spark application using the `spark-submit` command.
2. Spark-submit launches the driver program on the same node in (client mode) or on the cluster (cluster mode) and invokes the main method specified by the user.
3. The driver program contacts the cluster manager to ask for resources to launch executor JVMs based on the configuration parameters supplied.
4. The cluster manager launches executor JVMs on worker nodes.
5. The driver process scans through the user application. Based on the RDD actions and transformations in the program, Spark creates an operator graph.
6. When an action (such as collect) is called, the graph is submitted to a DAG scheduler. The DAG scheduler divides the operator graph into stages.
7. A stage comprises tasks based on partitions of the input data. The driver sends work to executors in the form of tasks.
8. The executors process the task and the result sends back to the driver through the cluster manager.

![cluster mode overview](https://spark.apache.org/docs/latest/img/cluster-overview.png)

![Complete Picture of Apache Spark Job Execution Flow.](https://d2h0cx97tjks2p.cloudfront.net/blogs/wp-content/uploads/sites/2/2017/08/Internals-of-job-execution-in-spark.jpg)

# API

You can write spark function (eg. map function, reduce funciton) using Java/scala/python/R API. See [api docs](https://spark.apache.org/docs/latest/).

# Installation On Yarn

See [run spark on Yarn](https://spark.apache.org/docs/2.4.0/running-on-yarn.html), [Install, Configure, and Run Spark on Top of a Hadoop YARN Cluster](https://www.linode.com/docs/databases/hadoop/install-configure-run-spark-on-top-of-hadoop-yarn-cluster/)

1. [downloads page](https://spark.apache.org/downloads.html) download the spark
2. `tar -xvf spark-xxx.tgz`
3. configuration

* config in `/conf/spark-env.sh`

```sh
# config this to specify the installed HADOOP path
export SPARK_DIST_CLASSPATH=$HADOOP_HOME/bin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

* config in `/conf/spark-default.conf`. ([all configuration properties](https://spark.apache.org/docs/2.4.0/configuration.html#spark-properties))

```sh
# config the spark master
spark.master                     yarn
spark.driver.memory    512m
spark.yarn.am.memory    512m
spark.executor.memory          512m
```

## history server config

When the spark job is running, you can access the job log by `localhost:4040`. When the job is finished, by default, the log is not persisted which means you can't access it. To access the logs later, need to config the following: (see [spark Monitoring and Instrumentation](https://spark.apache.org/docs/latest/monitoring.html) and [using history server to replace the spark web ui](https://spark.apache.org/docs/2.4.0/running-on-yarn.html#using-the-spark-history-server-to-replace-the-spark-web-ui))

```sh
# in /conf/spark-default.conf
# config history server
spark.ui.filters		 org.apache.spark.deploy.yarn.YarnProxyRedirectFilter

# tell spark use history server url as the trackint url
spark.yarn.historyServer.allowTracking  true

# enable log persistence
spark.eventLog.enabled           true

# log write dir. Here use the hdfs dir and you must create the dir in hdfs first
spark.eventLog.dir               hdfs://localhost:9000/spark-logs

# log read dir. Sometimes logs are transfered.
spark.history.fs.logDirectory     hdfs://localhost:9000/spark-logs
```

## example execution

* Start histroy server: `sbin/start-history-server.sh`
* execute spark job:

```sh
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode client \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    examples/jars/spark-examples*.jar \
    10
```

Then you can:

* Check the job/application info in yarn: `http://localhost:8088/cluster/apps`

* Check the job/application using Spark history server: `http://localhost:18080/`

# Glossary

[glossary](https://spark.apache.org/docs/latest/cluster-overview.html#glossary)

NoteThe following table summarizes terms you’ll see used to refer to cluster concepts:

| Term            | Meaning                                                      |
| :-------------- | :----------------------------------------------------------- |
| Application     | User program built on Spark. Consists of a *driver program* and *executors* on the cluster. |
| Application jar | A jar containing the user's Spark application. In some cases users will want to create an "uber jar" containing their application along with its dependencies**. The user's jar should never include Hadoop or Spark libraries, however, these will be added at runtime.** |
| Driver program  | The process running the main() function of the application and creating the SparkContext |
| Cluster manager | An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN) |
| Deploy mode     | Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster. |
| Worker node     | Any node that can run application code in the cluster        |
| Executor        | A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors. |
| Task            | A unit of work that will be sent to one executor             |
| Job             | A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g. `save`, `collect`); you'll see this term used in the driver's logs. |
| Stage           | Each job gets divided into smaller sets of tasks called *stages* that depend on each other (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs. |

# Deploy mode

## yarn-client vs yarn-cluster

[yarn-client vs yarn-cluster 深度剖析](https://www.cnblogs.com/ittangtang/p/7967386.html)

[stackoverflow](https://stackoverflow.com/questions/41124428/spark-yarn-cluster-vs-client-how-to-choose-which-one-to-use)

# spark-shell vs spark-submit

Spark shell is only intended to be use for testing and perhaps development of small applications - is only an interactive shell and should not be use to run production spark applications. For production application deployment you should use spark-submit. The last one will also allow you to run applications in yarn-cluster mode

# Spark DataFrame

## auto increment id

[two ways for auto increment id](https://blog.csdn.net/k_wzzc/article/details/84996172)

### Row_number

```scala
    /**
      * 设置窗口函数的分区以及排序，因为是全局排序而不是分组排序，所有分区依据为空
      * 排序规则没有特殊要求也可以随意填写
      */
    val spec = Window.partitionBy().orderBy($"lon")

    val df1 = dataframe.withColumn("id", row_number().over(spec))

    df1.show()
```

### rdd.zipWithIndex

```scala
   // 在原Schema信息的基础上添加一列 “id”信息
    val schema: StructType = dataframe.schema.add(StructField("id", LongType))

    // DataFrame转RDD 然后调用 zipWithIndex
    val dfRDD: RDD[(Row, Long)] = dataframe.rdd.zipWithIndex()

    val rowRDD: RDD[Row] = dfRDD.map(tp => Row.merge(tp._1, Row(tp._2)))

    // 将添加了索引的RDD 转化为DataFrame
    val df2 = spark.createDataFrame(rowRDD, schema)

    df2.show()
```

## Add constant column

[add constant column](https://stackoverflow.com/questions/32788322/how-to-add-a-constant-column-in-a-spark-dataframe)

```scala
import org.apache.spark.sql.functions.typedLit

df.withColumn("some_array", typedLit(Seq(1, 2, 3)))
df.withColumn("some_struct", typedLit(("foo", 1, 0.3)))
df.withColumn("some_map", typedLit(Map("key1" -> 1, "key2" -> 2)))

from pyspark.sql.functions import lit
df.withColumn('new_column', lit(10))
```

# Hive Hints

[hive hints](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-hints.html#partitioning-hints)

# Optimization

[performance tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

## cache

```python
spark.catalog.cacheTable("tableName")
spark.catalog.uncacheTable("tableName")

dataFrame.cache()
```



## broadcast join

[spark 执行 map-join 优化](https://zhuanlan.zhihu.com/p/58765338)

[spark broadcast join](https://www.jianshu.com/p/2c7689294a73)

几种方式：

### 1. spark 自动识别小表 broadcast

`spark.sql.statistics.fallBackToHdfs=True`, 这样它会直接分析文件的大小，而不是 metastore 数据

### 2. 使用 hint

[hive hints](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-hints.html#partitioning-hints)

```sql
select /*+ BROADCAST (b) */ * from a where id not in (select id from b)
```

### 3. 使用 dataframe api

```python
from pyspark.sql.functions import broadcast
broadcast(spark.table("b")).join(spark.table("a"), "id").show()
```

## cache vs broadcast

[cache vs broadcast](https://stackoverflow.com/questions/38056774/spark-cache-vs-broadcast)

>RDDs are divided into *partitions*. These partitions themselves act as an immutable subset of the entire RDD. When Spark executes each stage of the graph, each partition gets sent to a worker which operates on the subset of the data. In turn, each worker can *cache* the data if the RDD needs to be re-iterated.
>
>Broadcast variables are used to send some immutable state *once* to each worker. You use them when you want a local copy of a variable.
>
>These two operations are quite different from each other, and each one represents a solution to a different problem.

## 小文件问题

[spark-sql 优化小文件过多](https://blog.csdn.net/lhxsir/article/details/87882128)

1. coalesce
2. repartition
3. Distribute by
4. adaptive execution

[Adaptive Execution 让 Spark SQL 更高效更智能](http://www.jasongj.com/spark/adaptive_execution/)

```
# 启用 Adaptive Execution ，从而启用自动设置 Shuffle Reducer 特性
spark.conf.set("spark.sql.adaptive.enabled", 'true')
# 设置每个 Reducer 读取的目标数据量，单位为字节。默认64M，一般改成集群块大小
spark.conf.set("spark.sql.adaptive.shuffle.targetPostShuffleInputSize", '134217728')
```

# Issues

## Null-aware predicate sub-queries cannot be used in nested conditions

`not in` 不能和 `or` 之类的 condition 一块用。现在好像还没有修复，参见：[SPARK-25154](https://github.com/apache/spark/pull/22141)