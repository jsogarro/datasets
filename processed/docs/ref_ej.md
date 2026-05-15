# ej – Equi Join

## Syntax

```q
ej[c;t1;t2]
```

## Parameters

- `c`: A list of column names (or a single column name) to join on
- `t1` and `t2`: Tables to be joined

## Description

The equi join operation combines two tables based on matching values in specified columns. It returns one combined record for each row in `t2` that matches `t1` on the join columns.

## Examples

### Basic Equi Join

```q
q)t:([]sym:`IBM`FDP`FDP`FDP`IBM`MSFT;price:0.7029677 0.08378167 0.06046216 
    0.658985 0.2608152 0.5433888)
q)s:([]sym:`IBM`MSFT;ex:`N`CME;MC:1000 250)

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
sym  ex  MC
-------------
IBM  N   1000
MSFT CME 250

q)ej[`sym;s;t]
sym  ex  MC    price
-----------------------
IBM  N   1000  0.7029677
IBM  N   1000  0.2608152 
MSFT CME  250  0.5433888
```

### Handling Duplicate Values

When duplicate column values exist, values from `t2` are repeated in the result:

```q
q)t1:([] k:1 2 3 4; c:10 20 30 40)
q)t2:([] k:2 2 3 4 5; c:200 222 300 400 500; v:2.2 22.22 3.3 4.4 5.5)

q)ej[`k;t1;t2]
k c   v
-----------
2 200 2.2
2 222 22.22
3 300 3.3
4 400 4.4
```

## Related Topics

- [Joins](../../basics/joins/)
- _Q for Mortals_ §9.9.5 Equi Join
