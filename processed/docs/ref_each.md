# each, peach

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

These iterators "apply a unary function to each item of x and returns a result of the same length." The projections `each[v1;]`, `each[vv;]`, `peach[v1;]`, and `peach[vv;]` are uniform functions.

Both `each` and `peach` perform identical computations and return the same results. The key distinction is that `peach` distributes work across available secondary tasks for parallel execution.

## Examples

```q
q)count each ("the";"quick";" brown";"fox")
3 5 6 3

q)(+\)peach(2 3 4;(5 6;7 8);9 10 11 12)
2 5 9
(5 6;12 14)
9 19 30 42
```

## Notes

- `each` wraps the Each iterator; `peach` wraps the Each Parallel iterator
- `each` is considered redundant with atomic functions
- `peach` applies only unary values; for rank ≥2, use Apply to project as unary
- For matrix `m` with row arguments for `v4`: use `.[v4;]peach m`
- For table `t` with argument columns: use `.[v3;]peach flip t `b`c`a``

## Blocked Operations within peach

The following cannot be used within `peach`:
- Socket operations (hopen, websocket open, broadcast)
- Global variable amendments
- Loading master decryption keys
- System commands affecting global state

File handle usage is permitted if managed to prevent parallel access conflicts. Streaming execute (`-11!`) is allowed but global variable updates are restricted.

## Related

- Maps (binary and higher-rank uses)
- `.Q.fc` (parallel on cut)
- Parallel processing
- Atomic functions
