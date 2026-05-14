# Window Join (`wj`, `wj1`)

## Syntax

```q
wj [w; c; t; (q; (f0;c0); (f1;c1))]
wj1[w; c; t; (q; (f0;c0); (f1;c1))]
```

## Parameters

- **`t` and `q`**: Simple tables to be joined. The `q` table should be sorted by `` `sym`time `` with `` `p# `` attribute on sym. Since version 4.1t 2023.08.04, if `t` is a table name, it updates in place.
- **`w`**: Pair of lists representing time/timestamp intervals (begin and end)
- **`c`**: Names of common columns (typically syms and times, must be integral types)
- **`f0`, `f1`**: Aggregation functions applied to columns `c0` and `c1` from table `q` over specified intervals

## Returns

For each record in `t`, returns a new record with additional columns containing aggregation results computed over matching intervals in `w`.

## Description

A window join aggregates values from a quote table over time intervals, typically used for matching trades with quotes. "A quote is understood to be in existence until the next quote."

To inspect all values within each window, use the identity function `::` instead of aggregates:

```q
wj[w;c;t;(q;(::;c0);(::;c1))]
```

## Multi-column Arguments

Since version 3.6 2018.12.24, both functions support multi-column arguments, forming result column names from the final argument:

```q
wj[w; f; t; (q; (wavg;`asize;`ask); (wavg;`bsize;`bid))]
```

## Interval Behavior

Both `wj` and `wj1` use closed intervals `[]`, considering quotes ≥ beginning and ≤ end of the interval.

**`wj`**: Includes the prevailing quote on entry plus all subsequent quotes in the interval.

**`wj1`**: Considers only quotes from the window start onward. Use `wj1` when joins should include quotes arriving from the interval beginning.

### Version Differences

| Version | `wj1`    | `wj`             |
|---------|----------|------------------|
| 3.0+    | `[]`     | prevailing + `[]` |
| 2.7/2.8 | `[)`     | prevailing + `[]` |

## Examples

```q
q)t:([]sym:3#`ibm;time:10:01:01 10:01:04 10:01:08;price:100 101 105)
q)t
sym time     price
------------------
ibm 10:01:01 100
ibm 10:01:04 101
ibm 10:01:08 105

q)a:101 103 103 104 104 107 108 107 108
q)b:98 99 102 103 103 104 106 106 107
q)q:([]sym:`ibm; time:10:01:01+til 9; ask:a; bid:b)
q)q
sym time     ask bid
--------------------
ibm 10:01:01 101 98
ibm 10:01:02 103 99
ibm 10:01:03 103 102
ibm 10:01:04 104 103
ibm 10:01:05 104 103
ibm 10:01:06 107 104
ibm 10:01:07 108 106
ibm 10:01:08 107 106
ibm 10:01:09 108 107

q)f:`sym`time
q)w:-2 1+\:t.time

q)wj[w;f;t;(q;(max;`ask);(min;`bid))]
sym time     price ask bid
--------------------------
ibm 10:01:01 100   103 98
ibm 10:01:04 101   104 99
ibm 10:01:08 105   108 104
```

### Viewing All Interval Values

```q
q)wj[w;f;t;(q;(::;`ask);(::;`bid))]
sym time     price ask             bid
--------------------------------------------------
ibm 10:01:01 100   101 103         98 99
ibm 10:01:04 101   103 103 104 104 99 102 103 103
ibm 10:01:08 105   107 108 107 108 104 106 106 107
```

## Notes

- Window joins with multiple symbols require `` `p#sym `` schema. Typical RTD-like `` `g# `` attributes produce undefined results.
- Window join generalizes as-of join: an as-of join captures current state, while a window join aggregates values within intervals.

## Related

- [`aj`](../aj/), [`asof`](../asof/)
- [Joins documentation](../../basics/joins/)
