# reverse

## Overview

The `reverse` function inverts the order of items in a list or dictionary.

## Syntax

```q
reverse x
reverse[x]
```

## Description

"Returns the items of `x` in reverse order." The function operates on various data structures with different behaviors:

- **Lists**: Items are reordered from last to first
- **Atoms**: Returns the atom unchanged
- **Dictionaries**: Keys are reversed
- **Tables**: Columns are reversed

## Examples

### Basic list reversal

```q
q)reverse 1 2 3 4
4 3 2 1
```

### Dictionary operations

```q
q)d:`a`b!(1 2 3;"xyz")
q)reverse d
b| x y z
a| 1 2 3
```

Reversing each value in the dictionary:

```q
q)reverse each d
a| 3 2 1
b| z y x
```

### Table reversal

```q
q)reverse flip d
a b
---
3 z
2 y
1 x
```

## Related Functions

- [`rotate`](../rotate/) — Rotates items by a specified amount
