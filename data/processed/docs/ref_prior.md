# prior

## Syntax

```q
v2 prior x      prior[v2;x]
(vv) prior x    prior[vv;x]
```

## Parameters

- **v2**: a binary applicable value
- **vv**: a variadic applicable value
- **x**: the input data

## Description

The `prior` function "applies v2 or vv to each item of x and the item preceding it, and returns a result of the same length."

This projection creates uniform functions that operate on successive pairs of elements.

## Examples

```q
q)(+) prior til 10
0 1 3 5 7 9 11 13 15 17

q){x+y%10}prior til 10
0n 1 2.1 3.2 4.3 5.4 6.5 7.6 8.7 9.8
```

## Notes

- `prior` wraps the Each Prior iterator
- The first item's treatment follows the rules of the iterator (see Each Prior documentation)
- Best practice recommends using `prior` rather than the raw iterator except when composing multiple iterators, where brevity may be preferred

## See Also

[Each Prior iterator](../maps/#each-prior)
