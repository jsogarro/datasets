# inter

_Intersection of two lists or dictionaries_

## Syntax

```q
x inter y
inter[x;y]
```

## Description

Where `x` and `y` are lists or dictionaries, uses the result of `x in y` to return items or entries from `x`. 

**Important note:** This operation is only equivalent to set intersection if the items of `x` are unique. If `x` contains duplicated items that are also found in `y`, they remain duplicated in the result.

## Examples

### Lists

```q
q)1 3 4 2 inter 2 3 5 7 11
3 2
q)1 2 3 1 4 inter 4 1 4
1 1 4
```

### Dictionaries

Returns common values from dictionaries:

```q
q)show x:(`a`b)!(1 2 3;`x`y`z)
a| 1 2 3
b| x y z
q)show y:(`a`b`c)!(1 2 3;2 3 5;`x`y`z)
a| 1 2 3
b| 2 3 5
c| x y z
q)x inter y
1 2 3
x y z
```

### Tables

Returns common rows from simple tables:

```q
q)show x:([]a:`x`y`z`t;b:10 20 30 40)
a b
----
x 10
y 20
z 30
t 40
q)show y:([]a:`y`t`x;b:50 40 10)
a b
----
y 50
t 40
x 10
q)x inter y
a b
----
x 10
t 40
```

## Related Functions

- `in`
- `within`
