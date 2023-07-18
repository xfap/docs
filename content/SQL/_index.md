---
title: SQL
type: docs
weight: 5
---

LingoDB supports a subset of the SQL query engine that is frequently used for analytical workloads and benchmarks.
The general syntax is quite close to PostgreSQL

## Select statement
```sql
select t1.a, sum(x) as c -- scalar expressions, aggregates, sub-queries, window functions, ...
from t1, t2 left join t3 on  t2.y=t3.y -- relations, (outer joins)
where t1.a=t2.a -- scalar expression that evaluates to an (nullable) bool (no aggregates)
group by a,b*2 -- scalar expressions or column names
having count(*)>5 -- any scalar expression, can also contain aggregates
order by a,b,c -- must be columns or expressions already occuring in the select clause
limit k -- k must be constant
```

## Creating tables
```sql
create table t(
    -- usual column definitions
    a string,
    b int,
    c decimal(4,2),
    -- ...
    -- define primary key
    primary key(a,b)
)

```

## Inserting Data
Data can be inserted using the `INSERT INTO` statement with either a `VALUES` clause or a SQL query:
```sql
insert into t (column, ...) values ('a',...),('b',...)
insert into t values ('a',...),('b',...)
insert into t select x from other
```
Furthermore, CSV files can be imported using the `COPY FROM` statement:

```sql
copy t from 't.csv' csv;
copy t from 't.csv' csv escape '\' delimiter '|' null '';
```