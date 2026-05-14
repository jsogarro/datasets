# Inner Join: `ij` and `ijf`

## Syntax

```q
x ij y      ij[x;y]
x ijf y     ijf[x;y]
```

## Parameters

- `x` and `y` are tables
- `y` must be keyed with key columns present in `x`

## Description

The inner join operation combines two tables on the key columns of the second table, returning one combined record for each row in `x` matching a row in `y`.

## Example Usage

```q
q)t
sym  price
---------------
IBM  0.7029677
FDP  0.08378167
FDP  0.06046216
FDP  0.658985
IBM  0.2608152
MSFT 0.5433888

q)s
sym | ex  MC
----| --------
IBM | N   1000
MSFT| CME 250

q)t ij s
sym  price     ex  MC
-----------------------
IBM  0.7029677 N   1000
IBM  0.2608152 N   1000
MSFT 0.5433888 CME 250
```

## Handling Common Columns

Common columns are replaced with values from `y`:

```q
q)([] k:1 2 3 4; v:10 20 30 40) ij ([k:2 3 4 5]; v:200 300 400 500;s:`a`b`c`d)
k v   s
-------
2 200 a
3 300 b
4 400 c
```

## Version Notes

**V3.0 Changes:** When `y` contains nulls, `ij` uses the null value from `y`, unlike the V2.8 behavior which preserved the corresponding value from `x`. The earlier behavior is available as `ijf` in V3.4 and later.

## Performance

`ij` is implemented as a multithreaded primitive for enhanced performance.
