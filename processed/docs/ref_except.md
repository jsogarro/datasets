# except – Exclude items from a list

## Syntax

```q
x except y
except[x;y]
```

## Parameters

- `x`: a list
- `y`: a list or atom

## Description

Returns a list of all items from `x` that are not found in `y`.

The function uses [`in`](../in/) to identify items of `x` present in `y`, which internally uses [`find`](../find/).

## Examples

```q
q)1 2 3 except 2
1 3

q)1 2 3 4 1 3 except 2 3
1 4 1
```

## Related Functions

- [`find`](../find/)
- [`in`](../in/)
- [`within`](../within/)

---

**Topic:** [Selection](../../basics/by-topic/#selection)
