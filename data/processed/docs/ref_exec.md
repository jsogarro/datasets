# exec Keyword Reference

## Overview

The `exec` keyword is a qSQL query template that retrieves selected rows and columns from a table. As stated in the documentation, it "Return[s] selected rows and columns from a table."

## Syntax

```
exec [distinct] ps [by pb] from texp [where pw]
```

Where:
- `ps` = select phrase (columns/expressions)
- `pb` = grouping columns
- `texp` = table expression
- `pw` = where conditions

## From Phrase

The table expression may reference tables in memory or on disk (splayed but not partitioned). For partitioned tables, nest the exec inside a select:

```q
exec … from select … from …
```

## Select Phrase Behavior

The return type varies based on the select phrase:

**Omitted phrase** - returns the last record as a dictionary:
```q
exec from sp
s  | `s!0
p  | `p$`p5
qty| 400
```

**Single column** - returns a list:
```q
exec qty from sp
300 200 400 200 100 100 300 400 200 200 300 400
```

**Multiple columns with names** - returns a dictionary:
```q
exec qty, s from sp
qty| 300 200 400 200 100 100 300 400 200 200 300 400
s  | s1  s1  s1  s1  s4  s1  s2  s2  s3  s4  s4  s1
```

**Grouped aggregation** - returns dictionary by grouping key:
```q
exec sum qty by s from sp
s1| 1600
s2| 700
s3| 200
s4| 600
```

**With explicit column assignment**:
```q
exec amount:qty from sp
amount| 300 200 400 200 100 100 300 400 200 200 300 400
```

**Table result** (use `by 0b`):
```q
exec qty, s by 0b from sp
qty s
------
300 s1
200 s1
```

## Key Distinction from select

Unlike `select` queries which always return tables with uniform column lengths, "an `exec` query result is a dictionary, and column lengths can vary":

```q
q)exec name, distinct eye from t
name| `tom`dick`harry`jack`jill
eye | `blue`green`gray
```

This would error in a select query due to length mismatch.

## Limit Expression

The `distinct` keyword applies only to the first item:

```q
exec distinct s,p,s from sp
s | `s$`s1`s4`s2`s3
p | `p$`p1`p2`p3`p4`p5`p6`p1`p2`p2`p2`p4`p5
s1| `s$`s1`s1`s1`s1`s4`s1`s2`s2`s3`s4`s4`s1
```

## Limitations

Cond is not supported inside query templates.

## See Also

- [`delete`](../delete/)
- [`select`](../select/)
- [`update`](../update/)
- [qSQL](../../basics/qsql/)
- [Functional SQL](../../basics/funsql/)
