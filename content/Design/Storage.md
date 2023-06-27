---
title: Storage
weight: 1
---
The research conducted with LingoDB does not focus on storage aspects of database systems.
Thus, LingoDB does not come with an optimized storage backend and currently does not provide transactional semantics.

## In-Memory Format: Apache Arrow
The Apache Arrow columnar layout is used for the in-memory representation of tabular data.
Thus, LingoDB can exchange data with existing libraries and frameworks withoug any overhead and can directly query Apache Arrow tables.

## Persistent Storage
For many practical purposes, persistent storage is required.
We chose a pragmatic approach:

1. Each database is represented by multiple files placed in one *database directory*
2. In this directory, each table is represented by multiple files, each starting with the name of the table:
    1. *name*`.metadata.json`: stores metadata relevant to LingoDB. This includes basic informations like column names and internal column types, but also statistics and available indices
    2. *name*`.arrow`: Stores the contents of the table using Apache Arrow's IPC-Format
    3. *name*`.arrow.sample`: Optionally stores an sample of up to 1024 rows randomly selected from the table.

Given the database directory, LingoDB automatically detects the available tables, loads the metadata, data, and samples.
