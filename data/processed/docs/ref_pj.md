# pj – Plus Join

## Syntax

```q
x pj y     pj[x;y]
```

## Parameters

- `x` and `y` are tables. Since 4.1t 2023.08.04, if `x` is a table name, it updates in place.
- `y` must be keyed
- The key columns of `y` must exist in `x`

## Returns

A table combining `x` and `y` joined on the key columns of `y`. The operation adds matching records from `y` to `x` by summing common columns (excluding key columns). These common columns require appropriate types for addition.

## Behavior

For each record in `x`:

- If a matching record exists in `y`, it adds the values to the `x` record
- If no match exists, common columns remain unchanged; new columns contain zeros

## Example

```q
q)show x:([]a:1 2 3;b:`x`y`z;c:10 20 30)
a b c
------
1 x 10
2 y 20
3 z 30

q)show y:([a:1 3;b:`x`z]c:1 2;d:10 20)
a b| c d
---| ----
1 x| 1 10
3 z| 2 20

q)x pj y
a b c  d
---------
1 x 11 10
2 y 20 0
3 z 32 20
```

## Equivalence

The `pj` operation is equivalent to `x+0^y[`a`b#x]`, which retrieves matching `y` values on the specified columns, fills nulls with zeros, and adds to `x`.
