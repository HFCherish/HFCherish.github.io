---
title: duplicate rows in postgresql
toc: true
tags:
  - sql
date: 2018-11-06 14:53:28
---

参见 [Search and destroy duplicate rows in PostgreSQL](https://blog.theodo.fr/2018/01/search-destroy-duplicate-rows-postgresql/)

# Find duplicates

## using group

```sql
SELECT
  firstname,
  lastname,
  count(*)
FROM people
GROUP BY
  firstname,
  lastname
HAVING count(*) > 1;
```

## using partition

```sql
SELECT * FROM
  (SELECT *, count(*)
  OVER
    (PARTITION BY
      firstname,
      lastname
    ) AS count
  FROM people) tableWithCount
  WHERE tableWithCount.count > 1;
```

## Using not strict distinct

利用 not strict distinct `DISTINCT ON` 找到唯一的那些条，剩余的就是重复的，可以修改或删除

```sql
DELETE FROM people WHERE people.id NOT IN 
(SELECT id FROM (
    SELECT DISTINCT ON (firstname, lastname) *
  FROM people));

// more readable code
WITH unique AS
    (SELECT DISTINCT ON (firstname, lastname) * FROM people)
DELETE FROM people WHERE people.id NOT IN (SELECT id FROM unique);
```
