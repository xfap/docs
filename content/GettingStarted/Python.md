---
title: Python Package
weight: 3
---
## Installation
```
pip install lingodb
```
## Basic API Usage
```py
import lingodb
# create connection
con = lingodb.create_in_memory()
# prints result as Apache Arrow table
print(con.sql("select 42"))
# pyarrow.Table
#: int32
#----
#: [[42]]

# create table and insert rows
con.sql_stmt("create table t (x bigint, y varchar(30) not null, z bigint not null, primary key (x))")
con.sql_stmt("insert into t(x, y, z) values (1,'foo',42), (2,'bar',7)")

# execute query on table
result = con.sql("select * from t where y='foo'")
# convert to pandas using pyarrow
df = result.to_pandas()
print(df)
#   x    y   z
# 0  1  foo  42

```

### Creating a Connection Object

A connection object can be created through two methods:
* `lingodb.create_in_memory()`: creates a connection to an empty in-memory database, or by using
* `lingodb.connect_to_db(DB_DIR)`: creates a connection to an database instance stored in the `DB_DIR` folder. 

### Executing SQL statements
SQL statements such as `CREATE TABLE`, `INSERT INTO`, `COPY FROM` and `SET ...` can be executed using the `sql_stmt` method.
For example, if we want the database to be persistent (written back into the database directory), the following statement should be executed:

```py
con = lingodb.connect_to_db("test-dir")

con.sql_stmt("SET persist=1")
```

### Executing SQL queries
Read-only SQL queries can be executed using the `sql` method.
It will return an arrow table that can be used arbitrarily and e.g. converted to pandas.
```py
print(con.sql("select * from t").to_pandas())
```

### Executing MLIR modules
Similarly, also raw MLIR modules can be executed using the `mlir` method and produce an arrow table.
```py

# select count(*) from t where x>2
con.mlir("""module {
  func.func @main() {
    %0 = relalg.basetable {table_identifier = "t"} columns: { x => @t::@x({type = i64})}
    %1 = relalg.selection %0 (%arg0 : !tuples.tuple){
        %4 = db.constant(2) : i64
        %5 = tuples.getcol %arg0 @t::@x : i64
        %6 = db.compare gt %5 : i64, %4: i64
        tuples.return %6 : i1
    }
    %2 = relalg.aggregation %1 [] computes : [@aggr0::@tmp_attr0({type = i64})] (%arg0: !tuples.tuplestream,%arg1: !tuples.tuple){
      %4 = relalg.count %arg0
      tuples.return %4 : i64
    }
    %3 = relalg.materialize %2 [@aggr0::@tmp_attr0] => ["count"] : !subop.result_table<[count: i64]>
    subop.set_result 0 %3 :  !subop.result_table<[count: i64]>
    return
  }
}
""")

```
## Querying PyArrow Tables/Pandas DataFrames
Apache Arrow tables can be imported using the `add_table` method. This also enables querying of pandas dataframes through pyarrow.

```py
import pandas as pd
df = pd.DataFrame(data={'col1': [1, 2, 3, 4], 'col2': ["foo", "foo", "bar", "bar"]})

con = lingodb.create_in_memory()
#convert data frame to pyarrow table
arrow_table=pa.Table.from_pandas(df)
con.add_table("df",arrow_table)
#query just as any other table
print(con.sql("select * from df").to_pandas())
```
Furthermore, the rows of an pyarrow table can be added to an existing table (import) using the `append_table` method:
```py
con.append_table("df",arrow_table)
```