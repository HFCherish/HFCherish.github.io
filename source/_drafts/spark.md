---
title: spark
toc: true
date: 2019-01-10 10:17:01
tags:
	- hadoop
	- distributed computing
---

[spark](https://spark.apache.org/docs/2.4.0/) is a fast and general-purpose cluster computing system like Hadoop Map-reduce.



## Features

* **cluster computing system**. Provide API to  write computing functions.
* [Spark SQL](https://spark.apache.org/docs/2.4.0/sql-programming-guide.html). SQL for data processing, like hive?
* [MLlib](https://spark.apache.org/docs/2.4.0/ml-guide.html) for machine learning.
* [GraphX](https://spark.apache.org/docs/2.4.0/graphx-programming-guide.html) for graph processing
* [Spark Streaming](https://spark.apache.org/docs/2.4.0/streaming-programming-guide.html).

## API

You can write spark function (eg. map function, reduce funciton) using Java/scala/python/R API. See [api docs](https://spark.apache.org/docs/2.4.0/).

## Installation On Yarn

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

### history server config

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

### example execution

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