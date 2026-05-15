# As-of Join: `aj`, `aj0`, `ajf`, `ajf0`

## Syntax

```q
aj  [c; t1; t2]
aj0 [c; t1; t2]
ajf [c; t1; t2]
ajf0[c; t1; t2]
```

## Parameters

- **`t1`**: A table or table name as symbol (since 4.1t 2023.08.04, updates in place when passed as symbol)
- **`t2`**: A simple table
- **`c`**: Symbol vector of `n` column names common to both tables with matching types
- **`c[n]`**: Must be a sortable type (typically time)

## Behavior

Returns a table from the left-join of `t1` and `t2`. The join matches columns `c[0]...c[n-1]` for equality and takes the last value of `c[n]` (most recent time). For each record in `t1`:

- If matching records exist in `t2`, appends items from the last matching record
- Otherwise, remaining columns are null

## Example

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

| Function | Behavior |
|----------|----------|
| `aj` | Returns boundary time from `t1` |
| `aj0` | Returns actual time from `t2` |
| `ajf` | Fills from `t1` if corresponding `t2` value is null (V3.6+) |
| `ajf0` | Fills from `t1` if corresponding `t2` value is null, using actual time from `t2` |

### Fill Example

```q
q)t0:([]time:2#00:00:01;sym:`a`b;p:1 1;n:`r`s)
q)t1:([]time:2#00:00:01;sym:`a`b;p:0 1)
q)t2:([]time:2#00:00:00;sym:`a`b;p:1 0N;n:`r`s)
q)t0~ajf[`sym`time;t1;t2]
1b
```

## Performance Considerations

**Column Order**: Ensure the first argument lists columns in correct order (e.g., `` `sym`time ``). Incorrect ordering causes severe performance degradation.

**Expected throughput**: One to two million trade records per second.

**Optimal configuration**:

| Storage | First Column | Additional Columns |
|---------|--------------|-------------------|
| Memory | `` `g#`` attribute | Sorted within first column |
| Disk | `` `p#`` attribute | Sorted within first column |

**Notes**:
- The `g#` attribute provides no benefit on disk
- Virtual partition columns should only be selected when necessary (constructed on demand, slow for large partitions)

## Query Optimization

**In memory**: No need to select from `t2`. Use:
```q
aj[`sym`time;select … from trade where …;quote]
```

Instead of:
```q
aj[`sym`time;select … from trade where …;select … from quote where …]
```

**On disk (splayed)**:
```q
aj[`sym`time;select … from trade where …;select … from quote]
```

**On disk (partitioned)**:
```q
aj[`sym`time;select … from trade where …;select … from quote where date = …]
```

## Additional Notes

- `aj` is a multithreaded primitive
- No requirement for join columns to be keys, but keys improve performance
- Additional `where` constraints cause column copying rather than mapping, degrading performance
- When data spans multiple partitions, the `` `p#`` attribute may be lost; apply additional constraints or reapply the attribute
