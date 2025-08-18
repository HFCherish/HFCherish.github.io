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

## Core concepts???

- **RDDs (Resilient Distributed Datasets)**: RDDs are the fundamental data structure in Spark. They are immutable and can be split into multiple partitions that can be processed in parallel.
- **Transformations:** Transformations are operations that are applied to RDDs to create new RDDs. Transformations are lazy, which means that they are not executed until an action is called.
- **Actions:** Actions are operations that are applied to RDDs to produce a result. Actions trigger the execution of all of the transformations that have been applied to the RDD.
- **DAG (Directed Acyclic Graph)**: A DAG is a graph that represents the dependencies between transformations. Spark uses DAGs to schedule and execute transformations.
- **SparkContext:** The SparkContext is the entry point to the Spark API. It provides methods for creating RDDs, submitting jobs, and managing resources.
- **SparkSession:** The SparkSession is a newer API that provides a simpler and more concise way to use Spark. It is built on top of the SparkContext and provides a number of additional features, such as support for SQL and machine learning.

Spark Core is the foundation of Apache Spark and provides the essential functionality and APIs for distributed data processing. Here are some key Spark Core concepts explained with examples:

1. Resilient Distributed Datasets (RDDs): RDDs are the fundamental data structure in Spark Core. They represent an immutable distributed collection of objects that can be processed in parallel across a cluster. RDDs can be created from data in Hadoop Distributed File System (HDFS), local file systems, or by transforming existing RDDs. Here's an example of creating an RDD from a list of numbers:

```scala
val numbers = List(1, 2, 3, 4, 5)
val rdd = sparkContext.parallelize(numbers)
```

2. Transformations: Transformations are operations performed on RDDs to create new RDDs. Transformations are lazy and do not execute immediately but build a lineage graph to track the transformations applied. Examples of transformations include `map`, `filter`, `flatMap`, and `reduceByKey`. Here's an example of applying a transformation to square each element of an RDD:

```scala
val squaredRDD = rdd.map(num => num * num)
```

3. Actions: Actions are operations that trigger the execution of computations and return results or write data to external systems. Actions initiate the evaluation of RDDs and can trigger lineage-based optimizations. Examples of actions include `collect`, `count`, `reduce`, and `saveAsTextFile`. Here's an example of performing a sum operation on an RDD:

```scala
val sum = squaredRDD.reduce((a, b) => a + b)
```

4. Spark Context: The Spark Context (`sparkContext`) is the entry point and the central coordinator for Spark Core. It establishes a connection to the Spark cluster and manages the execution environment. The Spark Context is used to create RDDs and perform operations on them. Here's an example of creating a Spark Context:

```scala
import org.apache.spark.{SparkConf, SparkContext}

val conf = new SparkConf().setAppName("MySparkApp").setMaster("local[*]")
val sparkContext = new SparkContext(conf)
```

These are the core concepts of Spark Core that lay the foundation for distributed data processing in Apache Spark. By understanding RDDs, transformations, actions, and the Spark Context, you can build powerful and scalable data processing applications using Spark.

### Spark Core

Spark Core is the fundamental unit of the whole Spark project. Its key features are:

