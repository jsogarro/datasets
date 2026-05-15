# raze

## Overview

`raze` joins the items of a list, collapsing one level of nesting.

**Syntax:**
```q
raze x
raze[x]
```

## Description

The function returns items from `x` joined together, removing one layer of nesting. To collapse all nesting levels, use the Converge pattern: `raze/[x]`.

## Examples

### Basic usage with nested lists

```q
q)raze (1 2;3 4 5)
1 2 3 4 5
```

### Single-level flattening

```q
q)b:(1 2;(3 4;5 6);7;8)
q)raze b                 / flatten one level
1
2
3 4
5 6
7
8
```

### Complete flattening with Converge

```q
q)raze/[b]               / flatten all levels
1 2 3 4 5 6 7 8
```

### Atoms become single-item lists

```q
q)raze 42                / atom returned as a list
,42
```

### Dictionary values

```q
q)d:`q`w`e!(1 2;3 4;5 6)
q)value d
1 2
3 4
5 6
q)raze d
1 2 3 4 5 6
```

## Important Notes

`raze` is equivalent to the Join Over operator `,/` and requires that items can be joined together. Attempting to join incompatible types raises an error:

```q
q)d:`a`b!(1 2;3 5)
q)10,d          / cannot join integer and dictionary
'type
q)raze (10;d)   / raze will not work
'type
```

## Related

- [Join operator](../join/)
