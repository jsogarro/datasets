# `each`, `peach` - Iterate a Unary

## Syntax

```q
v1 each x   each[v1;x]       v1 peach x   peach[v1;x]  
(vv)each x   each[vv;x]      (vv)peach x   peach[vv;x]
```

## Parameters

- `v1`: a unary applicable value
- `vv`: a variadic applicable value
- `x`: the data to iterate over

## Description

Applies `v1` or `vv` as a unary function to each item of `x`, returning a result of the same length. The projections `each[v1;]`, `each[vv;]`, `peach[v1;]`, and `peach[vv;]` are uniform functions.

Both `each` and `peach` perform identical computations and return the same result. The distinction is that `peach` distributes work across available secondary tasks for parallel execution (see Parallel processing documentation).

`each` wraps the Each iterator; `peach` wraps the Each Parallel iterator. Using these forms is considered good q style for unary values.

## Examples

```q
q)count each ("the";"quick";" brown";"fox")
3 5 6 3

q)(+\)peach(2 3 4;(5 6;7 8);9 10 11 12)
2 5 9
(5 6;12 14)
9 19 30 42
```

## Higher-rank Values

`peach` accepts only unary values. For functions with rank ≥2, employ Apply to project the function as unary:

- Row-wise: `.[v4;]peach m` applies a 4-argument function to each row of a matrix
- Table columns: ``.[v3;]peach flip t `b`c`a`` applies a 3-argument function to specified column arguments in each row

## Blocked Operations within `peach`

The following cannot be used within `peach`:

- `hopen` (socket operations)
- `websocket open`
- `socket broadcast` (via `25!x`)
- Amending global variables
- Loading master decryption key (`-36!`)
- Any system command affecting global state

**Socket usage restrictions:** Sockets within `peach` are blocked (signaling `nosocket` error) unless wrapped in one-shot sync requests or HTTP client requests.

**File handling:** Safe when parallel file access is managed—avoid concurrent use of identical handles across threads.

**Streaming execute** (`-11!`) is permitted but limited by the prohibition on global variable updates.
