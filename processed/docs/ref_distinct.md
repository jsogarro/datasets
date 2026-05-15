# distinct – Unique Items of a List

## Syntax

```q
distinct x
distinct[x]
```

## Description

The `distinct` function returns the unique items from a list `x` in the order they first appear. The result does not have the unique attribute set.

For lists:
```q
q)distinct 2 3 7 3 5 3
2 3 7 5
```

For tables, it returns distinct rows:
```q
q)distinct flip `a`b`c!(1 2 1;2 3 2;"aba")
a b c
-----
1 2 a
2 3 b
```

## Important Notes

- The function does not apply comparison tolerance:
```q
q)\P 14
q)distinct 2 + 0f,10 xexp -13
2 2.0000000000001
```

- `distinct` is a multithreaded primitive

## Domain and Range

```
domain: B G X H I J E F C S P M D Z N U V T
range:  B G X H I J E F C S P M D Z N U V T
```

## Errors

| Error | Cause |
|-------|-------|
| `type` | `x` is an atom |

## See Also

- `.Q.fu` (apply unique)
- Precision documentation
- Search functionality
