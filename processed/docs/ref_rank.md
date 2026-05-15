# rank

## Syntax

```q
rank x
rank[x]
```

## Description

The `rank` function returns the position of each item in its argument within a sorted list or dictionary.

"Where `x` is a list or dictionary, returns for each item in `x` the index of where it would occur in the sorted list or dictionary."

This operation is equivalent to applying `iasc` twice to the input.

## Examples

```q
q)rank 2 7 3 2 5
0 4 2 1 3
```

```q
q)iasc 2 7 3 2 5
0 3 2 4 1
```

```q
q)iasc iasc 2 7 3 2 5            / same as rank
0 4 2 1 3
```

```q
q)asc[2 7 3 2 5] rank 2 7 3 2 5  / identity
2 7 3 2 5
```

```q
q)iasc idesc 2 7 3 2 5           / descending rank
3 0 2 4 1
```

## Related

- [`iasc`](../asc/#iasc) – ascending sort indices
- [Sorting](../../basics/by-topic/#sort)
