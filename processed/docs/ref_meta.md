# meta – Metadata for a table

## Syntax

```q
meta x    meta[x]
```

## Arguments

`x` is one of:
- A table in memory or memory mapped (by value or reference)
- A filesymbol for a splayed table

## Returns

A table keyed by column name with columns:

| Column | Description |
|--------|-------------|
| `c` | Column name |
| `t` | Data type |
| `f` | Foreign key (enums) |
| `a` | Attribute |

## Type Notation

The `t` column uses lowercase letters for atomic entries and uppercase letters for lists.

## Examples

### Basic usage with in-memory table

```q
q)\l trade.q
q)show meta trade
c    | t f a
-----| -----
time | t
sym  | s
price| f
size | i
```

### Accessing splayed table metadata

```q
q)show meta `trade
c    | t f a
-----| -----
time | t
sym  | s
price| f
size | i
```

### Metadata after setting attributes

```q
q)`sym xasc`trade;   / sort by sym thereby setting the `s attribute
q)show meta trade
c    | t f a
-----| -----
time | t
sym  | s   s
price| f
size | i
```

### Atomic vs. list columns

```q
q)show u:([] code:`F1; vr:(enlist 2.3))
code vr
--------
F1   2.3
q)meta u
c   | t f a
----| -----
code| s
vr  | f

q)show v:([] code:`F2; vr:(enlist (5.4; 43.2)))
code vr
-------------
F2   5.4 43.2
q)meta v
c   | t f a
----| -----
code| s
vr  | F
```

### Splayed table with sym column

```q
q)load `:db/sym  / required for meta to describe db/tr
`sym
q)meta `:db/tr
c    | t f a
-----| -----
date | d
time | u
vol  | j
inst | s
price| f
```

### Using loaded database

```q
q)\v
`s#`sym`tr
q)meta tr
c    | t f a
-----| -----
date | d
time | u
vol  | j
inst | s
price| f
```

## Notes

"The result of `meta` does not tell you whether a table in memory can be splayed, only the first item in each column is examined."

A splayed table with a symbol column requires its corresponding sym list to be loaded for `meta` to properly describe it.
