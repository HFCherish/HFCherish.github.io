---
title: MaxDOP of sql server
toc: true
date: 2020-04-08 12:43:51
tags:
	sql-server
---

# What's MaxDOP

[MaxDOP](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option?view=sql-server-ver15)(**max degree of parallelism**) is an option to limit the number of processors to use in parallel plan execution. For a single query you asked to sql server, it may use multiple worker threads to process it (each query in fact contains many operations internally). The MaxDOP limit the concurrency in that case.

# What's the proper value?

By default, it's 0, which means no limit, and sql server may use all the processors it has to process a query.

## MaxDOP = 1

When setting it as 1, you alwase process the query sequentially. You suppress the parallel plan generation.

Why? While this options seems to give better performance on queries and other options, why do we want to reduce it?



Say a server 