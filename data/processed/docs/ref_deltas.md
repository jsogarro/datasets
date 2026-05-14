# deltas – Differences Between Adjacent List Items

## Overview

The `deltas` function computes differences between consecutive pairs in a numeric or temporal vector, with the first item of the result being the first item of the input.

**Syntax:**
```q
deltas x    deltas[x]
```

**Domain and Range:**
```
domain: B G X H I J E F C S P M D Z N U V T
range:  i . i i i j e f . . n i i f n u v t
```

## Basic Examples

```q
q)deltas 1 4 9 16
1 3 5 7
```

```q
q)t:([]time:2020.01.01D09:00:00+1000*til 6; sym:`GOOG`AAPL`AAPL`GOOG`AAPL`GOOG; price:51 54 54 52 53 53)
q)show t:update diff:deltas price by sym from t
time                          sym  price diff
---------------------------------------------
2020.01.01D09:00:00.000000000 GOOG 51    51
2020.01.01D09:00:00.000001000 AAPL 54    54
2020.01.01D09:00:00.000002000 AAPL 54    0
2020.01.01D09:00:00.000003000 GOOG 52    1
2020.01.01D09:00:00.000004000 AAPL 53    -1
2020.01.01D09:00:00.000005000 GOOG 53    1
```

## Using with signum

Combine `deltas` with `signum` to identify price movement direction:

```q
q)select movement:signum deltas price by sym from t
sym | movement
----| --------
AAPL| 1 0 -1
GOOG| 1 1 1

q)select movement:1_ signum deltas price by sym from t
sym | movement
----| --------
AAPL| 0 -1
GOOG| 1 1

q)ungroup select movement:1_ signum deltas price by sym from t
sym  movement
-------------
AAPL 0
AAPL -1
GOOG 1
GOOG 1

q)select count i by sym, movement from ungroup select movement:1_ signum deltas price by sym from t
sym  movement| x
-------------| -
AAPL -1      | 1
AAPL 0       | 1
GOOG 1       | 2
```

## First Predecessor

The predecessor of the first item defaults to 0:

```q
q)deltas 2000 2005 2007 2012 2020
2000 5 2 5 8
```

To return 0 as the first item instead:

```q
q)deltas0:{first[x]-':x}
q)deltas0 2000 2005 2007 2012 2020
0 5 2 5 8
```

## Implementation Note

The derived function "Subtract Each Prior" (`-':`) implements `deltas` behavior. While this derived function supports both unary and binary application, `deltas` is supported only as unary; use the derived function directly for binary operations.
