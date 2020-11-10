---
title: sqoop
toc: true
date: 2020-11-10 13:39:30
tags:
	- big data
	- hadoop
---

# Concept

Sqoop: sq are the first two of "sql", oop are the last three of "hadoop". It **transfers bulk data between hdfs and relational database servers**. It supports:

* Full Load
* Incremental Load
* Parallel Import/Export (throught mapper jobs)
* Compression
* Kerberos Security Integration
* Data  loading directly to HIVE

> Sqoop cannot import .csv files into hdfs/hive. It only support databases / mainframe datasets import.

# Architecture

Sqoop provides CLI, thus you can use a simple command to conduct import/export.

The import/export are executes in fact through map tasks.

![image-20201110134912159](/images/sqoop-20201110134912159.png)

When Import from DB:

* it split to some map tasks. And each map task will connect to DB, and fetch some rows/tables, and write it to a file into HDFS

When export to DB:

* it also split to some map tasks. And each map task will fetch a HDFS file, and write each record in the file as a row by specified delimiter to some table.

![image-20201110135220097](/images/sqoop-20201110135220097.png)

# Sqoop v.s. Spark jdbc

sqoop uses mapreduce technique, while spark is a revolutionary engine to replace mapreduce technique with its in-memory execution and DAG smartness. Thus Spark jdbc is way faster than sqoop.

1. You can combine all the read, transform and write operations into one script/program instead of reading it separately through SQOOP in one script and then doing transformation and write in another.
2. You can define a new split column on the fly (using functions like ORA_HASH) if you want the data to be partitioned in a proper way.
3. You can control the number of connection to the database. Increasing the number of connection will surely speed up your data import.

# Common Commands

```sh
$  sqoop import \
	--connect jdbc:mysql://<ipaddress>/<database name>
	--table <my_table name>
	--username <username_for_my_sql> --password <password>
  --target-dir <target dir in HDFS where data needs to be imported>
  
$  sqoop export \
	--connect jdbc:mysql://<ipaddress>/<database name>
	--table <my_table name>
	--username <username_for_my_sql> --password <password>
  --export-dir <target dir in HDFS where data needs to be exported>
```

