---
title: yarn
toc: true
tags:
  - hadoop
  - distributed computing
  - resource manager
  - big data
date: 2019-01-09 17:27:05
---


# [yarn architecture](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)

Yarn is used to **manage/allocate cluster resource** & **schedule/moniter jobs**. These parts -- resource manager -- are split up from hadoop framework.

![yarn architecture](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/yarn_architecture.gif)

Yarn has two main components:

* **Schedular**: manage resources (cpu, memory, network, disk, etc.) and allocate it the applications.
  * node manager will tell Schedular the node resource info (node status)
  * application master will ask Schedular for resources.
  * When partitioning resources among various queues, applications, Schedular supports pluggable policies. For example:
    * [CapacityScheduler](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html) allocate resources by tenant request. It's used especially for **multi-tenant scenario**, designed to allow sharing a large cluster while giving each **organization** capacity guarantees. Each client/tenant can request any resources that are not used by others. And there's strict ACLs to ensure the **security** of resources between tenants. The primary abstraction is queue. Different tenant use different queue to utilize the resources. And **hierachical queues** are provided to support data separation in one tenant.
    * [FairScheduler](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/FairScheduler.html) assigning resources to appliations such that all apps get, **on average**, **an equal shares of resources over time**. It's mainly designed to share cluster between **a number of users**. It lets short apps are completed in a reasonable time while not starving long-lived apps. (resources might free up when new apps are submitted).
* **ApplicationManager**: accept job-submisons, negotiate to exeuct application masters, and moniter reboot app master when failure.
  * AppMaster are the one who 
    * apply to Schedular for resources
    * boot up job execution
    * moniter the job execution status
    * tell app manager if the job fails or succeeds.

Other components:

* [ReservationSystem](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/ReservationSystem.html): reserve some resources to ensure the predictable execution of important jobs.
* [YARN Federation](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/Federation.html): join clusters to scale and allow multiple independent clusters.

## an example

Take hive as example:

### load data -- non-distributed-computing jobs

1. user uses hive command to load data into hive table. (eg. `LOAD DATA LOCAL INPATH '/path/to/datafile/' OVERWRITE INTO TABLE table_name;`)
2. hive calls hdfs to write data.
3. node inform schedular the new node status.

### query data -- distributed computing jobs

1. user uses hive command to query data (eg. `select count(*) from xxx`)
2. hive submits a map-reduce job to appliction manager
3. application manager applies to Schedular (??? not sure) for a container to execute application master and boots it.
4. application master applies to Schedular for resoures to excute map-reduce job and boots the job.
5. the map-reduce job get input data from hdfs, and write output data into hdfs
6. the map-reduce job informs the application master the status of the job.
7. application manager will restart application master on failure (application failure/hardware failure). (when application failed, the job informs the app master, and app manager knows it, and then reboot it)

# JobHistoryServer

On YARN all applications page, here's a link to job history. However, you must config to make it take effect.

Follow the instructions [config of johhistory in hadoop](https://blog.csdn.net/xiaoduan_/article/details/79689882). Also, see [Hadoop Cluster Setup](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/ClusterSetup.html) to get info about starting log and jobhistory server. See [History Server Rest API](https://hadoop.apache.org/docs/r2.4.1/hadoop-yarn/hadoop-yarn-site/HistoryServerRest.html), [JobHistoryServer javadoc](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-hs/apidocs/index.html?org/apache/hadoop/mapreduce/v2/hs/package-tree.html)

> Notes:
>
> The host of `mapreduce.jobhistory.webapp.address` and `mapreduce.jobhistory.address` may need to be set as the real ip (get from `ipconfig getifaddr en0`) or some other host (eg. cncherish.local) instead of `localhost`.
>
> When start history server, you can see the start host in the log. It may look like this:
>
> ```
> STARTUP_MSG: Starting JobHistoryServer
> STARTUP_MSG:   host = CNcherish.local/192.168.xx.xxx
> STARTUP_MSG:   args = []
> STARTUP_MSG:   version = 3.1.1
> ```
>
> This might be because the JobHistoryServer told [yarn web proxy](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/WebApplicationProxy.html#Configurations) that its host is 'cncherish.local/192.168.xx.xxx' (mapping host 'cncherish.local' to the real ip '192.168.xx.xxx'), while yarn knows that history host for map-reduce job is 'localhost' from `mapred-site.xml` --- the config for map-reduced jobs. The incompatible info reduce the jobhistory link is unreachable.

1.add the following properties into the `mapred-site.xml` (config the map-reduce framework)

```xml
<!-- config to persist the jobhistory logs in hdfs -->
<property>
  <name>mapreduce.jobhistory.cleaner.enable</name>
  <value>false</value>
  <description></description>
</property>

<!-- 设置jobhistoryserver 没有配置的话 history入口不可用 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>192.168.x.xxx:10020</value>
</property>

<!-- 配置web端口 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>192.168.x.xxx:19888</value>
</property>

<!-- 配置正在运行中的日志在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.intermediate-done-dir</name>
    <value>/history/done_intermediate</value>
</property>

<!-- 配置运行过的日志存放在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.done-dir</name>
    <value>/history/done</value>
</property>
```

2.add the following properties into the `yarn-site.xml` (config the yarn --- resource manager)

```xml
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
```

3.start the historyserver

```sh
# The following command will run server as a background daemon
$ mapred --daemon start historyserver

# The following command will run server on the current terminal.
# In this way, you can know how the server is started, stopped and what it does.
# Also you can know the real server host from the log, which should be aligned by the mapred-site.xml
$ mapred historyserver
```

# rest api

* yarn rest api:  throught postman http://localhost:8088/ws/v1/cluster/apps (get all the apps)
* history rest api: http://cnpzzheng.local:19888/ws/v1/history (get server info)
  * use 19888 instead of 10020