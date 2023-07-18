---
title: Expressions
weight: 2
---

## Arithmetic Expressions
The standard arithmetic expressions are supported for numerical values:
```sql
select 3+2; -- returns 5
select 3-2; -- returns 1
select 3*2; -- returns 6
select 3/2; -- returns 1
select 3::decimal(1,0)/2; -- returns 1.5000000000000000000
select 3%2; -- returns 1 (modulo/remainder)
```
A subset is also supported for dates/intervals:
```sql
select '2023-07-18'::date + interval '5 day'; -- returns 2023-07-23
select '2023-07-18'::date - interval '5 day'; -- returns 2023-07-13
```

## Comparisons
Values can be compared using the standard comparison operations `=`,`!=` or `<>`,`<`, `>`, `<=`, `>=`.
In addition the following operations can be used to compare values:
```sql
x between y and z -- x>=y and x<=z
x not between y and z -- x<y or x>z
x in (1,2,3,4) -- returns true if x is one of 1,2,3,4
x is null -- true if x is null, false otherwise
x is not null -- false if x is null, true otherwise
```
## Boolean Expressions
`and`, `or`, and `not` can be used as expected (including three-valued logic).
```sql
select null and false; --returns false
select null and true; --returns null
select null or true; --returns true
select null or false; --returns null
select not true; -- returns false
```

## Casting
Values can be casted to a different type using one of the two cast constructs:
```sql
select cast(1.2 as string);
select '1.2'::float(32);
```
Note that the exact behavior depends on the source and destination types.

## Case statement
The case statement can be used to implement switches based on a condition:
```sql
select x, case when x>42 then x/2 else 0 end from t; -- simple ternary expression: x>42?x/2:0
select x, case when x=1 then 10 when x=2 then 20 else 0 end from t; -- chained if ... elseif ... else
select x, case x when 1 then 10 when 2 then 20 end from integers; -- switch x case 1: ...
```
## Coalesce
Coalesce returns the first non-null argument:
```sql
select coalesce(null,1); -- returns 1
```

## Sub Queries
(Correlated) sub-queries are supported in the select and where clauses.
The following kinds of sub-queries are supported:

```sql
(select ...) -- can return at most one value
exists(select ...) -- checks if any row is returned
not exists(select ...) -- checks that no row is returned
x in(select y from ...) -- is x in the returned values
x not in (select y from ...) -- is x not contained in the returned values
x = all(select y from ...) -- true if every y has the value of x
x = any(select y from ...) -- true if any y value has the value of x
```

## Strings
```sql
select 'a' || 'b'; -- string concatenation -> 'ab'
select 'hello world' like 'hello %'; -- pattern matching -> true
```

## Scalar Functions
Currently only few built-in functions are supported as others simply weren't necessary until now.

| Function       | Description | Example    | Result     |
|----------------|-------------|------------|------------|
| `date_part` | extract a part of a date          | `date_part('year','2023-07-11')` | `2023` |
| `substring` | substring of string        | `substring('hello world',2,5)` | `"ello"` |
| `abs` | absolute value of numer        | `abs(-1.5)` | `1.5` |
| `upper` | transform string to upper case        | `upper('abc'::string)` | `"ABC"` |
| `round` | found numer to number of digits       | `round(1.0006,3)` | `1.0010` |
| `hash` | fast hash function        | `hash(42)` | `539517765266996231` |