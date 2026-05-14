# QSQL Query Templates - Full Content

## QSQL query templates

The four primary query templates in kdb+ are:

```q
select [Lexp] [ps] [by pb] from texp [where pw]
exec [distinct] [ps] [by pb] from texp [where pw]
update ps [by pb] from texp [where pw]
delete from texp [where pw]
delete ps from texp
```

Evaluation order proceeds as: From phrase → Where phrase → By phrase → Select phrase → Limit expression.

## From Phrase

The From phrase `from texp` is mandatory. The table expression may be:

- A table or dictionary (call-by-value)
- A symbol atom referring to an in-memory or on-disk table (call-by-name)

Examples:

```q
update c:b*2 from ([]a:1 2;b:3 4)
select a,b from t
select a,b from `t
update c:b*2 from `:path/to/db
```

## Limit Expressions

Limit expressions constrain results from `select` or `exec` queries. The `distinct` modifier is available for `exec`.

## Result and Side Effects

For `select` queries, results are tables or dictionaries. For `exec`, results are column value lists or dictionaries. When `update` or `delete` use call-by-value tables, they return modified tables. With call-by-name, modifications occur in-place and the table name is returned:

```q
q)t1:t2:([]a:1 2;b:3 4)

q)update a:neg a from t1
a  b
----
-1 3
-2 4
q)t1~t2
1b

q)update a:neg a from `t1
`t1
q)t1~t2
0b
```

## Phrases and Subphrases

Phrases consist of comma-separated subphrases evaluated left-to-right. Each expression evaluates right-to-left using standard q syntax. Use parentheses for Join operators within subphrases:

```q
q)select (id,'4),val from tbl
x   val
-------
1 4 100
1 4 200
2 4 300
2 4 400
2 4 500
```

## Names in Subphrases

Names resolve as: column/key names → local function names → global workspace names. Dot notation accesses foreign keys:

```q
q)select sname:s.name, qty from sp
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

Colliding duplicate column names trigger parse-time errors, resolvable through explicit renaming.

## Computed Columns

Expressions compute new columns; colons name them:

```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)

q)select c1, c3*2 from t
c1 c3
------
a  2.2
b  4.4
c  6.6

q)select c1, dbl:c3*2 from t
c1 dbl
------
a  2.2
b  4.4
c  6.6
```

Unnamed computed columns are auto-named from leftmost terms or `x`. Name conflicts append numeric suffixes:

```q
q)select c1, c1, 2*c2, c2+c3, string c3 from t
c1 c11 x  c2   c3
--------------------
a  a   20 11.1 "1.1"
b  b   40 22.2 "2.2"
c  c   60 33.3 "3.3"
```

## Virtual Column i

The virtual column `i` represents row indices within partitions:

```q
q)select i, c1 from t
x c1
----
0 a
1 b
2 c

q)select from t where i in 0 2
c1 c2 c3
---------
a  10 1.1
c  30 3.3
```

## Where Phrase

Boolean lists select records. Successive subphrases apply sequential filters:

```q
q)select from t where 101b
c1 c2 c3
---------
a  10 1.1
c  30 3.3

q)select from t where c2>15,c3<3.0
c1 c2 c3
---------
b  20 2.2

q)select from t where (c2>15) and c3<3.0
c1 c2 c3
---------
b  20 2.2
```

The first form evaluates sequentially (more efficient), while the second evaluates all conditions before combining. For partitioned tables, the first Where subphrase should filter on partition columns to avoid loading all partitions.

## Aggregates

Aggregate functions within `by` phrases create grouped results:

```q
q)select total:sum amt by stock from trade
stock| total
-----| -----
bac  | 1000
ibm  | 2000
usb  | 815
```

Recognized aggregate functions include: `avg`, `cor`, `count`, `cov`, `dev`, `first`, `last`, `max`, `med`, `min`, `prd`, `sum`, `var`, `wavg`, `wsum`.

## Sorting

The query templates lack sorting provisions. Use `xasc` and `xdesc` post-query:

```q
q)`p xasc `qty xdesc select from sp where p in `p2`p4`p5
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

## Performance Recommendations

- Select only required columns
- Apply most restrictive constraints first
- Use appropriate attributes (`` `p `` or `` `g ``) on first non-virtual constraints
- Keep unmodified column names on constraint operator left sides
- Start By phrases with virtual fields

## Multithreading

Secondary thread usage via `peach` occurs when queries follow this pattern with `` `g `` or `` `p `` attributes:

```q
select … by sym, … from t where sym in …, …
```

For partitioned databases:

```q
select … by sym, … from t where date …, sym in …, …
```

## Special Functions

Functions `avg`, `cor`, `count`, `cov`, `dev`, `first`, `last`, `max`, `med`, `min`, `prd`, `sum`, `var`, `wavg`, `wsum` receive special query treatment. Wrapping them prevents optimization:

```q
q)select sum a from ([]a:1 2 3)
a
-
6
q)select {(),sum x}a from ([]a:1 2 3)
a
-
6
```

## Cond

Cond is unsupported directly in qSQL expressions:

```q
q)select from u where a like $[1b;"ref/*";"kb/*"]
'rank
```

Enclosing in lambdas or using Vector Conditional resolves this:

```q
q)select from u where a like {$[x;"ref/*";"kb/*"]}1b
a       b
---------
"ref/a" 0
"ref/b" 2
"ref/c" 4
```

## Functional SQL

The interpreter translates query templates into functional SQL for evaluation. Functional forms enable complex queries, but templates offer superior readability with no performance penalty.

## Stored Procedures

Lambdas integrate into queries:

```q
q)f:{[x] x+42}
q)select stock, f amount from trade
stock amount
------------
ibm   542
```

## Parameterized Queries

Expressions can parametrize queries via lambdas:

```q
q)myquery:{[tbl; amt] select stock, time from tbl where amount > amt}
q)myquery[trade; 100]
stock time
------------------
ibm   09:04:59.000
```

Column names cannot be parameters in template form; use functional qSQL instead.
