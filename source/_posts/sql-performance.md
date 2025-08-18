---
title: sql performance
toc: true
tags:
  - database
  - sql
date: 2020-07-22 14:37:30
---

[sql performance explained](https://use-the-index-luke.com/sql/anatomy)

- [ ] Read source code of `HashMap` in c#
- [ ] Read source code of `LinkedList` in c#
- [ ] Unlike the index, the table data is stored in a heap structure and is not sorted at all?????  Heap is either min-heap or max-heap, all is partially sorted.

# Developers need to know this?

Sql separate what & how.

However, developers needs to know how. Because the access path is what influence the performance most, and developers instead of DBAs know it.

# Structure (anatomy of an index)

> The primary purpose of an index is to **provide an ordered representation** of the indexed data. It is, however, not possible to store the data sequentially because an `insert` statement would need to move the following entries to make room for the new one. Moving large amounts of data is very time-consuming so the `insert` statement would be very slow. The solution to the problem is to establish a logical order that is independent of physical order in memory.

**Doubly-linked list: (leaf nodes)**

* manage the logic order between leaf nodes, which is redundancy of unordered table data.
* a leaf node is saved in a database block or page (the database's smallest storage unit). All blocks are of the same size, typically a few kilobytes.
* a leaf node contains several ordered index entries.
* Each index entry consists of the indexed columns (the key, column 2) and refers to the corresponding table row (via `ROWID` or `RID`)
* the index order is maintained on two different levels: the index entries within each leaf node, and the leaf nodes among each other using a doubly linked list.

![leaf nodes](https://use-the-index-luke.com/static/fig01_01_index_leaf_nodes.en.MMHwYDFb.png)

**B+ Tree (branch nodes)**

[b+ tree simulator](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)

* a balanced search tree

> The tree traversal is a very efficient operation—so efficient that I refer to it as the *first power of indexing*.
> 
> That is primarily because of the tree balance, which allows accessing all elements with the same number of steps, and secondly because of the logarithmic growth of the tree depth.
> 
>  That means that the tree depth grows very slowly compared to the number of leaf nodes. Real world indexes with millions of records have a tree depth of four or five. A tree depth of six is hardly ever seen. 

![b-tree structure](https://use-the-index-luke.com/static/fig01_02_tree_structure.en.BdEzalqw.png)

![tree traversal](https://use-the-index-luke.com/static/fig01_03_tree_traversal.en.niC7Q5jq.png)

# Slow Indexes

> An index lookup requires three steps: **(1) the tree traversal; (2) following the leaf node chain; (3) fetching the table data.** The tree traversal is the only step that has an upper bound for the number of accessed blocks—the index depth. The other two steps might need to access many blocks—they cause a slow index lookup.

The Oracle database has three distinct operations that describe a basic index lookup:

1. INDEX UNIQUE SCAN: performs the tree traversal only. (unique constraint)
2. INDEX RANGE SCAN: performs the tree traversal *and* follows the leaf node chain to find all matching entries. (non-unique constraint)
3. TABLE ACCESS BY INDEX ROWID: This operation is (often) performed for every matched record from a preceding index scan operation. This operation only get one block by the rowid once.
4. TABLE FULL SCAN: this opertion won't traverse the index tree, it scans the whole table. And it can get multiple blocks once. Sometimes it may be faster than index searching, since it can fetch batch blocks.

## over indexing

Every index causes ongoing maintenance. Function-based indexes are particularly troublesome because they make it very easy to create *redundant indexes*.

> Always aim to index the original data as that is often the most useful information you can put into an index.

# query optimizer<a name="query-optimizer" />

[slow indexes II](https://use-the-index-luke.com/sql/where-clause/the-equals-operator/slow-indexes-part-ii)

> The query optimizer, or query planner, is the database component that transforms an SQL statement into an execution plan. This process is also called *compiling* or *parsing*. There are two distinct optimizer types.
> 
> Cost-based optimizers (CBO) generate many execution plan variations and calculate a cost value for each plan. The cost calculation is based on the operations in use and the estimated row numbers. In the end the cost value serves as the benchmark for picking the “best” execution plan.
> 
> Rule-based optimizers (RBO) generate the execution plan using a hard-coded rule set. Rule based optimizers are less flexible and are seldom used today.

## How does the cost-based optimizer works

> A cost-based optimizer uses statistics about tables, columns, and indexes. Most statistics are collected on the column level: the number of distinct values, the smallest and largest values (data range), the number of `NULL` occurrences and the column histogram (data distribution). The most important statistical value for a table is its size (in rows and blocks).
> 
> The most important index statistics are the tree depth, the number of leaf nodes, the number of distinct keys and the clustering factor (see [Chapter 5, “*Clustering Data*”](https://use-the-index-luke.com/sql/clustering)).
> 
> The optimizer uses these values to estimate the selectivity of the `where` clause predicates.
> 
> If there are no statistics available—for example because they were deleted—the optimizer uses default values. The default statistics of the Oracle database suggest a small index with medium selectivity. 

The following plan shows that the `INDEX RANGE SCAN` will return 40 rows. It is a gross underestimate, as there are 1000 employees working for this subsidiary.

```sql
-- pk:  (subdiary_id, employ_id)
SELECT first_name, last_name, subsidiary_id, phone_number
  FROM employees
 WHERE last_name  = 'WINAND'
   AND subsidiary_id = 30;

# execution plan 1 (picked when no statitics, and it's in fact slower)
---------------------------------------------------------------
|Id |Operation                   | Name         | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT            |              |    1 |   30 |
|*1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES    |    1 |   30 |
|*2 |  INDEX RANGE SCAN          | EMPLOYEES_PK |   40 |    2 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
  1 - filter("LAST_NAME"='WINAND')
  2 - access("SUBSIDIARY_ID"=30)

# execution plan with statitics
---------------------------------------------------------------
|Id |Operation                   | Name         | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT            |              |    1 |  680 |
|*1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES    |    1 |  680 |
|*2 |  INDEX RANGE SCAN          | EMPLOYEES_PK | 1000 |    4 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
  1 - filter("LAST_NAME"='WINAND')
  2 - access("SUBSIDIARY_ID"=30)

# execution plan 2 (picked when with statitics, and it's in fact faster)
SELECT /*+ NO_INDEX(EMPLOYEES EMPLOYEES_PK) */ 
       first_name, last_name, subsidiary_id, phone_number
  FROM employees
 WHERE last_name  = 'WINAND'
   AND subsidiary_id = 30;
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    1 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    1 |  477 |
----------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter("LAST_NAME"='WINAND' AND "SUBSIDIARY_ID"=30)
```

The execution plan with statistics uses the column histograms to estimate the number of rows returned from the index lookup—the result is used for the cost calculation. 

Column histograms are most useful if the values are not uniformly distributed. For columns with uniform distribution, it is often sufficient to divide the number of distinct values by the number of rows in the table. This method also works when using bind parameters.

## the estimates are wrong?

Contradicting estimates often indicate problems with the statistics.

```sql
# range scan gets 40 rows, while the substitute table access gets 100 rows. It's weird estimate.
--------------------------------------------------------------
|Id |Operation                   | Name        | Rows | Cost |
--------------------------------------------------------------
| 0 |SELECT STATEMENT            |             |  100 |   41 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES   |  100 |   41 |
|*2 |  INDEX RANGE SCAN          | EMP_UP_NAME |   40 |    1 |
--------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
  2 - access(UPPER("LAST_NAME")='WINAND')
```

> The Oracle database maintains the information about the number of distinct column values as part of the table statistics. These figures are reused if a column is part of multiple indexes.

Statistics for a function-based index (FBI) are also kept on table level as virtual columns. Although the Oracle database collects the *index statistics* for new indexes automatically ([since release 10*g*](https://docs.oracle.com/cd/B14117_01/server.101/b10763/compat.htm#sthref320)), it does not update the *table statistics*. For this reason, the Oracle documentation recommends updating the table statistics after creating a function-based index:

> After creating a function-based index, collect statistics on both the index and its base table using the `DBMS_STATS` package. Such statistics will enable Oracle Database to correctly decide when to use the index.
> 
> — [Oracle Database SQL Language Reference](https://docs.oracle.com/database/122/SQLRF/CREATE-INDEX.htm#GUID-1F89BBC0-825F-4215-AF71-7588E31D8BFE__I2100962)

My personal recommendation goes even further: after every index change, update the statistics for the base table and all its indexes. That might, however, also lead to unwanted side effects. Coordinate this activity with the database administrators (DBAs) and make a backup of the original statistics.

## Statistics

How to gather the statistics?

1. mannually
   1. in oracle: `EXEC DBMS_STATS.gather_table_stats('SCOTT', 'EMPLOYEES', estimate_percent => 15, cascade => TRUE);` (see [DBMS_STATS](https://oracle-base.com/articles/misc/cost-based-optimizer-and-database-statistics#table_index_stats))
   2. In hive/spark: ((see [spark cost-based optimizer](https://www.waitingforcode.com/apache-spark-sql/spark-sql-cost-based-optimizer/read)), [spark reordering joins - cost-based optimization](https://www.waitingforcode.com/apache-spark-sql/reorder-join-optimizer-cost-based-optimization/read)) 
      1. Must generate statistics before the query execution
      2. Must enable some config: `spark.sql.cbo.enabled`, `spark.sql.cbo.joinReorder.enabled`, `spark.sql.statistics.histogram.enabled`...
      3. table level statistics: `analyze table xxx compute statitics`
      4. column level statistics: `analyze table xxx compute statistics for columns col1, col2`
      5. spark collected statistics for the following two types:
      6. ![img](https://databricks.com/wp-content/uploads/2017/08/Screen-Shot-2017-08-30-at-3.00.54-PM.png)
2. automatically
   1. By default Oracle 10g automatically gathers optimizer statistics using a scheduled job called `GATHER_STATS_JOB`. By default this job runs within a maintenance windows between 10 P.M. to 6 A.M. week nights and all day on weekends. (see [automatic statistics collection](https://oracle-base.com/articles/10g/performance-tuning-enhancements-10g#automatic_optimizer_statistics_collection))
   2. seems no automatically collect in spark/hive

# Optimization Points

The [B-Tree traversal](https://use-the-index-luke.com/sql/anatomy/the-tree) is the first power of indexing.

[Clustering](https://use-the-index-luke.com/sql/clustering) is the second power of indexing.

Pipelined `order by` is the third power of indexing.

## Where

### concatenated index (Index order)

**Rule of thumb: index for equality first—then for ranges.**

**Always aim to index the original data as that is often the most useful information you can put into an index.**

**Avoid function-based indexing for expressions that cannot be used as access predicates.**

**Avoid `select *` and fetch only the columns you need.**

> In general, a database can use a concatenated index when searching with the leading (leftmost) columns. An index with three columns can be used when searching for the first column, when searching with the first two columns together, and when searching using all columns.
> 
> Even though the two-index solution delivers very good `select` performance as well, the single-index solution is preferable. It not only saves storage space, but also the maintenance overhead for the second index. The fewer indexes a table has, the better the `insert`, `delete` and `update` performance.

```sql
CREATE UNIQUE INDEX employees_pk ON employees (employee_id, subsidiary_id);
SELECT first_name, last_name  FROM employees WHERE subsidiary_id = 20; # slow query, no index used

CREATE UNIQUE INDEX employees_pk ON employees (subsidiary_id, employee_id);
SELECT first_name, last_name  FROM employees WHERE subsidiary_id = 20; # index used
```

![img](https://use-the-index-luke.com/static/fig02_01_concatenated_index.en.Q-6eso0Z.png)

```sql
SELECT first_name, last_name, date_of_birth
  FROM employees
 WHERE date_of_birth >= TO_DATE(?, 'YYYY-MM-DD')
   AND date_of_birth <= TO_DATE(?, 'YYYY-MM-DD')
   AND subsidiary_id  = ?
```

The *access predicates* are the start and stop conditions for an index lookup. They define the scanned index range.

*Index filter predicates* are applied during the [leaf node traversal](https://use-the-index-luke.com/sql/anatomy/the-leaf-nodes) only. They do not narrow the scanned index range.

The appendix explains how to recognize access predicates in [MySQL](https://use-the-index-luke.com/sql/explain-plan/mysql/access-filter-predicates), [SQL Server](https://use-the-index-luke.com/sql/explain-plan/sql-server/filter-predicates) and [PostgreSQL](https://use-the-index-luke.com/sql/explain-plan/postgresql/filter-predicates).

![img](https://use-the-index-luke.com/static/fig02_02_range_scan.en.swMTd1Fa.png)

![img](https://use-the-index-luke.com/static/fig02_03_range_scan.en.vpSEKNtw.png)

### Function-based indexes

Almost all the db supports function-based indexes, or computed-column index. (see [db support on function-based indexes](https://use-the-index-luke.com/sql/where-clause/functions)). In implementation,  the computation result is what to be put into the index.

Although we can create function-based indexes, we must look out the over-indexing. And **use a redundant condition** on the most significant column when a range condition combines multiple columns, to avoid some  function-based indexes.

Requirements: 

* Only functions that always return the same result for the same parameters—**functions that are deterministic**—can be indexed.
* Besides *being* deterministic, PostgreSQL and the Oracle database require functions to be *declared* to be deterministic when used in an index so you have to use the keyword `DETERMINISTIC` (Oracle) or `IMMUTABLE` (PostgreSQL).

> An index whose definition contains functions or expressions is a so-called function-based index (FBI). Instead of copying the column data directly into the index, **a function-based index applies the function first and puts the result into the index.** As a result, the index stores the names in all caps notation.
> 
> The database can use a function-based index if the *exact* expression of the index definition appears in an SQL statement
> 
> **Warning**:
> 
> Sometimes ORM tools use `UPPER` and `LOWER` without the developer’s knowledge. Hibernate, for example, [injects an implicit `LOWER`](https://use-the-index-luke.com/sql/myth-directory/dynamic-sql-is-slow#myth-dynamic-sql-sample) for case-insensitive searches.

```sql
# this function cannot be used in index, since it's not derterministic.
CREATE FUNCTION get_age(date_of_birth DATE) 
RETURN NUMBER
AS
BEGIN
  RETURN 
    TRUNC(MONTHS_BETWEEN(SYSDATE, date_of_birth)/12);
END


# with concatenated indexes on (a,b) (order can be changed), the following query won't using index
SELECT a, b FROM table_name WHERE 3*a + 5 = b;
# with functional index, the following query will use index
CREATE INDEX math ON table_name (3*a - b);
SELECT a, b FROM table_name WHERE 3*a - b = -5;

# this query won't use index on numberic_number. Since to resolve this calculation expression, the optimizer need to resolve equotion, and most database vendors say no.
SELECT numeric_number FROM table_name WHERE numeric_number - 1000 > ?
```

### Parameterized queries

benefits:

* Security
  
  * Bind variables are the best way to prevent [SQL injection](https://en.wikipedia.org/wiki/SQL_injection).

* Performance
  
  * Databases with an execution plan cache like SQL Server and the Oracle database can reuse an execution plan when executing the same statement multiple times. It saves effort in rebuilding the execution plan but works only if the SQL statement is *exactly* the same. If you put different values into the SQL statement, the database handles it like a different statement and recreates the execution plan.   When using bind parameters you do not write the actual values but instead insert placeholders into the SQL statement. That way the statements do not change when executing them with different values.
  * this will only work if the data is equally distributted (for skewed data, the better plan maybe changed for each query value,  see [query optimizer](#query-optimizer)). However, generating and evaluating all execution plan variants is a huge effort that does not pay off if you get the same result in the end anyway. Not using bind parameters is like recompiling a program every time.
    * Example: Unevenly distributed status codes like “todo” and “done” are a good example. The number of “done” entries often exceeds the “todo” records by an order of magnitude. Using an index only makes sense when searching for “todo” entries in that case.
  * Similar as the `reuse exchange` in spark execution plan

You should **always use bind parameters except for** values that *shall* influence the execution plan. In all reality, there are only a few cases in which the actual values affect the execution plan. You should therefore use bind parameters if in doubt—just to prevent SQL injections.

> ### Cursor Sharing and Forced Parameterization
> 
> The more complex the optimizer and the SQL query become, the more important execution plan caching becomes. Many databases use a shared execution plan cache like DB2, the Oracle database, or SQL Server.
> 
> The SQL Server and Oracle databases have features to automatically replace the literal values in a SQL string with bind parameters. These features are called `CURSOR_SHARING` (Oracle) or forced parameterization (SQL Server).
> 
> Both features are workarounds for applications that do not use bind parameters at all. Enabling these features prevents developers from intentionally using literal values.
> 
> [Planning for Execution Plan Reuse](https://use-the-index-luke.com/blog/2011-07-16/planning-for-reuse)

Example (See also: [`PreparedStatement` ](https://docs.oracle.com/javase/6/docs/api/java/sql/PreparedStatement.html)class documentation): 

```java
int subsidiary_id;
PreparedStatement command = connection.prepareStatement(
                    "select first_name, last_name" 
                  + "  from employees"
                  + " where subsidiary_id = ?"
                  );
command.setInt(1, subsidiary_id);
```

#### Heuristic execution plan

When the data is unevenly distributed, the sharing cache may bring new problems and bugs. The optimizer may try to use some **smart logic** to make it work better.

> The Oracle database uses a shared execution plan cache (“SQL area”) and is fully exposed to the problem described in this section.

Oracle introduced the so-called bind peeking with release 9*i*. Bind peeking enables the optimizer to use the actual bind values of the first execution when preparing an execution plan. The problem with this approach is its nondeterministic behavior: the values from the first execution affect all executions. The execution plan can change whenever the database is restarted or, less predictably, the cached plan expires and the optimizer recreates it using different values the next time the statement is executed.

> 

Release 11*g* introduced adaptive cursor sharing to further improve the situation. This feature allows the database to cache multiple execution plans for the same SQL statement. Further, the optimizer peeks the bind parameters and stores their estimated selectivity along with the execution plan. When the cache is subsequently accessed, the selectivity of the current bind values must fall within the selectivity ranges of a cached execution plan to be reused. Otherwise the optimizer creates a new execution plan and compares it against the already cached execution plans for this query. If there is already such an execution plan, the database replaces it with a new execution plan that also covers the selectivity estimates of the current bind values. If not, it caches a new execution plan variant for this query — along with the selectivity estimates, of course.

Sql server has similar polices, and adds **hints** to help the decision (see [smart optimizer logic in other database](https://use-the-index-luke.com/sql/where-clause/obfuscation/smart-logic)):

> SQL Server uses so-called parameter sniffing. Parameter sniffing enables the optimizer to use the actual bind values of the first execution during parsing. The problem with this approach is its nondeterministic behavior: the values from the first execution affect all executions. The execution plan can change whenever the database is restarted or, less predictably, the cached plan expires and the optimizer recreates it using different values the next time the statement is executed.

SQL Server 2005 added new query hints to gain more control over parameter sniffing and recompiling. The query hint RECOMPILE bypasses the plan cache for a selected statement. OPTIMIZE FOR allows the specification of actual parameter values that are used for optimization only. Finally, you can provide an entire execution plan with the USE PLAN hint.

> 

The original implementation of the OPTION(RECOMPILE) hint had a bug so it did not consider all bind variables. The new implementation introduced with SQL Server 2008 had another bug, making the situation very confusing. Erland Sommarskog has collected all the relevant information covering all SQL Server releases.

### Like

`LIKE` filters can only use the characters *before the first wild card* during tree traversal. The remaining characters are just filter predicates that do not narrow the scanned index range. A single `LIKE` expression can therefore contain two predicate types: (1) the part before the first wild card as an access predicate; (2) the other characters as a filter predicate.

```sql
SELECT first_name, last_name, date_of_birth
  FROM employees
 WHERE UPPER(last_name) LIKE 'WIN%D'

# plan
---------------------------------------------------------------
|Id | Operation                   | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 | SELECT STATEMENT            |             |    1 |    4 |
| 1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES   |    1 |    4 |
|*2 |   INDEX RANGE SCAN          | EMP_UP_NAME |    1 |    2 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access(UPPER("LAST_NAME") LIKE 'WIN%D')
       filter(UPPER("LAST_NAME") LIKE 'WIN%D')
```

### index merge

is it better to create one index for each column or a single index for all columns of a `where` clause? 

* one index with multiple columns is better—that is, a concatenated or compound index.

* One index scan is faster than two.

Many database products offer a hybrid solution between B-tree and bitmap indexes. In the absence of a better access path, they convert the results of several B-tree scans into in-memory bitmap structures. Those can be combined efficiently. The bitmap structures are not stored persistently but discarded after statement execution, thus bypassing the problem of the poor write scalability. The downside is that it needs a lot of memory and CPU time. This method is, after all, an optimizer’s act of desperation.

#### bitmap index

**Bitmap indexes are almost unusable for online transaction pro­cessing (OLTP).**

The advantage of bitmap indexes is that they can be combined rather easily. That means you get decent performance when indexing each column individually. Conversely if you know the query in advance, so that you can create a tailored multi-column B-tree index, it will still be faster than combining multiple bitmap indexes.

By far the greatest weakness of bitmap indexes is the ridiculous `insert`, `update` and `delete` scalability. Concurrent write operations are virtually impossible. That is no problem in a data warehouse because the load processes are scheduled one after another. In online applications, bitmap indexes are mostly useless.

### Partial index

With *partial* (PostgreSQL) or *filtered* (SQL Server) indexes you can specify the *rows* that are indexed. It will decrease the index size and thus quicken the index search.

> The `where` clause of a partial index can become arbitrarily complex. The only fundamental limitation is about functions: you can only use deterministic functions as is the case everywhere in an index definition. SQL Server has, however, more restrictive rules and [neither allow functions nor the `OR` operator in index predicates](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-index-transact-sql).

```sql
CREATE INDEX messages_todo ON messages (receiver) WHERE processed = 'N'
```

### Obfuscated conditions

#### null in the oracle

Null is processed differently in many dbs, e.g. `is null` instead of  `= null`. But in Oracle there're more oddities.

In oracle:

1. Empty string `''` is treated and stored as null
2. for single column index, it's in fact partial index, since all the rows with null index column are not included in the index.
   1. however, for concatenated indexes, only when all the index columns are null, the row won't be indexed.
3. For some query that requests `null` rows, Oracle only use the index when it knows the index is not nullable. When the index col is nullable, oracle thinks that some rows are not included in the index. And then for some queries, the indexes won't be used, e.g. `count(*)`. Oracle can knows an index is not nullable in the following ways:
   1. `NOT NULL` constraint on the indexing column
   2. non-null constant expression as the index
   3. **Some internal functions** that oracles knows (for other user defined functitons, oracle sees it as blackbox). `NOT NULL` constraint may still be required. e.g. `UPPER(last_name)`

Solutions to index null in oracle:

```sql
# use concatenated index, for the following 2 indexes, last_name must has `NOT NULL` constraint
CREATE INDEX emp_dob_name ON employees (date_of_birth, last_name);
CREATE INDEX emp_dob_upname ON employees (date_of_birth, upper(last_name))
# however, for the following 2 queries, it will search index only when the last_name has `NOT NULL` constraint.
SELECT * FROM employees WHERE date_of_birth IS NULL;
select count(*) from employees;

# use function-index to change to non-null index. This will tell oracle the index is not nullable, too.
CREATE INDEX emp_dob ON employees (date_of_birth, '1')
```

#### date

Oracle only has Date (with time part) type, so most of the time, you: 

1. must specify the exact range
2. create simple index on the date col (instead of function-based index) & do not use function for the left part in the query condition.

```sql
# this will use the index
SELECT ...
  FROM sales
 WHERE sale_date >= TO_DATE('1970-01-01', 'YYYY-MM-DD') 
   AND sale_date <  TO_DATE('1970-01-01', 'YYYY-MM-DD') 
                  + INTERVAL '1' DAY                 

# this won't use index either, since in like, the `sale_date` will be converted to string implicitly
select * from sales where sale_date like TO_DATE('1970-01-01', 'YYYY-MM-DD') 
```

#### Implict type conversion in query

DB may conduct some implict type conversion for query (e.g. the above `like`). When this  happens, the index won't work since the index column changed. And the implict conversion alter decrease performance (even a little)

**Avoid implict type conversion**

e.g.

1. Numeric strings: `select * from test where numberic_string = 42` is in fact `select * from test where to_number(numberic_string) = 42`

```sql
# In spark, it's the same. Alwasy explicitly convert types.
create temporary view test as select * from values 
    ('one', '2.1'), ('two', '0.35')
as t1(col1, col2);

# return one, two
select * from test where col1 = 0.0;

# return one
select * from test where col1 = 0;
```

### use redundant condition/column

Although we can create function-based indexes, we must look out the over-indexing. And **use a redundant condition** on the most significant column when a range condition combines multiple columns, to avoid some  function-based indexes.

```sql
# the will use the index, too. The second condition is redundant, to use the index first.
SELECT ...
  FROM sales                  
 WHERE ADDTIME(date_column, time_column)
     > DATE_ADD(now(), INTERVAL -1 DAY)
   AND date_column
    >= DATE(DATE_ADD(now(), INTERVAL -1 DAY)) 
```

## Join

There are 3 kinds of join:

1. **nested loop join:**
   1. seems to be the least  useful. Spark doesn't has this
   2. Join order is import here
   3. n+1 problem in orm
   4. Use case: when the outer loop has very few records and the inner loop can use index, it can be really fast
2. **hash join:** 
   1. Build the smaller dataset as a hash table. In spark, broadcast hash join & shuffle hash join, both use the hash join in the second join stage.
   2. The hash table size is the key to the performance. Shrink it both vertically (use more filters and add indexes on the independent filters)  and horizontally (project columns in use)
   3. use case: often used when joining big table & small table
3. **sort merge join**
   1. Similar to spark sortMergeJoin
   2. use case: join two tables with similar size

```sql
# this will use a as outer loop
select * from
    employees a
        join employees b on a.last_name = b.last_name
where b.employee_id > 1000 and a.employee_id = 100 and a.subsidiary_id = 1;

|--Nested Loops(Inner Join, WHERE:([employees].[last_name] as [b].[last_name]=[employees].[last_name] as [a].[last_name]))
     |--Nested Loops(Inner Join, OUTER REFERENCES:([Bmk1000]))
     |    |--Index Seek(OBJECT:([employees].[employees_pk] AS [a]), SEEK:([a].[employee_id]=(100.) AND [a].[subsidiary_id]=(1.)) ORDERED FORWARD)
     |    |--RID Lookup(OBJECT:([employees] AS [a]), SEEK:([Bmk1000]=[Bmk1000]) LOOKUP ORDERED FORWARD)
     |--Table Scan(OBJECT:([employees] AS [b]), WHERE:([employees].[employee_id] as [b].[employee_id]>(1000.)))

# this will use b as outer loop even we write the join order as a first
select * from
    employees a
        join employees b on a.last_name = b.last_name
where a.employee_id > 1000 and b.employee_id = 100 and b.subsidiary_id = 1;

|--Nested Loops(Inner Join, WHERE:([employees].[last_name] as [b].[last_name]=[employees].[last_name] as [a].[last_name]))
     |--Nested Loops(Inner Join, OUTER REFERENCES:([Bmk1002]))
     |    |--Index Seek(OBJECT:([employees].[employees_pk] AS [b]), SEEK:([b].[employee_id]=(100.) AND [b].[subsidiary_id]=(1.)) ORDERED FORWARD)
     |    |--RID Lookup(OBJECT:([employees] AS [b]), SEEK:([Bmk1002]=[Bmk1002]) LOOKUP ORDERED FORWARD)
     |--Table Scan(OBJECT:([employees] AS [a]), WHERE:([employees].[employee_id] as [a].[employee_id]>(1000.)))
```

### join order

 pipelined execution

## Clustered Index

Clustering data means to store consecutively accessed data closely together so that accessing it requires fewer IO operations.

> **index clustering factor**:
> 
> The index clustering factor is an indirect measure of the probability that two succeeding index entries refer to the same table block. The optimizer takes this probability into account when calculating the cost value of the `TABLE ACCESS BY INDEX ROWID` operation.

Index-only  scan

clustered index: only used when you have one index for a table.  If a table has more than one index, then the other indexes would be the secodary index, which are very inefficient.

![img](https://use-the-index-luke.com/static/intentional-filter-predicate.en.N5J1kRR3.gif)

index-based access on a heap table:

![img](https://use-the-index-luke.com/static/fig05_02_index_on_heap_table.en.Mo+uOg1k.png)

Secondary index on an IOT:

![img](https://use-the-index-luke.com/static/fig05_02_index_on_heap_table.en.Mo+uOg1k.png)

## use index to avoid sorting

### Order by

pipelined order by

If the index order corresponds to the `order by` clause, the database can omit the explicit sort operation.

Databases can read indexes in both directions.

When using mixed `ASC` and `DESC` modifiers in the `order by` clause, you must define the index likewise in order to use it for a pipelined `order by`.

This does not affect the index’s usability for the `where` clause.

```sql
# use the index, omit order
CREATE INDEX sales_dt_pr ON sales (sale_date, product_id)
SELECT sale_date, product_id, quantity
  FROM sales
 WHERE sale_date = TRUNC(sysdate) - INTERVAL '1' DAY
 ORDER BY sale_date, product_id
---------------------------------------------------------------
|Id | Operation                   | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 | SELECT STATEMENT            |             |  320 |  300 |
| 1 |  TABLE ACCESS BY INDEX ROWID| SALES       |  320 |  300 |
|*2 |   INDEX RANGE SCAN          | SALES_DT_PR |  320 |    4 |
--------------------------------------------------------------- 

# this query can use index, too, because on some day, the index order is already requested
SELECT sale_date, product_id, quantity
  FROM sales
 WHERE sale_date = TRUNC(sysdate) - INTERVAL '1' DAY
 ORDER BY product_id

# use the index, scan reversely
SELECT sale_date, product_id, quantity
  FROM sales
 WHERE sale_date >= TRUNC(sysdate) - INTERVAL '1' DAY
 ORDER BY sale_date DESC, product_id DESC
---------------------------------------------------------------
|Id |Operation                    | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT             |             |  320 |  300 |
| 1 | TABLE ACCESS BY INDEX ROWID | SALES       |  320 |  300 |
|*2 |  INDEX RANGE SCAN DESCENDING| SALES_DT_PR |  320 |    4 |
---------------------------------------------------------------

# use mixed order, define index with asc/desc, to use the index
CREATE INDEX sales_dt_pr ON sales (sale_date ASC, product_id DESC)
SELECT sale_date, product_id, quantity
  FROM sales
 WHERE sale_date >= TRUNC(sysdate) - INTERVAL '1' DAY
 ORDER BY sale_date ASC, product_id DESC
---------------------------------------------------------------
|Id | Operation                   | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 | SELECT STATEMENT            |             |  320 |  301 |
| 1 |  TABLE ACCESS BY INDEX ROWID| SALES       |  320 |  301 |
|*2 |   INDEX RANGE SCAN          | SALES_DT_PR |  320 |    4 |
---------------------------------------------------------------
```

Figure 6.5 Database/Feature Matrix

![img](https://use-the-index-luke.com/static/fig06_05_asc_desc_nulls_overview.en.2+dRUs0p.png)

### Group by

pipelined group by

Two group by algorithms:

* Hash algorithm: aggregates the input records in a temporary hash table. Once all input records are processed, the hash table is returned as the result. 
* sort/group algorithm: first sorts the input data by the grouping key so that the rows of each group follow each other in immediate succession. Afterwards, the database just needs to aggregate them

In general, both algorithms need to materialize an intermediate state, so they are not executed in a pipelined manner. Nevertheless the sort/group algorithm can use an index to avoid the sort operation, thus enabling a pipelined `group by`.

```sql
# omit sort using the index
CREATE INDEX sales_dt_pr ON sales (sale_date, product_id)
SELECT product_id, sum(eur_value)
  FROM sales
 WHERE sale_date = TRUNC(sysdate) - INTERVAL '1' DAY
 GROUP BY product_id
---------------------------------------------------------------
|Id |Operation                    | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT             |             |   17 |  192 |
| 1 | SORT GROUP BY NOSORT        |             |   17 |  192 |
| 2 |  TABLE ACCESS BY INDEX ROWID| SALES       |  321 |  192 |
|*3 |   INDEX RANGE SCAN          | SALES_DT_PR |  321 |    3 |
---------------------------------------------------------------

# cannot use index to omit the sort in the query, thus choose the hash algorithm instead.
SELECT product_id, sum(eur_value)
  FROM sales
 WHERE sale_date >= TRUNC(sysdate) - INTERVAL '1' DAY
 GROUP BY product_id
---------------------------------------------------------------
|Id |Operation                    | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT             |             |   24 |  356 |
| 1 | HASH GROUP BY               |             |   24 |  356 |
| 2 |  TABLE ACCESS BY INDEX ROWID| SALES       |  596 |  355 |
|*3 |   INDEX RANGE SCAN          | SALES_DT_PR |  596 |    4 |
---------------------------------------------------------------
```

## Partial Results

Pipeline execution

Inform the database whenever you don’t need all rows.

The database can only optimize a query for a partial result if it knows this from the beginning.

```sql
#  The respective top-N syntax just aborts the execution after fetching ten rows. And it omit sort by using index.
# SQL Server provides the top clause to restrict the number of rows to be fetched.
CREATE INDEX sales_dt_pr ON sales (sale_date, product_id)
SELECT TOP 10 *
  FROM sales
 ORDER BY sale_date DESC
-------------------------------------------------------------
| Operation                     | Name        | Rows | Cost |
-------------------------------------------------------------
| SELECT STATEMENT              |             |   10 |    9 |
|  COUNT STOPKEY                |             |      |      |
|   VIEW                        |             |   10 |    9 |
|    TABLE ACCESS BY INDEX ROWID| SALES       | 1004K|    9 |
|     INDEX FULL SCAN DESCENDING| SALES_DT_PR |   10 |    3 |
-------------------------------------------------------------

# without the index, it must sort the whole table first, and then stroke the result
drop index sales_dt_pr
--------------------------------------------------
| Operation               | Name  | Rows |  Cost |
--------------------------------------------------
| SELECT STATEMENT        |       |   10 | 59558 |
|  COUNT STOPKEY          |       |      |       |
|   VIEW                  |       | 1004K| 59558 |
|    SORT ORDER BY STOPKEY|       | 1004K| 59558 |
|     TABLE ACCESS FULL   | SALES | 1004K|  9246 |
--------------------------------------------------
```

# Control the index scale!

**Index maintenance is expensive! The tree balance, the block size**... e.g. During insert, once the correct leaf node has been identified, the database confirms that there is enough free space left in this node. If not, the database splits the leaf node and distributes the entries between the old and a new node. This process also affects the reference in the corresponding branch node as that must be duplicated as well. Needless to say, the branch node can run out of space as well so it might have to be split too. In the worst case, the database has to split all nodes up to the root node. This is the only case in which the tree gains an additional layer and grows in depth.

Nevertheless, the performance without indexes is so good that it can make sense to **temporarily drop all indexes while loading large amounts of data**—provided the indexes are not needed by any other SQL statements in the meantime. This can unleash a dramatic speed-up which is visible in the chart and is, in fact, a common practice in data warehouses.

# Horizontal scalability

Bigger hardware is not always faster—but it can usually handle more load. 

With more nodes, you have complex consistency problems to solve; and there is network latency and even firewall latency between nodes. It is really not always faster, even slower.

> Performance has two dimensions: response time and throughput.
> More hardware will typically not improve query response time.
> Proper indexing is the best way to improve query response time.

# Questions

- [x] How's the tree traversal going? especially on each node, branch node, leaf node?
1. For leaf node, 
   1. if it's index unique scan, then it will locate a leaf node, and then use binary search or traverse that leaf node (from the image, it should be  using tragersing), that will touch the target.
   2. If it's index range scan, then it will locate a start_leaf_node (maybe end_leaf_node by the sort order), and then it can determine all the rows to fetch.
   3. If decide to not use index, it will traverse all the table rows
2. For branch node,
   1. if it's the index unique scan, then it will traverse the balance tree to get to the only target leaf node.
   2. if it's the index range scan, then it will traverse the balance tree to get to the start_leaf_node (and the end_leaf_node ===> maybe optional)
   3. if it's no index scan, then it will not traverse any branch nodes, and to to the table directly
- [x] If we traverse the branch node & the leaf node in the same way, and the branch node & the leaf node contain the same count of entries, then they are the same for step (1) and step (2)
1. for unique scan, yes, it is. For range scan, no, you need to traverse several leaf nodes sequentially.
- [x] For equality condition and concatenated index, does the order in where matter?
  - [x] No. What matters is the the order in index. When there is  range condition in where, the index column order matters. 
- [x] why for like operator, it only uses the characters *before the first wild card* during tree traversal???
  - [x] because if  you want to traverse, you must know some leading chars to conduct the traversing. No leading chars, how to traverse?
- [x] if we create indexes for multiple column, in index searching, will it use all these indexes or just pick one???
  1. will use all  indexes, and then trigger the index merge
  2. Then the database must scan both indexes first and then combine the results. The duplicate index lookup alone already involves more effort because the database has to traverse two index trees. Additionally, the database needs a lot of memory and CPU time to combine the intermediate results.
- [x] how does the bitmap index work?
  - [x] [bitmap explain](http://yihongwei.com/2012/03/data-structure-bitmap/)
  - [x] [bitmap use cases](http://yihongwei.com/2012/03/data-structure-bitmap/)
  - [x] [bitmap index in example](https://www.cnblogs.com/lbser/p/3322630.html)
- [x] Why is there nested loop join? How oracle choose the 3 kinds of join? Is join order important for the hash join & sort merge join?
  - [x] Nested loop join: when the outer loop records are little and the inner loop can use index, it may be very fast, whereas the sort merge join must sort & merge which can be slower
  - [x] Sort merge join: often used when join two tables with simlar size
  - [x] hash join: other used when join big table and small table
  - [x] Join order: seems that the optimizer will optimize it.
- [x] How to share this knowlege?
  - [x] By knowing the index structure, what improvment we can conduct?
  - [x] What kind of strategy to improve performance?
    - [x] use the index
    - [x] join (order, hash join, sort merge join)
    - [x] clustering data
    - [x] decrease I/O
  - [x] some principals
- [x] Can hive have index?
  - [x] it has index first, but dropped since 3.0 because it's kind of useless in hive
- [x] why does hive remove index?
  - [x] because if we save data in columnar file formats (parquet/orc), the data is kind of indexed by nature. 
  - [x] And there's materialized views with automatic rewriting. This will improve performance to some extent, maybe from this way it works similarily to indexing??
  - [x] see [什么是列式存储](https://juejin.cn/post/6844904118872440840)

# practice

```sql
# concatenated indexes order (pk1, pk2, pk3)
select * from test where pk1 = 'x' and pk2 = 'y';
select * from test where pk2 = 'y' and pk1 = 'x';

# index range search maybe slower than table full scan
# does the cost, rows in explain correct when there's no statistics? How to get the statistics?
select * from test where pk1 = 'x' and val1 = 'v';
```

# Sharing notes

- 今天关注的是 sql 的 how，为什么要关注 how？
  - Sql 分离了 what & how。sql 语句是对 what 的描述，但我们需要关注 how 吗？
    - 需要。sql 的写法，直接影响 performance，很多时候是很大程度的影响，所以需要关注
- 数据库存储结构是怎么样的？（剖析了结构后，才能知道 how）
  - 索引的结构：b+ 树 + 双向链表
  - 索引的分类：
    - 复合索引
    - function-based 索引
    - clustered 索引
    - Partial index
- Query optimizer（how）
  - Cost-based optimizer
  - execution plan
  - execution plan 是怎么选择的
  - statistics 是怎么获取的？
- 基于我们定义的索引，query 是如何被执行的？（How with instances）
  - 简单的 equal 查询 （use the index）
  - 简单的 range 查询（use the index）
  - function-based 查询（use the index？？？--- need to change the index）
  - 复合索引查询（wrong order）（use the index？？？---no)
  - 复合索引查询（correct order - all cols)（use the index)
  - 复合索引查询（correct order - first few cols)（use the index）
  - 聚簇索引（单 table 单索引）
  - 聚簇索引（单 table 多索引）
  - order by (use the index)
  - group by (use the index)
  - parameterized queries（why it's better）
  - like
  - Implict type conversion
- Some key points
  - index for equality first—then for ranges.
  - Keep the scanned index range as small as possible
  - Take care of over-indexing. 
  - Always aim to index the original data as that is often the most useful information you can put into an index.
  - Avoid function-based indexing for expressions that cannot be used as access predicates.
  - Avoid `select *` and fetch only the columns you need.
  - concatenated index instead of multiple single col index on a table
  - index order is very important
  - prefer parameterized query
  - Avoid implict type conversion
  - Always aim to index the original data as that is often the most useful information you can put into an index
  - only use clustered index when you have one index for a table
  - be carefule with `like`, expecially when the leading chars are generic chars 

# 
