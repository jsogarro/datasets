# parse

_Parse a string_

```q
parse x     parse[x]
```

## Syntax

Where `x` is a string representing:

- A well-formed q expression, returns a parse tree (V3.4 can accept newlines within the string; earlier versions cannot.)
- A function, returns the function

## Examples

Basic parsing:

```q
q)parse "1 2 3 + 5"            / the list 1 2 3 is parsed as a single item
+
1 2 3
5

q)parse "{x*x}"
{x*x}
```

Parse trees clarify order of execution:

```q
q)parse "1 2 3 +/: 5 7"        / Each Right has postfix syntax
(/:;+)
1 2 3
5 7

q)parse "1 2 3 +neg 5 7"       / neg is applied before +
+
1 2 3
(-:;5 7)
```

Execute a parse tree with `eval`:

```q
q)eval parse "1 2 3 +/: 5 7"
6 7 8
8 9 10
```

Explicit definitions in `.q` are shown in full:

```q
q)foo:{x+2}
q)parse "foo each til 5"
k){x'y}
`foo
(k){$[0>@x;!x;'`type]};5)
```

## Warning

"Should not be used with input data over 2GB in length (0Wi). Returns domain error with this condition since 4.1 2022.04.15."

## QSQL

QSQL queries are parsed to their functional form:

```q
q)\l sp.q
q)x:parse "select part:p,qty by sup:s from sp where qty>200,p=`p1"
q)x
?
`sp
,((>;`qty;200);(=;`p;,`p1))
(,`sup)!,`s
`part`qty!`p`qty

q)eval x
sup| part qty
---| --------
s1 | p1   300
s2 | p1   300
```

## Views

Views are not parsable with `parse`:

```q
q)eval parse"a::5"
5
q)a
5
q)views[]
`symbol$()
```

## Related

- [`eval` and `reval`](../eval/)
- [Parse trees](../../basics/parsetrees/)
