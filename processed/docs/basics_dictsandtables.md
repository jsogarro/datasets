# Dictionaries in kdb+/q

## Overview

A dictionary maps keys to values, similar to how a list maps indexes to items. In q, dictionaries are created using the `!` operator to pair a key list with a value list.

```q
d:`tom`dick`harry!1040 59 27
```

This creates:
```
tom  | 1040
dick | 59
harry| 27
```

## Key Concepts

**Construction**: Use the `!` operator with two same-length lists. Keys should be unique to avoid undefined behavior.

```q
d:`a`b`c!1 2 3
```

**Accessing keys and values**:
- `key d` returns the key list
- `value d` returns the value list

**Indexing**: Dictionaries are indexed by their keys, not numeric positions.

```q
dic:`a`b`c`d`e!10 20 30 40 50
dic`d`b        / Returns 40 20
```

**Upsert semantics**: Assigning to a dictionary key updates existing entries or adds new ones.

```q
dic[`x`b]:42 100
```

## Searching

Use `?` (Find) to locate a key by value, or `where` to find all matching keys:

```q
d:`a`b`c`d!10 20 30 10
d?30           / Returns `c
where d=10     / Returns `a`d
```

## Operations

Dictionaries maintain insertion order. You can take/drop from ends or select specific keys:

```q
-2#d           / Last 2 items
`b`d#d         / Select specific keys
`b`x_d         / Drop specific keys
```

**Joining** uses upsert semantics:

```q
(`a`b`c!10 20 30),`c`d!400 500
/ Results in: a|10 b|20 c|400 d|500
```

## Special Cases

**Empty dictionaries** require typed lists: `()!()` or `(`symbol$())!`float$()`

**Column dictionaries** have same-length list values, effectively functioning as transposed tables via `flip`.
