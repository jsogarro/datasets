# value

## Syntax

```q
value x
value[x]
```

## Description

Returns the value of `x` by recursing the interpreter. The behavior depends on the input type:

- **Dictionary**: Returns the dictionary values
- **Symbol atom**: Returns the value of the variable it names
- **Enumeration**: Returns corresponding symbol vector
- **String**: Result of evaluating it in current context
- **List**: Result of calling or indexing the first element with remaining elements (if first element is string/symbol, it is evaluated first)
- **Projection**: Returns function and arguments
- **Composition**: Returns list of composed values
- **Derived function**: Returns argument of iterator operator
- **Internal code**: Returns internal representation
- **View**: Returns metadata list
- **Lambda**: Returns structure information
- **File symbol**: Returns content of datafile

## Examples

### Dictionary
```q
q)value `q`w`e!(1 2;3 4;5 6)
1 2
3 4
5 6
```

### Symbol (Variable)
```q
q)a:1 2 3
q)value `a
1 2 3
```

### Enumeration
```q
q)e:`a`b`c
q)x:`e$`a`a`c`b
q)value x
`a`a`c`b
```

### String Evaluation
```q
q)value "enlist a:til 5"
0 1 2 3 4

q)value "iasc 2 7 3 1"
3 0 2 1
```

### Namespace Context
```q
q)\d .a
q.a)value"b:2"
q.a)b
2
q.a)\d .
q).a.b
2
```

### List (Function Application)
```q
q)value(+;1;2)
3

q)value(`.q.neg;2)
-2

q)value("{x+y}";1;2)
3
```

### Projection
```q
q)value +[2]
+
2
```

### Composition
```q
q)value differ
$["b"]
~~':
```

### Derived Function
```q
q)f:,/:\:
q)value f
,/:
```

### Operators
```q
q)value each (::;+;-;*;%)
0 1 2 3 4
```

## View

Returns a list of metadata:

- Cached value
- Parse tree
- Dependencies
- Definition

When the view is pending, the cached value is `::`.

```q
q)a:1
q)b::a+1
q)get`. `b
::
(+;`a;1)
,`a
"a+1"
q)b
2
q)get`. `b
2
(+;`a;1)
,`a
"a+1"
```

## Lambda

The structure of `value` on a lambda (V3.5+) is:

```
(bytecode;parameters;locals;(namespace,globals);constants[0];…;constants[n];m;n;f;l;s)
```

Where:
- `m`: bytecode to source position map (or `-1` if unknown)
- `n`: fully qualified function name with namespace (or `@` for inner lambdas)
- `f`: full path to source file (empty string if n/a)
- `l`: line number in file (`-1` if n/a)
- `s`: source code

```q
q)f:{[a;b]d::neg c:a*b+5;c+e}
q)value f
0xa0624161430309220b048100028269410004
`a`b
,`c
``d`e
5
21 19 20 17 18 0 16 11 0 9 0 9 0 25 23 24 2
"..f"
""
-1
"{[a;b]d::neg c:a*b+5;c+e}"
```

With namespace context:

```q
q)\d .test
q.test)f:{[a;b]d::neg c:a*b+5;c+e}
q.test)value f
0xa0624161430309220b048100028269410004
`a`b
,`c
`test`d`e
5
21 19 20 17 18 0 16 11 0 9 0 9 0 25 23 24 2
".test.f"
""
-1
"{[a;b]d::neg c:a*b+5;c+e}"
```

## Local Values in Suspended Functions

See debugging documentation for V3.5+ changes supporting local value inspection in suspended functions.

## get

The function `value` is identical to `get`. By convention, `get` is used for file I/O, but they are interchangeable.

```q
q)get "2+3"
5

q)value each (get;value)
19 19
```

## Related Functions

- `eval`
- `get`
- `parse`
- `.Q.v`
