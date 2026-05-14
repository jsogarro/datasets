# QSQL Query Templates

## Overview

The qSQL query templates provide a SQL-like syntax for querying kdb+ tables. Four main templates exist:

- **select**: Return part of a table, possibly with new columns
- **exec**: Return columns from a table, possibly with new columns
- **update**: Add rows or columns to a table
- **delete**: Remove rows or columns from a table

## Template Syntax

```
select [Lexp] [ps] [by pb] from texp [where pw]
exec [distinct] [ps] [by pb] from texp [where pw]
update ps [by pb] from texp [where pw]
delete from texp [where pw]
delete ps from texp
```

Evaluation order:
1. From phrase (texp)
2. Where phrase (pw)
3. By phrase (pb)
4. Select phrase (ps)
5. Limit expression (Lexp)

## From Phrase

The From phrase is required. The table expression can be:
- A table or dictionary (call-by-value)
- A symbol atom representing a table/dictionary name (call-by-name)

```q
update c:b*2 from ([]a:1 2;b:3 4)
select a,b from t
select a,b from `t
update c:b*2 from `:path/to/db
```

## Result and Side Effects

For **select** queries, results are tables or dictionaries.

For **exec** queries, results are lists or dictionaries.

For **update/delete** queries:
- Call-by-value: returns modified table/dictionary
- Call-by-name: modifies in place and returns the name

```q
t1:t2:([]a:1 2;b:3 4)

update a:neg a from t1
a  b
----
-1 3
-2 4
t1~t2
1b

update a:neg a from `t1
`t1
t1~t2
0b
```

## Phrases and Subphrases

Select, By, and Where phrases consist of comma-separated subphrases. Subphrases evaluate left-to-right, with each expression evaluated right-to-left in normal q syntax.

Use parentheses for Join operators in subphrases:

```q
select (id,'4),val from tbl
x   val
-------
1 4 100
1 4 200
2 4 300
2 4 400
2 4 500
```

## Names in Subphrases

Names resolve in order:
1. Column or key names
2. Local names in encapsulating function
3. Global names in working namespace

Dot notation accesses foreign keys:

```q
select sname:s.name, qty from sp
sname qty
---------
smith 300
smith 200
smith 400
smith 200
clark 100
smith 100
jones 300
jones 400
blake 200
clark 200
clark 300
smith 400
```

Duplicate column names trigger an error during parse:

```q
parse"select b by b from t"
'dup names for cols/groups b
  [2]  select b by b from t
       ^
```

## Computed Columns

Create new columns with expressions, optionally named with colons:

```q
t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)

select c1, c3*2 from t
c1 c3
------
a  2.2
b  4.4
c  6.6

select c1, dbl:c3*2 from t
c1 dbl
------
a  2.2
b  4.4
c  6.6
```

Without explicit names, q names columns by their leftmost term or `x`:

```q
select c1, c1, 2*c2, c2+c3, string c3 from t
c1 c11 x  c2   c3
--------------------
a  a   20 11.1 "1.1"
b  b   40 22.2 "2.2"
c  c   60 33.3 "3.3"
```

## Virtual Column `i`

The `i` virtual column represents row index:

```q
select i, c1 from t
x c1
----
0 a
1 b
2 c

select from t where i in 0 2
c1 c2 c3
---------
a  10 1.1
c  30 3.3
```

In partitioned tables, `i` is relative to the partition.

## Where Phrase

Boolean lists select records:

```q
select from t where 101b
c1 c2 c3
---------
a  10 1.1
c  30 3.3
```

Subphrases apply successive filters:

```q
select from t where c2>15,c3<3.0
c1 c2 c3
---------
b  20 2.2

select from t where (c2>15) and c3<3.0
c1 c2 c3
---------
b  20 2.2
```

The first example (successive filters) is more efficient—it only tests `c3` values where `c2>15`. Start Where phrases with most stringent tests.

For partitioned tables, the first subphrase should filter by partition columns to avoid loading all partitions.

Use [`fby`](../../ref/fby/) to filter on groups.

## Aggregates

q aggregates resemble SQL GROUP BY:

```q
select total:sum amt by stock from trade
stock| total
-----| -----
bac  | 1000
ibm  | 2000
usb  | 815
```

The grouping column becomes a key in the result.

## Sorting

qSQL provides no sorting syntax. Use [`xasc`](../../ref/asc/#xasc) and [`xdesc`](../../ref/desc/#xdesc):

```q
sp
s  p  qty
---------
s1 p1 300
s1 p2 200
s1 p3 400
s1 p4 200
s4 p5 100
s1 p6 100
s2 p1 300
s2 p2 400
s3 p2 200
s4 p2 200
s4 p4 300
s1 p5 400

`p xasc `qty xdesc select from sp where p in `p2`p4`p5
s  p  qty
---------
s2 p2 400
s1 p2 200
s3 p2 200
s4 p2 200
s4 p4 300
s1 p4 200
s1 p5 400
s4 p5 100
```

Sorts are stable and can be combined.

## Performance Guidelines

- Select only necessary columns
- Use most restrictive constraints first
- Place suitable attributes on first non-virtual constraint (e.g., `` `p `` or `` `g `` on sym)
- Keep unmodified column names on constraint operator left side
- When aggregating, use virtual field first in By phrase

## Multithreading

This pattern uses secondary threads via `peach` when `sym` has `` `g `` or `` `p `` attributes:

```
select … by sym, … from t where sym in …, …
```

Available since V3.2 2014.05.02.

## Special Functions

These functions receive special treatment in `select`:

```
avg     first   prd
cor     last    sum
count   max     var
cov     med     wavg
dev     min     wsum
```

When wrapped in other functions, q doesn't recognize the need for additional steps:

```q
select sum a from ([]a:1 2 3)
a
-
6

select {(),sum x}a from ([]a:1 2 3)
a
-
6
```

## Cond

[Cond](../../ref/cond/) is not supported inside qSQL expressions:

```q
u:([]a:raze ("ref/";"kb/"),\:/:"abc"; b:til 6)
select from u where a like $[1b;"ref/*";"kb/*"]
'rank
  [0]  select from u where a like $[1b;"ref/*";"kb/*"]
                                  ^
```

Enclose in a lambda:

```q
select from u where a like {$[x;"ref/*";"kb/*"]}1b
a       b
---------
"ref/a" 0
"ref/b" 2
"ref/c" 4
```

Or use Vector Conditional instead.

## Functional SQL

The interpreter translates query templates into functional SQL. Functional forms are more general but less readable. Prefer templates where possible—there is no performance penalty.

## Stored Procedures

Lambdas work in queries:

```q
f:{[x] x+42}
select stock, f amount from trade
stock amount
------------
ibm   542
```

## Parameterized Queries

Evaluate template expressions in lambdas:

```q
myquery:{[tbl; amt] select stock, time from tbl where amount > amt}
myquery[trade; 100]
stock time
------------------
ibm   09:04:59.000
```

Column names cannot be query parameters—use functional qSQL for such cases.
