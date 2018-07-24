---
title: hikariCP configuration
toc: true
tags:
  - database
date: 2018-07-19 10:56:58
---


[hikariCP](https://github.com/brettwooldridge/HikariCP) 是一个轻量级的数据库连接池。引用 [数据库连接池性能对比](https://github.com/brettwooldridge/HikariCP) 的说法（我并没有测试过）：

> 1. 性能方面 hikariCP>druid>tomcat-jdbc>dbcp>c3p0 。hikariCP的高性能得益于最大限度的避免锁竞争。
> 2. druid功能最为全面，sql拦截等功能，统计数据较为全面，具有良好的扩展性。
> 3. 综合性能，扩展性等方面，可考虑使用druid或者hikariCP连接池。
> 4. 可开启prepareStatement缓存，对性能会有大概20%的提升。

在使用 spring jpa 时，默认使用的连接池是 hikariCP，所以最终采用了这个连接池。

使用过程中出现了一些坑，总结一下。

# java.sql.SQLTransientConnectionException

仅使用默认配置，在运行所有测试时，会出现如下异常信息：

![sql exception.png](https://upload-images.jianshu.io/upload_images/721960-3c58b4722a0f8ab5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是因为默认的连接池数量是 10，而并行运行测试时，连接池数量不够了。通过设置 [`maximumPoolSize`](https://github.com/brettwooldridge/HikariCP#frequently-used) 解决。

```yaml
spring.datasource.hikari.maximum-pool-size:1000
```

# too many connections

在进行上述设置后，启动应用，连接数据库，发现数据库无法连接，报 `too many connections`。

## fixed-size pool

按 [hikariCP owner 的解释](https://github.com/brettwooldridge/HikariCP/issues/657)，这可能是因为当设置 `maximumPoolSize` 后，这就变成了 fix-size 的连接池了，即总是会占有 1000（上边的设置）个连接池。idle 连接不会被释放，因为释放了也要创建新的 idle 连接池来保证 fix-size。从而新的连接就无法建立了。

>When running as a fixed-size pool (default) the `idleTimeout` has no effect. Your example is a fixed-size pool -- when `minimumIdle` is not defined it defaults to `maximumPoolSize`.
>
>`idleTimeout` is meant to shrink the pool from `maximumPoolSize` down toward `minimumIdle` when connections are unused in the pool. However, when `minimumIdle` == `maximumPoolSize` then closing an "idle" connection makes no sense as it will be replaced immediately in the pool.

所以配置 `minimumIdle` 可以解决上述问题。

```yaml
spring.datasource.hikari.maximum-pool-size:1000
spring.datasource.hikari.miniumIdle:10
```

## wait-timeout too long

有时就是因为 connection 一直没被释放，这可能是这是的 connection wait\_timeout 太长了。参照 [change the mysql timeout on a server](https://support.rackspace.com/how-to/how-to-change-the-mysql-timeout-on-a-server/) 来了解 'wait_timeout' 的设置。

引用原文：

>Choose a reasonable `wait_timeout` value. Stateless PHP environments do well with a 60 second timeout or less. Stateful applications that use a connection pool (Java, .NET, etc.) will need to adjust `wait_timeout` to match their connection pool settings. The default 8 hours (`wait_timeout = 28800`) works well with properly configured connection pools.
>
>Configure the `wait_timeout` to be slightly longer than the application connection pool’s expected connection lifetime. This is a good safety check.

在设置了合理的 mysql `wait_timeout` 后，同样也设置 hikariCP 的连接池空闲时间，参考[FAQ](https://github.com/brettwooldridge/HikariCP/wiki/FAQ#q-i-am-getting-a-commysqljdbcexceptionsjdbc4communicationsexception-communications-link-failure-exception-logged-in-the-isconnectionalive-method-of-hikaripool-in-my-logs-what-is-happening)

>If you set the MySQL `wait_timeout = 28800` (seconds = 8 hours), you should set HikariCP `idleTimeout` and `maxLifetime` to the slightly shorter 28000000 (milliseconds = 7 hours 46 minutes).

```yaml
spring.datasource.hikari.maximum-pool-size:1000
spring.datasource.hikari.miniumIdle:10
spring.datasource.hikari.idleTimeout:28000000
spring.datasource.hikari.maxLifetime:28000000
```

# 性能优化配置

按照前边的说法，合理启用 prepareStatement 缓存，可以大幅提升性能。官方推荐的配置可参考 [mysql configuration](https://github.com/brettwooldridge/HikariCP/wiki/MySQL-Configuration)

一个典型配置如下：

```yaml
jdbcUrl=jdbc:mysql://localhost:3306/simpsons
user=test
password=test
dataSource.cachePrepStmts=true
dataSource.prepStmtCacheSize=250
dataSource.prepStmtCacheSqlLimit=2048
dataSource.useServerPrepStmts=true
dataSource.useLocalSessionState=true
dataSource.rewriteBatchedStatements=true
dataSource.cacheResultSetMetadata=true
dataSource.cacheServerConfiguration=true
dataSource.elideSetAutoCommits=true
dataSource.maintainTimeStats=false
```