---
title: Data Types
weight: 1
---
Currently, the following data types are supported:

* `bool`: boolean type that can hold two values: `true` and `false`
* `int4`,`int`,`integer`: 32-bit integer type
* `int8`,`bigint`: 64-bit integer type
* `float4`: float with a precision of 32-bit
* `float8`: float with a precision of  64-bit
* `decimal(p,s)`: decimal type with precision `p` and scale `s`
* `string`: string type (utf8, but string operations currently only assume ASCII)
* `char(length)`: char type, length must be <=8 
* `date`: stores date
* `interval`: stores an time interval