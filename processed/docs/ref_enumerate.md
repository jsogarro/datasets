# Enumerate (`$`) Operator

## Syntax

```q
x$y    $[x;y]
```

## Parameters

- `x`: symbol containing the name of a global variable `d`
- `d`: a list (the enumeration domain)
- `y`: a list (values to enumerate)
- `d~distinct d`: domain must have unique items
- All items in `y` must exist in `d`

## Description

"returns `y` as an enumeration of `d`, using `x` as the name of the enumeration domain." Values are stored as indices, consuming less memory.

## Example

```q
q)d:`a`b`c
q)y:`a`b`c`b`a`b`c`c`c`c`c`c`c
q)show e:`d$y;
`d$`a`b`c`b`a`b`c`c`c`c`c`c`c

q)"i"$e
0 1 2 1 0 1 2 2 2 2 2 2 2i
```

Modifying the domain affects all corresponding enumerated values:

```q
q)d[0]:`o
q)e
`d$`o`b`c`b`o`b`c`c`c`c`c`c`c
q)"i"$e
0 1 2 1 0 1 2 2 2 2 2 2 2i
```

## Retrieving Components

```q
q)key e
`d
q)value e
`o`b`c`b`o`b`c`c`c`c`c`c`c
```

## Domain Requirements

"all items of `y` are all items of `d`" — every value must exist in the domain. Missing values trigger a cast error.

```q
q)y:`a`b`c`b`a`b`c`c`c`c`c`c`c
q)x:`a`b
q)`x$y
'cast
```

Use `?` (Enum Extend) to expand the domain instead.

## Errors

| Error | Cause |
|-------|-------|
| cast | item/s of `y` not in `d` |

## See Also

[Enum Extend](../enum-extend/), [Enumeration](../enumeration/), [`$` dollar](../overloads/#dollar)
