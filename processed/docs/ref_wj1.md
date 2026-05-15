# Window Join (wj, wj1)

## Syntax

```q
wj [w; c; t; (q; (f0;c0); (f1;c1))]
wj1[w; c; t; (q; (f0;c0); (f1;c1))]
```

## Parameters

- **t** and **q**: Simple tables to join (q should be sorted by `sym` and `time` with `p#` attribute on sym). Since v4.1, if t is a table name, it updates in place.
- **w**: Pair of time/timestamp lists representing interval begin and end
- **c**: Names of common columns (syms and times with integral types)
- **f0, f1**: Aggregation functions applied to q columns c0, c1 over intervals

## Description

Returns one record per row in table `t`, with additional columns containing aggregation results from matching intervals in `w`. A quote persists until the next quote.

To view all values in each window, use the identity function `::` instead of aggregates:

```q
wj[w;c;t;(q;(::;c0);(::;c1))]
```

## Multi-column Arguments

Since v3.6 (2018.12.24), wj and wj1 support multi-column arguments, forming result column names from the last argument:

```q
wj[w; f; t; (q; (wavg;`asize;`ask); (wavg;`bsize;`bid))]
```

## Interval Behavior

Both `wj` and `wj1` use closed intervals `[]` (quotes ≥ beginning and ≤ end).

**wj**: Includes the prevailing quote at window entry (quotes are step functions).

**wj1**: Includes quotes from window entry onward. Use this when considering quotes arriving from interval beginning.

| Version | wj1 | wj |
|---------|-----|-----|
| 3.0+ | `[]` | prevailing + `[]` |
| 2.7/2.8 | `[)` | prevailing + `[]` |

## Example

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

Viewing all values in windows:

```q
q)wj[w;f;t;(q;(::;`ask);(::;`bid))]
sym time     price ask             bid
--------------------------------------------------
ibm 10:01:01 100   101 103         98 99
ibm 10:01:04 101   103 103 104 104 99 102 103 103
ibm 10:01:08 105   107 108 107 108 104 106 106 107
```

## Notes

Window joins with multiple symbols require `p#sym` schema. Using typical RTD-like `g#` attribute produces undefined results.

Window join generalizes as-of join: as-of takes a snapshot of current state; window join aggregates values within intervals.

## Related

[`aj`](../aj/), [`asof`](../asof/), [Joins](../../basics/joins/), Q for Mortals 9.9.9
