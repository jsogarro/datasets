# first, last

## first

_First item of a list_

```q
first x    first[x]
```

Returns the first item of a list or dictionary; otherwise returns `x` unchanged.

Commonly used with [Each](../maps/#each) to retrieve the first item from each element in a list or each value in a dictionary.

### Examples

```q
q)first 1 2 3 4 5
1
q)first 42
42
q)RaggedArray:(1 2 3;4 5;6 7 8 9;0)
q)first each RaggedArray
1 4 6 0
q)RaggedDict:`a`b`c!(1 2;3 4 5;"hello")
q)first RaggedDict  / value of first key
1 2
q)first each RaggedDict
a| 1
b| 3
c| "h"
```

Returns the first row when applied to a table:

```q
q)\l sp.q
q)first sp
s  | `s$`s1
p  | `p$`p1
qty| 300
```

`first` serves as the dual to [`enlist`](../enlist/):

```q
q)a:10
q)a~first enlist 10
1b
q)a~first first enlist enlist 10
1b
```

`first` qualifies as an aggregate function.

## last

_Last item of a list_

```q
last x    last[x]
```

Returns the last item of a list or dictionary; otherwise returns `x` unchanged.

### Examples

```q
q)last til 10
9
q)last `a`b`c!1 2 3
3
q)last 42
42
```