- It is in charge of essential I/O functionalities.
- Provide API to defines and manipulate the RDDs. Significant in programming and observing the role of the **[Spark cluster](http://data-flair.training/blogs/install-apache-spark-multi-node-cluster/)**.
- Task dispatching, scheduling
- Fault recovery.
- It overcomes the snag of**[ MapReduce](http://data-flair.training/blogs/hadoop-mapreduce-introduction-tutorial-comprehensive-guide/)** by using in-memory computation.

Spark makes use of Special data structure known as **[RDD (Resilient Distributed Dataset)](http://data-flair.training/blogs/rdd-in-apache-spark/)**. Spark Core is distributed execution engine with all the functionality attached on its top. For example, MLlib, **[SparkSQL](http://data-flair.training/blogs/spark-sql-tutorial/)**, GraphX, **[Spark Streaming](http://data-flair.training/blogs/apache-spark-streaming-comprehensive-guide/)**. Thus, allows diverse workload on single platform. All the basic functionality of Apache Spark Like **[in-memory computation](http://data-flair.training/blogs/apache-spark-in-memory-computing/), [fault tolerance](http://data-flair.training/blogs/apache-spark-streaming-fault-tolerance/)**, memory management, monitoring, task scheduling is provided by Spark Core.
Apart from this Spark also provides the basic connectivity with the data sources. For example, **[HBase](http://data-flair.training/blogs/category/hbase/)**, Amazon S3, **[HDFS ](http://data-flair.training/blogs/comprehensive-hdfs-guide-introduction-architecture-data-read-write-tutorial/)**etc.

#### Action, Job, Stage, Task<a name="action-job-stage-task" />

**Actions** are RDD’s operation. *reduce, collect, takeSample, take, first, saveAsTextfile, saveAsSequenceFile, countByKey, foreach* are common actions in Apache spark.

In a Spark application, when you invoke an action on RDD, a **job** is created. Jobs are the main function that has to be done and is submitted to Spark. The jobs are divided into **stages** depending on how they can be separately carried out (mainly on shuffle boundaries). Then, these stages are divided into **tasks**. Tasks are the smallest unit of work that has to be done the executor.

When you call `collect()` on an RDD or Dataset, the whole data is sent to the **Driver**. This is why you should be careful when calling `collect()`.

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
> 3. Other steps will be organized into ***stages\***, with each job being the result of a sequence of stages. For simple things a job can have a single stage, but the need to repartition data (for instance, the join on step 3) or anything that breaks the locality of the data usually causes more stages to appear. 
> 4. You can think of stages as computations that produce intermediate results, which can in fact be persisted. For instance, we can persist RDD1 since we'll be using it more than once, avoiding recomputation.
> 5. All 3 above basically talk about how the *logic* of a given algorithm will be broken. In contrast, a ***task\*** is a particular *piece of data* that will go through a given stage, on a given executor.

### RDD

RDD 数据模型

| 属性名                | 成员类型 | 属性含义           |
|:------------------:|:----:|:--------------:|
| dependencies       | 变量   | 生成该RDD所依赖的父RDD |
| compute            | 方法   | 生成该RDD的计算接口    |
| partitions         | 变量   | 该RDD的所有数据分片实体  |
| partitioner        | 方法   | 划分数据分片的规则      |
| preferredLocations | 变量   | 数据分片的物理位置偏好    |

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

[Cluster Mode Overview - Spark 3.4.1 Documentation](https://spark.apache.org/docs/latest/cluster-overview.html)

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

## Execution Framework???

The execution framework in Apache Spark is responsible for executing the computations on RDDs and managing the distributed processing across a cluster. Spark provides a flexible execution model that optimizes the execution of transformations and actions based on the directed acyclic graph (DAG) of the computation. Let's explore the execution framework with examples:

1. DAG Scheduling: Spark's execution framework utilizes a DAG scheduler to analyze the sequence of transformations applied on RDDs and determine an optimal execution plan. It breaks down the computation into stages, where each stage represents a set of transformations that can be executed independently. For example:

```scala
val rdd = sparkContext.parallelize(List(1, 2, 3, 4, 5))
val squaredRDD = rdd.map(num => num * num)
val sum = squaredRDD.reduce((a, b) => a + b)
```

In this example, Spark recognizes that the `map` transformation can be executed in a single stage, and the `reduce` action requires a separate stage.

2. Task Execution: Spark's execution framework assigns tasks to the available executors in the cluster. Each task is a unit of computation that operates on a subset of the data. The tasks are sent to the executors, where they are executed in parallel. Spark dynamically partitions the data and assigns partitions to different tasks for efficient parallel processing. For example:

```scala
val data = sparkContext.parallelize(1 to 1000)
val filteredData = data.filter(num => num % 2 == 0)
val count = filteredData.count()
```

In this example, Spark distributes the data across the cluster and assigns tasks to different partitions of the RDD for parallel execution.

3. Lazy Evaluation: Spark's execution framework incorporates lazy evaluation, which means that transformations on RDDs are not executed immediately. Instead, Spark builds a lineage graph that tracks the transformations applied to the RDDs. The transformations are evaluated only when an action is triggered. This allows Spark to optimize the execution plan based on the entire computation flow. For example:

```scala
val rdd = sparkContext.parallelize(List(1, 2, 3, 4, 5))
val squaredRDD = rdd.map(num => num * num)
val filteredRDD = squaredRDD.filter(num => num > 10)
val count = filteredRDD.count()
```

In this example, Spark defers the execution of transformations until the `count` action is called. It optimizes the execution plan by combining the `map` and `filter` operations into a single stage.

4. Data Shuffling: Spark's execution framework handles data shuffling, which is the process of redistributing data across the cluster. Shuffling occurs when operations like groupBy, sortBy, or join require data to be redistributed to ensure that related data is co-located. The execution framework optimizes data shuffling by minimizing data movement and leveraging in-memory operations when possible.

5. Fault Tolerance: Spark's execution framework provides fault tolerance by maintaining the lineage information of RDDs. If a partition or task fails, Spark can use the lineage graph to recompute the lost or corrupted data. This ensures the resiliency and reliability of computations.

Overall, Spark's execution framework efficiently manages the distributed execution of transformations and actions on RDDs. It optimizes the execution plan, leverages parallel processing, and provides fault tolerance to ensure fast and reliable data processing across large-scale clusters.

The Spark execution framework consists of the following components:

- **SparkContext:** The SparkContext is the entry point to the Spark API. It provides methods for creating RDDs, submitting jobs, and managing resources.
- **DAGScheduler:** The DAGScheduler is responsible for scheduling Spark jobs. It creates a Directed Acyclic Graph (DAG) of transformations and then schedules the execution of the DAG.
- **TaskScheduler:** The TaskScheduler is responsible for executing Spark jobs. It assigns tasks to executors and monitors the progress of tasks.
- **Executors:** Executors are the worker nodes in a Spark cluster. They are responsible for executing tasks.
- **Driver:** The driver is the main process in a Spark cluster. It is responsible for submitting jobs, managing resources, and collecting results.

The Spark execution framework works as follows:

1. A user submits a Spark job to the driver.
2. The driver creates a SparkContext.
3. The SparkContext creates a DAG of transformations.
4. The DAGScheduler schedules the execution of the DAG.
5. The TaskScheduler assigns tasks to executors.
6. The executors execute the tasks.
7. The driver collects the results of the tasks.

The Spark execution framework is a powerful and flexible system that can be used to run a wide variety of Spark jobs. It is designed to be efficient and scalable, and it can be used to run jobs on a variety of cluster sizes.

Here are some examples of how the Spark execution framework can be used:

- To run a simple word count job, you can use the following code:

```
import org.apache.spark.sql.SparkSession

object WordCount {

  def main(args: Array[String]) {

    // Create a SparkSession
    val spark = SparkSession.builder.appName("WordCount").getOrCreate()

    // Read the input data
    val inputData = spark.read.text("input.txt")

    // Split the input data into words
    val words = inputData.flatMap(_.split(" "))

    // Count the number of occurrences of each word
    val counts = words.countByValue()

    // Print the results
    counts.show()

  }

}
```

# API

You can write spark function (eg. map function, reduce funciton) using Java/scala/python/R API. See [api docs](https://spark.apache.org/docs/latest/).

# Installation On Yarn

See [run spark on Yarn](https://spark.apache.org/docs/2.4.0/running-on-yarn.html), https://www.linode.com/docs/databases/hadoop/install-configure-run-spark-on-top-of-hadoop-yarn-cluster/

## understand spark installation

Here, we have:

- **machine 'A'**: which is the place to install spark and run `spark-submit` script
  
  - this can be inside or outside of yarn cluster
  
  - spark is installed here
  
  - yarn configuration files(e.g. `yarn-site.xml`), hadoop configuration files (e.g. `core-site.xml`) must be maintained here. And in the `spark-env.sh`, `HADOOP_CONF_DIR` and `YARN_CONF_DIR` must be set to point the these configuration files.

- **yarn cluster**: where the spark applications really run. 
  
  - We don't need to pre-install spark here.
  
  - YARN will take care of distributing the necessary Spark binaries (JAR) to the nodes dynamically when you submit a Spark application.

- **`spark.yarn.jars` Configuration** (used in `spark-submit`)
  
  - specifies the Spark JARs that will be distributed to the YARN cluster when you submit your application. This version should match the Spark version installed on machine 'A'.
  - If not specified, the default jars, located in `lib` directory of your Spark installation on machine 'A'

- Key properties in `yarn-site.xml` that can be relevant for Spark on a client machine:

```xml
<!-- YARN ResourceManager address -->
<property>
  <name>yarn.resourcemanager.address</name>
  <value><YARN_RESOURCEMANAGER_ADDRESS></value>
</property>
```

- Key properties in `core-site.xml` that can be relevant for Spark on a client machine:
  
  ```xml
  <!-- Hadoop NameNode address -->
  <property>
    <name>fs.defaultFS</name>
    <value><HADOOP_NAMENODE_ADDRESS></value>
  </property>
  ```

### spark versions

There are 4 versions in topic:

1. **Version 1: Spark Dependency Version (Application Code):**
   
   - This is the version of Spark that your application code depends on when you write and build your Spark application. It includes Spark libraries and APIs used in your code.

2. **Version 2: Installed Spark Version on Machine 'A' (spark-submit):**
   
   - This is the version of Spark installed on the machine from which you submit your Spark application using `spark-submit`. It is the Spark version that the `spark-submit` script uses to package and submit your application to the cluster.

3. **Version 3: Spark Version for Driver and Executors (spark.yarn.jars):**
   
   - This is the Spark version used by the driver program and executors running on the YARN cluster. The `spark.yarn.jars` configuration points to the Spark JARs (libraries) that will be distributed to the YARN cluster for executing your application.
   - If `spark.yarn.jars` not specified, this version is the same as the Version 2, as the jar of that installed spark is used.

4. **Version 4: Spark Version Installed on YARN Cluster:**
   
   - This is the version of Spark installed on the machines in the YARN cluster where your Spark application will run. This is in fact the same as Version 3.  Version 3 is the configuration, and Version 4 is the final distributed binaries based on this configuration.

Generally speaking, all these versions, in fact the first 3 versions, should point to the same spark binary distribution to avoid compatibility issues. Version 1 and Version 3 must be the same. Even though when `spark.yarn.jars` provided, the Version 2 may not influence at all. We still recommend to keep it the same to avoid some unknown compatible issues.

## install

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
spark.ui.filters         org.apache.spark.deploy.yarn.YarnProxyRedirectFilter

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

| Term            | Meaning                                                                                                                                                                                                                                                                    |
|:--------------- |:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application     | User program built on Spark. Consists of a *driver program* and *executors* on the cluster.                                                                                                                                                                                |
| Application jar | A jar containing the user's Spark application. In some cases users will want to create an "uber jar" containing their application along with its dependencies**. The user's jar should never include Hadoop or Spark libraries, however, these will be added at runtime.** |
| Driver program  | The process running the main() function of the application and creating the SparkContext                                                                                                                                                                                   |
| Cluster manager | An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN)                                                                                                                                                                          |
| Deploy mode     | Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster.                                                                       |
| Worker node     | Any node that can run application code in the cluster                                                                                                                                                                                                                      |
| Executor        | A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors.                                                                                                      |
| Task            | A unit of work that will be sent to one executor                                                                                                                                                                                                                           |
| Job             | A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g. `save`, `collect`); you'll see this term used in the driver's logs.                                                                                              |
| Stage           | Each job gets divided into smaller sets of tasks called *stages* that depend on each other (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs.                                                                            |

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

## select latest record

[stackoverflow](https://stackoverflow.com/questions/55615716/select-latest-record-from-spark-dataframe)

# Hive Hints

[hive hints](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-hints.html#partitioning-hints)

# PySpark???

## context switching

When executing Python code, such as Pandas or PySpark UDFs, the related data is processed within the executor's Python interpreter. The executor's Python interpreter operates directly on the data available on the executor node, without the need to transfer the data back to the driver node.

This is one of the benefits of using PySpark and executing Python code within the Spark framework. The data is distributed across the worker nodes, and the Python code is executed in parallel on the executor nodes where the data resides. This avoids unnecessary data transfer between the executor nodes and the driver node, which can improve performance and reduce network overhead.

When you invoke Python code, such as a Pandas or PySpark UDF, on a Spark DataFrame, the processing happens in a distributed manner on the executor nodes. Each executor processes the corresponding subset of the data and applies the Python code to that subset independently. The results are then combined, and the final result is returned to the driver node.

It's worth noting that the data transfer between the executor nodes and the driver node typically occurs when there is a need to **exchange intermediate results, aggregate data, or perform actions that require collecting data to the driver**, such as calling `collect()` or `toPandas()`. In those cases, the relevant data is transferred from the executor nodes to the driver node. However, for processing performed within the executor's Python interpreter, the data remains on the executor node and is processed locally within the Python environment of each executor.

Within each executor process on the worker nodes, there is an instance of the Spark engine. The Spark engine is responsible for coordinating the execution of tasks, managing data partitions, optimizing query plans, and handling data distribution and parallelization.

The Spark engine within the executor process, often referred to as the "Spark Task Execution Engine," is implemented in Java and runs within the JVM portion of the executor. It handles the coordination and execution of tasks assigned to that executor, manages data partitioning, and applies optimizations to the execution plan.

The Python interpreter within the executor process interacts with the Spark engine through a bridge, commonly known as the PySpark API or PySpark Gateway. This bridge facilitates the communication and coordination between the Python interpreter and the Spark engine. It allows the Python interpreter to send instructions and data to the Spark engine, and the Spark engine to send results and intermediate data back to the Python interpreter.

The PySpark API acts as a bridge between the Python code and the Spark engine, enabling seamless integration between Python and Spark. It allows Python code, including Pandas or PySpark UDFs, to interact with the Spark engine's distributed computing capabilities and leverage its optimizations and parallel processing.

Through this bridge, the Python interpreter can issue Spark operations, manipulate distributed data, and coordinate with the Spark engine for task execution, data shuffling, and data serialization. The Spark engine, in turn, manages the execution plan, coordinates task execution across worker nodes, and optimizes the execution for distributed processing.

So, in summary, there is a Spark engine within each executor process on the worker nodes, and there is a bridge (PySpark API) that facilitates communication between the Python interpreter and the Spark engine within the executor process.

--------------wrong????-----------------------

1. The Python interpreter referred to in the context of PySpark is primarily associated with the driver program. The driver program runs the user code and coordinates the execution of the Spark job. It initiates the execution of tasks on the worker nodes (executors) and manages the overall workflow of the Spark application.

When a PySpark UDF is used, the UDF code itself runs within the Python interpreter on the driver. However, the data on which the UDF operates is distributed across the worker nodes. PySpark leverages a client-server architecture to transfer the necessary data from the worker nodes to the Python interpreter on the driver. This means that when the UDF is applied to a DataFrame column, the data required for computation is shipped from the executor nodes to the driver node. This movement of data incurs serialization and deserialization overhead.

2. When using PySpark to define a UDF, the UDF operates within the Python environment. The UDF defined in Python is not directly converted to a Scala UDF. Instead, PySpark provides a bridge between Python and the underlying Spark engine, allowing the execution of Python code within the Spark framework. The UDF is executed by the Python interpreter on the driver, and the necessary data is shipped from the executors to the driver for computation.

3. Let's dive into the practices mentioned in more detail:
- Use Broadcast Variables: Spark's broadcast variables allow you to efficiently share small read-only data structures across all the tasks in a Spark job. Instead of sending the data separately to each executor, the data is broadcasted to all the nodes, reducing the need for repeated serialization and deserialization. This is particularly useful when you have small lookup tables or configuration data that is required by multiple tasks. Broadcasting such variables helps in avoiding the overhead of transferring the same data to each task individually.

- Utilize Vectorized Operations: Vectorized operations refer to performing operations on entire arrays or columns of data instead of individual elements. Libraries like NumPy and Pandas provide vectorized operations, which process data in bulk using optimized low-level operations. By leveraging these libraries within PySpark, you can take advantage of efficient vectorized operations and reduce the need for individual serialization and deserialization of data. This can significantly improve performance, especially when working with large datasets.

By using broadcast variables and utilizing vectorized operations, you can optimize the performance of your PySpark applications by reducing the serialization overhead and minimizing data movement across the Python and Java/Scala runtime environments.

1. The bridge between Python and the underlying Spark engine is primarily established on the driver node. PySpark acts as a connector between the Python code running on the driver and the Spark engine, which is typically implemented in Scala and Java. The driver node runs the Python interpreter and coordinates the execution of Spark tasks on the executor nodes.

2. The Scala UDF definition/registration is sent to the executor nodes during the execution of a Spark job. When you define or register a Scala UDF, it is part of the overall job's execution plan. When the job is submitted to Spark for execution, the plan is distributed to the worker nodes (executors). The necessary functions, including the Scala UDF, are then available on each executor to perform the required computations.

3. When utilizing vectorized operations with libraries like Pandas or NumPy within PySpark, it is correct that the data required for processing needs to be sent from the data nodes to the driver node. This transfer of data incurs serialization and deserialization overhead. However, vectorized operations are designed to process data in bulk, leveraging highly optimized low-level operations. This allows for efficient computation within the Python interpreter on the driver node, potentially reducing the overall processing time. It's important to consider the trade-off between the serialization overhead and the performance gain achieved through vectorized operations. This approach is most effective when the benefits of vectorized operations outweigh the cost of data transfer between the nodes.

Certainly! Let's walk through the end-to-end process and the components involved when a PySpark application is submitted. We'll consider a scenario where both Spark UDFs and Python UDFs are registered and used, and Pandas is utilized to process certain columns. The example will cover the key components and steps involved in the process:

1. Driver Node and Application Submission:
   
   - The driver node is a machine where the PySpark application is submitted and executed.
   - The driver node runs the Python interpreter and initiates the execution of the Spark application.

2. Spark Context and Session:
   
   - The Spark Context (SparkContext) is created on the driver node as the entry point to the Spark application.
   - The Spark Session (SparkSession) is a higher-level API that provides a unified interface for interacting with Spark and enables DataFrame and Dataset operations.

3. Job Execution and Task Distribution:
   
   - The PySpark application code, including the registration of Spark UDFs and Python UDFs, is executed on the driver node.
   - The application code defines transformations and actions on Spark DataFrames or RDDs.
   - Spark breaks down the transformations into a directed acyclic graph (DAG) of stages and tasks.
   - The tasks are distributed across the worker nodes (executors) for parallel execution.

4. Data Processing:
   
   - When a transformation or action is invoked, Spark creates a logical plan that represents the computations required.
   - If a Python UDF is used, data required for processing is transferred from the executor nodes to the driver node for UDF computation.
   - If a Spark UDF is used, the UDF computation occurs within the Spark engine on the executor nodes, without data transfer to the driver.

5. Serialization and Deserialization:
   
   - Data serialization and deserialization occur when transferring data between the Python and Java/Scala runtime environments.
   - When using Pandas to process columns, the data may need to be serialized for transfer to the driver and deserialized back into Pandas DataFrame structures.

6. Spark Catalyst Optimizer and Execution:
   
   - Spark's Catalyst optimizer analyzes the logical plan and optimizes it by applying various rule-based transformations to generate an optimized physical plan.
   - The optimized physical plan is executed by the Spark engine on the executor nodes, leveraging Spark's execution engine components such as the Task Scheduler, Shuffle Manager, and Memory Management.

7. Result Aggregation and Return:
   
   - The results of transformations and actions are aggregated and returned to the driver node.
   - If required, Pandas can be used to process the result data on the driver node, leveraging its functionality for further analysis or visualization.

It's important to note that the components and steps involved can vary depending on the specifics of the PySpark application and the cluster configuration. The example provided highlights the major components and steps, focusing on the interactions between the driver node, executor nodes, and data transfer between Python and Java/Scala environments.

----------------wrong????------------------

## pyspark udf vs scala udf

Certainly! Let's delve into the performance aspect of using UDFs defined in Scala versus PySpark in more detail:

1. Execution Engine Integration: Scala UDFs have a closer integration with Spark's execution engine since Scala is the native language for Spark. This tight integration allows Scala UDFs to leverage Spark's Catalyst optimizer and take advantage of advanced query optimization techniques. This can potentially lead to better query planning and execution, resulting in improved performance.

2. Serialization and Deserialization Overhead: When using PySpark UDFs, data must be serialized and deserialized between the Python and Java/Scala runtime environments. This incurs additional overhead and can impact performance, especially when working with large datasets. In contrast, Scala UDFs avoid this serialization overhead since they operate directly within the Spark engine.

3. JIT Compilation: Scala benefits from just-in-time (JIT) compilation provided by the Java Virtual Machine (JVM). This means that Scala UDFs can be optimized at runtime, potentially resulting in better performance. In contrast, Python code executed within PySpark does not have the same level of JIT compilation, which can lead to slower execution.

4. Data Movement and Shuffling: Both Scala and PySpark UDFs may involve data movement and shuffling operations, which can impact performance. However, since Scala UDFs have a closer integration with Spark's execution engine, they may enable more efficient data processing and minimize data shuffling, leading to better performance in certain scenarios.

It's important to note that the performance impact of using UDFs can vary depending on the specific workload, data size, and operations performed. In some cases, the overhead introduced by PySpark UDFs may not be significant, especially if the UDFs themselves are not the main bottleneck in the application. Additionally, other factors such as cluster configuration, data skew, and overall data processing pipeline design can also influence performance.

To determine the performance implications of using UDFs, it's recommended to benchmark and profile your specific Spark application using both Scala and PySpark implementations. This will provide insights into the actual performance differences and help you make an informed decision based on your specific requirements and constraints.

## Arrow

[Apache Arrow in PySpark 3.4.1 documentation](https://spark.apache.org/docs/latest/api/python/user_guide/sql/arrow_pandas.html)

# Optimization

[deep dive - spark optimization](https://www.youtube.com/watch?v=daXEp4HmS-E)

[performance tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)

## Get Baseline

1. 利用 spark-ui 观察任务运行情况（long stages，spill，laggard tasks, etc.）
2. 利用 yarn 等观察资源利用情况（CPU 利用率 etc.）

## Memory spill<a name = "memory-spill" />

Spark 运行时会分配一定的 memory（可以[指定资源需求](#specify-resources))， 分 storage 和 working memory。

* storage memory 是 persist 会用的 memory。当调用 persist（或 cache，一种使用 `StorageLevel.MemoryAndDisk` 的 persist）时，如果指定的 storage_level 有 memory，那么就会将数据存到 memory。
* working memory 是 spark 运算所需要的 memory，这个大小是动态变化的。当 storage memory 占用过多内存时，working memory 就不够了。然后就会有 spill，就会慢。

memory spill 表示 working memory 不够，spark 开始使用 disk。而 disk 的 I/O 效率是极低的。所以一旦出现 spill，性能就会大大降低。

working memory 不够有很多原因：

1. Memory 资源申请的太少了，就是不够 ====》 增加 `spark.executor.memory`
   1. 数据在 memory/disk 的存储一般是 serialized，以节省空间。但数据 load 到 working memory 时，一般都是 deserialized 的，处理更快，但是更占空间。
2. 资源可以了，partition 太少，每个 partition 处理的数据太多，所以 spill 了 ====》 增加 [shuffle partition](shuffle-partition)
3. 有不均衡出现，导致某些 task 处理的数据尤其多 ====》see [balance](#balance)
4. 有太多 persist，持久化了太多东西，占用过多的 storage memory ====》see [persistence](#persistence)

![image-20210319124829765](../images/spark-storage-hierachy.png)

![memory overhead](../images/spark-memory-overhead.png)

## 指定资源需求<a name="specify-resources" />

Spark-submit 运行时，可通过指定以下参数来定义运行所需的资源：

* `--conf spark.num.executors=xx` (或 `--num-executors xx`)：指定运行时需要几个 executor（也可以通过 [dynamic allocation](#dynamic-allocation) 来根据运算动态分配 executors）
* `--conf spark.executor.memory=xxG`（或 `--executor-memory xxG`）：指定每个 executor 所需要的内存
* `--conf spark.executor.cores=xx`（或 `--executor-cores xx`）：指定每个 executor 所需要的 cores
* `--conf spark.driver.memory=xxG`（或 `--driver-memory xxG`）：指定每个 driver 所需要的内存。当执行 `df.collect()`时，会将数据 collect 到 driver，此时就需要 driver 有很多的 memory
* `--conf spark.driver.cores=xx`（或 `--driver-cores xx`）：指定每个 driver 所需要的 cores

## Some issues

### --executor-cores settinng not working

需要配置 `yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DominantResourceCalculator`，因为默认的使用的是 [DefaultResourceCalculator](https://apache.googlesource.com/hadoop-common/+/e0c9f893b684246feb5b4adbb95a05a436cdb790/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-common/src/main/java/org/apache/hadoop/yarn/util/resource/DefaultResourceCalculator.java)，它只看 memory(--executor-memory)，DominantResourceCalculator 则同时考虑 cpu 和 memory

see [stackoverflow](https://stackoverflow.com/questions/33248108/spark-executor-on-yarn-client-does-not-take-executor-core-count-configuration)

### --spark.dynamicAllocation.maxExecutors not working<a name="dynamic-allocation" />

这个需要和其他配置配合使用

> spark.dynamicAllocation.enabled = true
> This requires `spark.shuffle.service.enabled` or `spark.dynamicAllocation.shuffleTracking.enabled` to be set. The following configurations are also relevant: `spark.dynamicAllocation.minExecutors`, `spark.dynamicAllocation.maxExecutors`, and `spark.dynamicAllocation.initialExecutors` `spark.dynamicAllocation.executorAllocationRatio`

如果还不工作，可能要按 [spark dynamic allocation not working](https://community.cloudera.com/t5/Support-Questions/Spark-dynamic-allocation-dont-work/td-p/140227 设置各 nodemanager 并重启

See [spark dynamic allocation](https://spark.apache.org/docs/latest/configuration.html#dynamic-allocation)

## Partitions

接下来从输入、运行、输出三个阶段的 partition 优化来看

一般 1 partition -> 1 task，分多少个 partition，就拆多少个 task 来运行。

1. **Avoid the spills**
2. **Maximize parallelism**
   1. utilize all cores
   2. provision only the cores you need

### 输入（input）

```
spark.default.parallelism (don't use)
spark.sql.files.maxPartitionBytes (mutable，控制每个 partition 读的文件大小)
```

### Shuffle<a name="shuffle-partition" />

```
spark.sql.shuffle.partitions（控制使用多少个 partition 来 shuffle）
spark.default.parallelism（控制 rdd 的 partition 数目？？？？？？）
```

如果配置了 `spark.conf.set("spark.sql.adaptive.enabled", 'true')` 或 `spark.sql.adaptive.coalescePartitions.enabled` ，它会动态控制 parition count （参见 [coalescing post shuffle partitions](https://spark.apache.org/docs/latest/sql-performance-tuning.html#coalescing-post-shuffle-partitions)），根据 shuffle 数据大小来动态设置 partition 数目。但是这个设置可能不合理，因为 shuffle 过程中，最终操作的数据可能远大于 shuffle read 的大小，这个过程中存在 deserialize 等。如果配置了动态控制，依然出现了 shuffle spill，那么可以先关掉这个配置，手动控制 shuffle partitions 大小。

### 输出（output）

```
coalesce
repartition
repartition(range) ===> range partitioner???
df.localCheckPoint().repartition().... ==> how to use tis
```

## Balance<a name="balance" />

When some partitions are significantly larger than most, there is skew.

Balance 体现在很多方面：网络、GC、数据，当然最常见的问题是数据的不均匀。

通过查看 spark ui 可以看到不均匀的任务（这个时候需要停掉重跑）：

1. 查看 straggling tasks
   1. 查看 stage 执行进度：stage 里剩余几个 task 执行特别慢，这个时候各个 task 处理的数据肯定存在不均匀，导致那几个 task 处理的尤其慢
      1. ![image-20210317130715804](../images/spark-straggling-task1.png)
   2. 查看 stage 执行 metric：大部分时候没有 spill，但是 max 的时候有 spill；或者大部分的时候 read size 和 max read size 有很大差别
      1. ![image-20210317130618356](../images/spark-straggling-task2.png)
2. 查看 stage 里各个节点的 GC time，GC time 分布不均匀，也是有问题的（什么问题？？）
   1. ![image-20210317130839506](../images/spark-gc-skew.png)
3. 

## Persistence<a name="persistence" />

当 execution plan 中，有些 superset 被多个 subset 所使用，superset 计算复杂、耗时久，这个时候就可以选择将 superset persist，从而避免重复运算。

> [spark core](#action-job-stage-task) 中有几个概念，其中只有 action 会触发一次 dag 的运行。同一段代码，可能会生成不同的 dag，每次都需要执行。所以如果被多次使用的 superset，最好将它 cache，避免后续的重复运算。

persist/cache 要慎用，因为：

1. 占资源。当 persist 消耗了太多的 storage memory 时，就会出现 [memory spill](#memory-spill)
2. 也有时间损耗（serialize, deserialize, I/O)。persist 一般都以 serialized 的形式存储，节省空间，而 load 到 working memory 时，又需要 deserialiize

> In Python, stored objects will always be serialized with the [Pickle](https://docs.python.org/3/library/pickle.html) library, so it does not matter whether you choose a serialized level. The available storage levels in Python include `MEMORY_ONLY`, `MEMORY_ONLY_2`, `MEMORY_AND_DISK`, `MEMORY_AND_DISK_2`, `DISK_ONLY`, `DISK_ONLY_2`, and `DISK_ONLY_3`.*

### cache

Cache 是选择 default 的 persist。persist 可以选择不同的 [persistence storage level](http://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-persistence) 

With `cache()`, you use only the default storage level :

- `MEMORY_ONLY` for **RDD**
- `MEMORY_AND_DISK` for **Dataset**

With `persist()`, you can specify which storage level you want for both **RDD** and **Dataset**.

From the official docs:

> - You can mark an `RDD` to be persisted using the `persist`() or `cache`() methods on it.
> - each persisted `RDD` can be stored using a different `storage level`
> - The `cache`() method is a shorthand for using the default storage level, which is `StorageLevel.MEMORY_ONLY` (store deserialized objects in memory).

Use `persist()` if you want to assign a storage level other than :

- `MEMORY_ONLY` to the **RDD**
- or `MEMORY_AND_DISK` for **Dataset**

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

> RDDs are divided into *partitions*. These partitions themselves act as an immutable subset of the entire RDD. When Spark executes each stage of the graph, each partition gets sent to a worker which operates on the subset of the data. In turn, each worker can *cache* the data if the RDD needs to be re-iterated.
> 
> Broadcast variables are used to send some immutable state *once* to each worker. You use them when you want a local copy of a variable.
> 
> These two operations are quite different from each other, and each one represents a solution to a different problem.

## 小文件问题

[spark-sql 优化小文件过多](https://blog.csdn.net/lhxsir/article/details/87882128)

[On Spark, Hive, and Small Files: An In-Depth Look at Spark Partitioning Strategies](https://medium.com/airbnb-engineering/on-spark-hive-and-small-files-an-in-depth-look-at-spark-partitioning-strategies-a9a364f908)

#### 为什么会有小文件？

当 spark 要 write 到 hive 表时，这实际也是一个 shuffle stage，就会分很多个 sPartition (spark partition)。每个 sPartition 在处理时，都会生成一个文件（如果是动态分区，则更严重，因为每个 sPartition 的数据分布式均匀的，每个 sPartition 可能包含很多个 hive paritition key，spark 每遇到一个 partition key 就生成一个文件），那么 sPartition 数目越多（动态分区的情况下，会更不可控），文件数就会越多。

简单来说，就是 spark 的一个 stage 分成了很多个 task（shuffle partitions 控制这个数量），即 sPartition，每个 sPartition 可能对应多个 hPartitiion（hive partition）key，多个 sPartition 也对应一个 hPartition key。而每个 sPartition 里对应的每个 hPartition key，都会生成一个文件。

那么，如果一个 sPartition 和 hPartition 只是一个 **多（可控数目，对应最后每个 hPartitiion 的文件数）对一** 的情况，那么文件数就是可控的。

> 使用 hive 时，不会有小文件问题。hive 里只需要设置下边的这些参数，就
> 
> In pure Hive pipelines, there are configurations provided to automatically collect results into reasonably sized files, nearly transparently from the perspective of the developer, such as *hive.merge.smallfiles.avgsize*, or *hive.merge.size.per.task*.

#### 解决方案

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

## python udf vs scala udf

[python udf vs scala udf](https://medium.com/quantumblack/spark-udf-deep-insights-in-performance-f0a95a4d8c62)

![img](https://miro.medium.com/max/1570/1*ddtDqMvoDGhxsw0CDEnfag.png)

![img](https://miro.medium.com/max/1415/1*SMlxTZJBsAPKmpdRH5VFVw.png)

![img](https://miro.medium.com/max/1500/1*FFi8Yk6mwSc6AvI-avWcYw.png)

# Issues

## Null-aware predicate sub-queries cannot be used in nested conditions

`not in` 不能和 `or` 之类的 condition 一块用。现在好像还没有修复，参见：[SPARK-25154](https://github.com/apache/spark/pull/22141)

# References

1. Apache Spark Documentation: The official documentation for Apache Spark is an excellent resource to understand Spark's concepts, APIs, and optimization techniques. It includes a Quick Start guide and in-depth documentation on Spark's features and optimizations.

2. Hadoop Definitive Guide: Although this is a book, it is freely available online. "Hadoop: The Definitive Guide" by Tom White provides a comprehensive understanding of Hadoop and its ecosystem. It covers various optimization techniques and best practices for managing Hadoop clusters.

3. YouTube Tutorials:
   
   - Simplilearn's Spark Tutorial: Simplilearn offers a comprehensive Spark tutorial series on YouTube, covering various aspects of Spark programming, optimization, and cluster management.
   - Hadoop Tutorials by edureka!: edureka! provides a playlist of Hadoop tutorials on YouTube that cover the basics of Hadoop, including optimization and cluster management topics.

4. Apache Hadoop YouTube Channel: The official Apache Hadoop YouTube channel hosts a collection of videos covering different topics related to Hadoop. You can find videos on optimization techniques, cluster management, and other Hadoop-related aspects.

5. Hortonworks Community Connection (HCC): Hortonworks Community Connection is a platform where Hadoop enthusiasts share knowledge and resources. It includes articles, tutorials, and videos related to Hadoop optimization and cluster management. You can explore the HCC website to find relevant resources.
