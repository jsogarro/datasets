# where

## Overview

The `where` function returns copies of indexes from a list or keys from a dictionary based on their corresponding values.

## Vector of Non-Negative Integers

For a vector input, `where` returns indices repeated according to their values:

```q
q)where 2 3 0 1
0 0 1 1 1 3
q)raze x #' til count x:2 3 0 1
0 0 1 1 1 3
```

### Boolean Vector Usage

When applied to boolean vectors, `where` identifies positions containing 1s:

```q
q)where 0 1 1 0 1
1 2 4
```

### Practical Examples

Finding even numbers by index:

```q
q)x:1 5 6 8 11 17 20 21
q)where 0 = x mod 2        / indices of even numbers
2 3 6
q)x where 0 = x mod 2      / select even numbers from list
6 8 20
```

## Dictionary with Non-Negative Integer Values

When the input is a dictionary, `where` expands keys according to their values:

```q
q)d:`amr`ibm`msft!2 3 1
q)where d
`amr`amr`ibm`ibm`ibm`msft
q)where 2 3 0 1               / usual operation on integer list
0 0 1 1 1 3
q)where 0 1 2 3 ! 2 3 0 1     / same on dictionary with indices as keys
0 0 1 1 1 3
```

## Related Documentation

- `where` in q-SQL queries
- Selection operations
