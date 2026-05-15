# asof – As-of Join

## Syntax

```q
t asof d     asof[t;d]
```

## Parameters

- `t`: a table
- `d`: a dictionary (or table) with `n` keys (or columns) corresponding to columns in `t`
- the last key (or column) of `d` must correspond to a sortable column in `t` (typically time)

## Description

Returns values from the remaining columns of the last row in `t` where:

- the first `n-1` values match the first `n-1` values of `d`
- the last value does not exceed the last value of `d`

When no matching rows exist, either due to mismatches in the first `n-1` columns or when the final value is smaller than the corresponding value in matching rows, the function returns a dictionary of nulls.

## Example

```q
q)show t:([] time:6#09:00+10*til 3; sym:raze flip 3 2#`AAPL`GOOG; px:6?100f; vol:6?100)
time  sym  px       vol
-----------------------
09:00 AAPL 81.77547 36
09:10 AAPL 75.20102 12
09:20 AAPL 10.86824 97
09:00 GOOG 95.98964 92
09:10 GOOG 3.668341 99
09:20 GOOG 64.30982 45

q)t asof `sym`time!(`AAPL;09:15)
px | 75.20102
vol| 12

q)t asof ([]sym:`GOOG`MSFT; time:09:05)
px       vol
------------
95.98964 92
                  / a row of nulls for no match
```

## Notes

`asof` is a multithreaded primitive.

## Related

- [`aj`](../aj/)
- [`wj`](../wj/)
- [Joins](../../basics/joins/)
