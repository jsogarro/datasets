# Median of a Numeric List

## `med`

_Median_

```q
med x    med[x]
```

Where `x` is a numeric list, returns its median value.

### Examples

```q
q)med 10 34 23 123 5 56
28.5
q)select med price by sym from trade where date=2001.10.10,sym in`AAPL`LEH
```

`med` is an aggregate function, equivalent to:

```q
{avg x (iasc x)@floor .5*-1 0+count x,:()}
```

## Domain and Range

| Domain | b g x h i j e f c s p m d z n u v t |
|--------|---------------------------------------|
| Range  | f . f f f f f f f . f f f f f f f f   |

## Implicit Iteration

`med` applies to dictionaries and tables.

```q
q)k:`k xkey update k:`abc`def`ghi from t:flip d:`a`b!(10 -21 3;4 5 -6)

q)med d
7 -8 -1.5

q)med t
a| 3
b| -6

q)med k
a| 3
b| -6
```

## Partitions and Segments

`med` signals a part error when running a median over partitions or segments. This prevents returning median of medians, which should instead be explicitly coded as a cascading select:

```q
select med price by sym from 
  select price, sym from trade 
    where 
      date within 2001.10.10 2001.10.11, 
      sym in `AAPL`LEH
```
