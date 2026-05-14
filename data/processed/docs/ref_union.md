# Union

## Definition

The `union` function computes the union of two lists, returning distinct items from the combined arguments.

**Syntax:**
```q
x union y
union[x;y]
```

## Parameters

- `x`: A list or atom
- `y`: A list or atom

## Return Value

A list containing the distinct items of the combined arguments, equivalent to `distinct x,y`.

## Examples

### Basic List Union

```q
q)1 2 3 3 6 union 2 4 6 8
1 2 3 6 4 8
```

### Equivalent to Distinct Join

```q
q)distinct 1 2 3 3 6, 2 4 6 8
1 2 3 6 4 8
```

### Table Union

```q
q)t0:([]x:2 3 5;y:"abc")
q)t1:([]x:2 4;y:"ad")
q)t0 union t1
x y
---
2 a
3 b
5 c
4 d
```

### Verification

```q
q)(distinct t0,t1)~t0 union t1
1b
```

## Related Functions

- [`in`](../in/) - Membership test
- [`inter`](../inter/) - Intersection of lists
- [`within`](../within/) - Containment check
