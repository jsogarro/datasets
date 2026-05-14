# fkeys

## Overview

The `fkeys` function returns a dictionary mapping foreign-key columns to their corresponding tables.

## Syntax

```
fkeys x
fkeys[x]
```

## Parameters

**x** — a table

## Return Value

A dictionary where keys are foreign-key column names and values are the table names they reference.

## Example

```q
q)f:([x:1 2 3]y:10 20 30)
q)t:([]a:`f$2 2 2;b:0;c:`f$1 1 1)
q)meta t
c| t f a
-| -----
a| j f
b| j
c| j f
q)fkeys t
a| f
c| f
```

In this example, table `t` has two foreign-key columns: `a` and `c`, both referencing table `f`.

## Related

See [Metadata](../../basics/metadata/) for other metadata functions.
