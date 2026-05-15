# desc, idesc, xdesc – Descending Sorts

## desc

**Descending sort**

```q
desc x    desc[x]
```

Where `x` is a:

- **vector**: returns its items in descending order of value
- **mixed list**: returns items sorted descending by datatype, then descending within datatype
- **nested list**: returns items sorted descending lexicographically
- **dictionary**: returns it sorted by the values
- **table**: returns it sorted descending lexicographically by the non-key columns

Q selects from various sorting algorithms based on datatype and data distribution.

Unlike `asc`, which sets the sorted (or parted) attribute, `desc` sets none, as there is no attribute indicating a descending sort.

**Examples:**

```q
q)desc 2 1 3 4 2 1 2                       / vector
4 3 2 2 2 1 1

q)show l:desc (1;1b;"b";2009.01.01;"a";0)  / mixed list
2009.01.01
"b"
"a"
1
0
q)type each l
-14 -10 -10 -7 -7 -1h                      / datatypes sorted by type number

q)desc `a`b`c!2 1 3                        / dictionary
c| 3
a| 2
b| 1

q)desc([]a:4 4 1;b:`a`d`s)                 / table
a b
---
4 d
4 a
1 s

q)meta desc([]a:3 4 1;b:`a`d`s)
c| t f a
-| -----
a| j
b| s
```

**Domain and Range:** B G X H I J E F C S P M D Z N U V T

## idesc

**Descending grade**

```q
idesc x    idesc[x]
```

Where `x` is a list or dictionary, returns the indices needed to sort it in descending order.

**Examples:**

```q
q)L:2 1 3 4 2 1 2
q)idesc L
3 2 0 4 6 1 5
q)L idesc L
4 3 2 2 2 1 1
q)(desc L)~L idesc L
1b
q)idesc `a`c`b!1 2 3
`b`c`a
```

**Domain and Range:** J J J J J J J J J J J J J J J J J J

## xdesc

**Sorts a table in descending order of specified columns.**

```q
x xdesc y    xdesc[x;y]
```

Where `x` is a symbol vector of column names defined in `y`, which is passed by:

- value: returns sorted result
- reference: updates `y` sorted in descending order by `x`

The sorted attribute is not set. The sort is stable, preserving order amongst equals.

**Examples:**

```q
q)show t:0N?([]sym:raze 2#/:`a`b`c; date:6#2025.01.01+til 2; val:50+6?10f)
sym date       val
-----------------------
c   2025.01.01 51.95847
a   2025.01.02 53.40721
b   2025.01.01 50.54001
b   2025.01.02 55.49794
a   2025.01.01 53.83946
c   2025.01.02 55.61526

q)`date xdesc t
sym date       val
-----------------------
a   2025.01.02 53.40721
b   2025.01.02 55.49794
c   2025.01.02 55.61526
c   2025.01.01 51.95847
b   2025.01.01 50.54001
a   2025.01.01 53.83946

q)`sym`date xdesc t
sym date       val
-----------------------
c   2025.01.02 55.61526
c   2025.01.01 51.95847
b   2025.01.02 55.49794
b   2025.01.01 50.54001
a   2025.01.02 53.40721
a   2025.01.01 53.83946

q)`sym`date xdesc `t
`t

q)meta t                      / no attribute set
c   | t f a
----| -----
sym | s
date| d
val | f
```

### Duplicate Column Names

`xdesc` signals `'dup` and the duplicate column name if it finds duplicate columns in the right argument (since V3.6 2019.02.19).

### Sorting Data on Disk

`xdesc` can sort data on disk directly without loading the entire table into memory (see `xasc`).

**Note:** Duplicate keys in a dictionary or duplicate column names in a table will cause sorts and grades to return unpredictable results.
