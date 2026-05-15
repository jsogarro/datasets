# As-of Join: `aj`, `aj0`, `ajf`, `ajf0`

## Syntax

```q
aj  [c; t1; t2]
aj0 [c; t1; t2]
ajf [c; t1; t2]
ajf0[c; t1; t2]
```

## Parameters

- `t1`: a table or table name as symbol (since 4.1t 2023.08.04, updates in place when symbol)
- `t2`: a simple table
- `c`: symbol vector of `n` column names common to both tables with matching types
- `c[n]`: must be of sortable type (typically time)

## Functionality

Returns a table containing records from a left-join of `t1` and `t2`. The join matches columns `c[0]...c[n-1]` for equality and selects the last value of `c[n]` (most recent time). For each `t1` record:

- matching records in `t2` append their last (by row order) matching record's items
- non-matching records get null values in remaining columns

## Basic Example

```q
q)t:([]time:10:01:01 10:01:03 10:01:04;sym:`msft`ibm`ge;qty:100 200 150)
q)t
time       sym  qty
-----------------
10:01:01 msft 100
10:01:03 ibm  200
10:01:04 ge   150

q)q:([]time:10:01:00 10:01:00 10:01:00 10:01:02;sym:`ibm`msft`msft`ibm;px:100 99 101 98)
q)q
time     sym  px 
-----------------
10:01:00 ibm  100
10:01:00 msft 99 
10:01:00 msft 101
10:01:02 ibm  98 

q)aj[`sym`time;t;q]
time       sym  qty px
---------------------
10:01:01 msft 100 101
10:01:03 ibm  200 98
10:01:04 ge   150
```

## Variant Differences

**`aj` vs `aj0`**: Different time values returned—`aj` uses boundary time from `t1`; `aj0` uses actual time from `t2`.

**`ajf` and `ajf0`** (since V3.6 2018.05.18): Fill from `t1` if corresponding `t2` value is null.

```q
q)t0:([]time:2#00:00:01;sym:`a`b;p:1 1;n:`r`s)
q)t1:([]time:2#00:00:01;sym:`a`b;p:0 1)
q)t2:([]time:2#00:00:00;sym:`a`b;p:1 0N;n:`r`s)
q)t0~ajf[`sym`time;t1;t2]
1b
```

## Key Notes

- `aj` is a multithreaded primitive
- No requirement for join columns to be keys, but performance improves with keys
- Ensure search columns argument is in correct order (e.g., `` `sym`time ``)—incorrect ordering causes severe performance degradation
- Expected throughput: million or two trade records per second

## Performance Optimization

**Column Ordering**: First argument columns must follow pattern like `` `sym`time ``.

**Attribute Requirements** (on disk):

| Medium | `t2[c1]` | `t2[c2…]` | Example |
|--------|----------|----------|---------|
| disk | `p#` | sorted within `c1` | `quote` has `` `p#sym `` and `time` sorted within `sym` |

Note: on disk, `g#` attribute provides no help.

## Usage Recommendations

**In Memory**: Select from `t2` is unnecessary:

```q
aj[`sym`time;select … from trade where …;quote]
```

**On Disk (Splayed)**:

```q
aj[`sym`time;select … from trade where …;select … from quote]
```

**On Disk (Partitioned)**:

```q
aj[`sym`time;select … from trade where …;
             select … from quote where date = …]
```

Only select the virtual partition column if needed—construction on demand is slow for large partitions.

## Related

- [`asof`](../asof/)
- [Joins](../../basics/joins/)
- _Q for Mortals_ §9.9.8
