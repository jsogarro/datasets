# xgroup

## Syntax

```q
x xgroup y
xgroup[x;y]
```

## Parameters

- `y`: a table passed by value
- `x`: a symbol atom or vector of column names in `y`

## Returns

Table `y` grouped by the columns specified in `x`. This is equivalent to a `select … by` statement, but all remaining columns are grouped without explicit listing.

## Description

The `xgroup` function groups a table by values in selected columns. Rather than requiring you to enumerate all non-grouped columns in a `select` statement, `xgroup` automatically groups all remaining columns into nested structures.

## Examples

### Basic grouping

```q
q)`a`b xgroup ([]a:0 0 1 1 2;b:`a`a`c`d`e;c:til 5)
a b| c
---| ---
0 a| 0 1
1 c| ,2
1 d| ,3
2 e| ,4
```

### Grouping with enumerated columns

```q
q)\l sp.q
q)meta sp
c  | t f a
---| -----
s  | s s
p  | s p
qty| i

q)`p xgroup sp
p | s               qty
--| -------------------------------
p1| `s$`s1`s2       300 300
p2| `s$`s1`s2`s3`s4 200 400 200 200
p3| `s$,`s1         ,400
p4| `s$`s1`s4       200 300
p5| `s$`s4`s1       100 400
p6| `s$,`s1         ,100
```

### Equivalent select statement

```q
q)select s,qty by p from sp
p | s               qty
--| -------------------------------
p1| `s$`s1`s2       300 300
p2| `s$`s1`s2`s3`s4 200 400 200 200
p3| `s$,`s1         ,400
p4| `s$`s1`s4       200 300
p5| `s$`s4`s1       100 400
p6| `s$,`s1         ,100
```

### Ungrouping results

```q
q)ungroup `p xgroup sp
p  s  qty
---------
p1 s1 300
p1 s2 300
p2 s1 200
p2 s2 400
p2 s3 200
p2 s4 200
p3 s1 400
..
```

## Notes

"Duplicate keys in a dictionary or duplicate column names in a table will cause sorts and grades to return unpredictable results."

## Related

- [`group`](../group/)
- [Dictionaries & tables](../../basics/dictsandtables/)
