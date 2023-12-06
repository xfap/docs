---
title: Command Line Tools
type: docs
weight: 2
---
LingoDB comes with a few command line tools to simplify experimentation, development and debugging.

## Interactive SQL Shell
```sh
$ sql DBDIR
sql> select 1
```

Similar to other systems, LingoDB can also be used interactively using the `sql` binary that is pointed to a (possibly empty) directory that holds the database to be queried. Each query must be terminated by a `;`. By default, only a *read-only* session is created. For persistent changes enter `SET persist=1;`.

## Converting SQL to MLIR
```sh
$ sql-to-mlir SQL-File DBDIR
```
Using the `sql-to-mlir` tool, SQL queries can be converted to a corresponding, unoptimized MLIR module. As this requires the database schema, also the database directory must be provided.

## Performing Optimizations and Lowerings
```sh
$ mlir-db-opt [--use-db DBDIR] [Passes] MLIR-File
```
The `mlir-db-opt` command can be used to manually apply MLIR passes on a MLIR module provided by a file. For high-level optimizations that require e.g. database statistics, the database directory should be provided using the `--use-db` argument.

## Running MLIR Modules

```sh
$ run-mlir MLIR-File [DBDIR]
```
MLIR modules can be executed using the `run-mlir` binary. A database directory can be provided as second argument.

## Running SQL queries

```sh
$ run-sql SQL-File [DBDIR]
```
Single (read-only) SQL queries can be run with the `run-sql` utlity. If the query requires a database, the corresponding database directory must be provided as second argument.


## The Trace of a Query
With the following commands you can explore how a SQL query gets compiled layer by layer by looking at the different files:
```sh
# write example query to file
$ echo "select * from studenten where name='Carnap'" > test.sql
# translate sql to canonical MLIR module
$ sql-to-mlir test.sql resources/data/uni/ > canonical.mlir
# perform query optimization
$ mlir-db-opt --use-db resources/data/uni/ --relalg-query-opt canonical.mlir > optimized.mlir
# lower relational operators to sub-operators
$ mlir-db-opt --lower-relalg-to-subop optimized.mlir > subop.mlir
# lower sub-operators to imperative code
$ mlir-db-opt --lower-subop subop.mlir > hl-imperative.mlir
# lower database-specific scalar operations
$ mlir-db-opt --lower-db hl-imperative.mlir > ml-imperative.mlir
# lower mid-level abstraction (such as arrow tables) to low-level imperative code
$ mlir-db-opt --lower-dsa ml-imperative.mlir > ll-imperative.mlir
```