# hsym

## Overview

The `hsym` function converts a symbol into a file or process symbol by prefixing it with a colon if not already present.

## Syntax

```q
hsym x     hsym[x]
```

## Parameters

- `x`: A symbol atom or vector (supported since V3.1)

## Return Value

Returns the symbol(s) prefixed with a colon if not already beginning with one.

## Examples

```q
q)hsym`c:/q/test.txt                / file path to symbolic file handle
`:c:/q/test.txt

q)hsym`10.43.23.197                 / IP address to symbolic handle
`:10.43.23.197

q)hsym `host:port`localhost:8001    / hostname to symbolic handle
`:host:port`:localhost:8001

q)hsym `abc`:def`::ghi
`:abc`:def`::ghi
```

## Related Functions

- [`hopen`](../hopen/)

## See Also

- [File system](../../basics/files/)
- [Interprocess communication](../../basics/ipc/)
