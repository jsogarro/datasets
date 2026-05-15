# ratios — Ratios Between Items

## Syntax

```q
ratios y     ratios[y]
```

## Description

Where `y` is a non-symbolic sortable list, this function computes ratios of consecutive pairs of items by dividing each element by its predecessor. It functions as an aggregate operation.

"ratios is an aggregate function."

## Use Cases

Common applications include calculating financial returns:

```q
update ret:ratios price by sym from trade
select log ratios price from trade
```

For price movements with sign detection:

```q
update diff:deltas price by sym from trade
select count i by signum deltas price from trade
```

## Implicit Iteration

The function applies to dictionaries and tables:

```q
q)k:`k xkey update k:`abc`def`ghi from t:flip d:`a`b!(10 21 3;4 5 6)

q)ratios d
a| 10  21        3
b| 0.4 0.2380952 2

q)ratios t
a         b
--------------
10        4
2.1       1.25
0.1428571 1.2

q)ratios k
k  | a         b
---| --------------
abc| 10        4
def| 2.1       1.25
ghi| 0.1428571 1.2
```

## First Predecessor

"The predecessor of the first item is 1."

```q
q)ratios 2000 2005 2007 2012 2020
2000 1.0025 1.000998 1.002491 1.003976
```

For alternative behavior placing 1 at the result's start:

```q
q)ratios0:{first[x]%':x}
q)ratios0 2000 2005 2007 2012 2020
1 1.0025 1.000998 1.002491 1.003976
```

## Implementation Notes

The derived function `%':` (Divide Each Prior) underlies this operation. While this operator supports both unary and binary applications, `ratios` functions only as unary; use the derived function for binary cases.

## Related Functions

- Each Prior
- `differ`
- Divide
- Mathematics
