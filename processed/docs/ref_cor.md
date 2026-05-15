# cor – Correlation Coefficient

## Syntax

```q
x cor y    cor[x;y]
```

## Description

The `cor` function calculates the Pearson correlation coefficient between two conforming numeric lists, returning a float value in the range `-1f` to `1f`. Nulls (along with their pairs) are ignored.

## Examples

```q
q)29 10 54 cor 1 3 9
0.7727746

q)10 29 54 cor 1 3 9
0.9795734

q)1 3 9 cor 1 3 9
1f

q)1 3 9 cor neg 1 3 9
-1f

q)1 3 1 3 cor 1 1 3 3
0f

q)1 1 1 cor 1 3 9
0n

q)1 3 0N cor 1 3 9
1f

q)1000101000b cor 0010011001b
-0.08908708
```

## Implementation

The `cor` function is equivalent to `{cov[x;y]%dev[x]*dev y}`, making it expressible through covariance and deviation calculations.

## Performance Characteristics

`cor` is a multithreaded primitive, enabling parallelized execution on multi-core systems.

## Domain and Range

The function accepts numeric types (B, X, H, I, J, E, F, C, P, M, D, Z, N, U, V, T) and produces float outputs. String (S) and general (G) types are not supported.

---

**Category:** Mathematics
