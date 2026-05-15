# `update` Keyword Documentation

## Overview

The `update` keyword modifies or adds rows and columns to tables or dictionary entries. It functions as a qSQL query template with specialized syntax distinct from standard q.

## Syntax

```
update <select_phrase> [by <by_phrase>] from <table_expression> [where <where_phrase>]
```

This follows the qSQL query template structure.

## Key Sections

### From Phrase

"update will not modify a splayed table on disk." This means disk-based splayed tables remain unchanged when using `update` directly on them.

Since version 4.1t (2021.06.04), updates leveraging splayed table paths utilize parallel processing when secondary threads are configured.

### Select Phrase

Column references in the select phrase denote new or modified table columns:

```q
q)t:([] name:`tom`dick`harry; age:28 29 35)
q)update eye:`blue`brown`green from t
name  age eye
---------------
tom   28  blue
dick  29  brown
harry 35  green
```

### Where Phrase

The where clause restricts which rows receive updates:

```q
q)t:([] name:`tom`dick`harry; hair:`fair`dark`fair; eye:`green`brown`gray)
q)update eye:`blue from t where hair=`fair
name  hair eye
----------------
tom   fair blue
dick  dark brown
harry fair blue
```

New values must match the column's existing type. When adding columns via where clause, unmatched rows contain type-appropriate null values.

### By Phrase

Grouping applies updates across row clusters. Aggregate functions assign aggregated group values uniformly across all group members:

```q
q)update avg weight by city from p
```

Uniform functions apply sequentially within groups, enabling cumulative calculations:

```q
q)update cumqty:sums qty by s from sp
s p  qty cumqty
---------------
0 p1 300 300
0 p2 200 500
```

Since version 4.1 (2024.04.29), dictionary updates containing by clauses throw type errors.

### Cond

Conditional expressions are unsupported within query templates.

## Related Functions

[`delete`](../delete/), [`exec`](../exec/), [`select`](../select/)

[qSQL](../../basics/qsql/), [Functional SQL](../../basics/funsql/)
