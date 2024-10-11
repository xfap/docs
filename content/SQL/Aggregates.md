---
title: Aggregations and Window Functions
weight: 3
---
Currently the following aggregate functions are supported:

| Function       | Description                                                         |
|----------------|---------------------------------------------------------------------|
| min(x)         | Returns the minimal value of x, or null if no rows were processed   |
| max(x)         | Returns the maximal value of x, or null if no rows were processed   |
| sum(x)         | Returns the sum of all x values, or null if no rows were processed  |
| count(x)       | Returns the number of rows that have a non-null value of x          |
| avg(x)         | Returns the average value of x, or null if no rows were processed   |
| count(*)       | Returns the total number of rows, that is always a non-null integer |
| stddev_samp(x) | Returns the sample standard deviation.                              |

Furthermore, instead of `x`, also `distinct x` can be used for every aggregate function to only aggregate distinct values.

## Window Functions
Currently a subset of window functionality is supported:

```sql
-- sum(x) is only chosen as example. Other aggregate functions/window functions can be used as well
select sum(x) over (partition by y order by z) ...
select sum(x) over (partition by y order by z rows between unbounded preceding and current row) ...
select sum(x) over (partition by y order by z rows between 100 preceding and 100 following) ...
```
Additionally, the following window functions are supported
| Function       | Description                                                         |
|----------------|---------------------------------------------------------------------|
| rank(x)         | Returns the rank of the current row in the current frame |
