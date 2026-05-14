# Accumulators – Reference

## Overview

An accumulator is an iterator that takes an applicable value as argument and derives a function evaluating the value on its entire first argument, then on results of successive evaluations.

Two accumulators exist: **Scan** and **Over**. They perform identical computations but differ in output:
- Scan returns results of each evaluation
- Over returns only the final result

```q
(+\)2 3 4    / Scan
2 5 9
(+/)2 3 4    / Over
9
```

> "Over requires less memory, because it does not store intermediate results."

## Unary Values

### Converge

Syntax: `(v1\)x` or `(v1/)x`

Evaluates until two successive evaluations match or an evaluation matches `x`.

```q
(neg\)1
1 -1

l:-10?10
(l\)iasc l
4 0 8 5 7 2 6 3 1 9
0 1 2 3 4 5 6 7 8 9
1 8 5 7 0 3 6 4 2 9
8 2 3 4 1 7 6 0 5 9
2 5 7 0 8 4 6 1 3 9
5 3 4 1 2 0 6 8 7 9
3 7 0 8 5 1 6 2 4 9
7 4 1 2 3 8 6 5 0 9

(rotate[1]\)"abcd"
"abcd"
"bcda"
"cdab"
"dabc"

({x*x}\)0.1
0.1 0.01 0.0001 1e-08 1e-16 1e-32 1e-64 1e-128 1e-256 0

(route\)`Genoa
`Genoa`Milan`Vienna`Berlin`London`Paris
```

### Do

Syntax: `n v1\x` or `n v1/x` (n is non-negative integer)

Applies the derived function `n` times.

```q
dbl:2*
3 dbl\2 7
2  7
4  14
8  28
16 56

5 enlist\1
1
,1
,,1
,,,1
,,,,1
,,,,,1

5(`f;)\1
1
(`f;1)
(`f;(`f;1))
(`f;(`f;(`f;1)))
(`f;(`f;(`f;(`f;1))))
(`f;(`f;(`f;(`f;(`f;1)))))

/ First 10+2 numbers of Fibonacci sequence
10{x,sum -2#x}/0 1
0 1 1 2 3 5 8 13 21 34 55 89

fibonacci:{x,sum -2#x}/[;0 1]
fibonacci 10
0 1 1 2 3 5 8 13 21 34 55 89

m:(0 1f;1 1f)
10 (m mmu)\1 1f
1  1
1  2
2  3
3  5
5  8
8  13
13 21
21 34
34 55
55 89
89 144

3 route\`London
`London`Paris`Genoa`Milan
```

### While

Syntax: `t v1\x` or `t v1/x` (t is unary truth map)

Evaluates while `t` applied to the result returns true or non-zero.

```q
(10>)dbl\2
2 4 8 16

{x<1000}{x+x}\2
2 4 8 16 32 64 128 256 512 1024

inc:1+
inc\[105>;100]
100 101 102 103 104 105

inc\[105>sum@;84 20]
84 20
85 21

(`Berlin<>)route\`Paris
`Paris`Genoa`Milan`Vienna`Berlin

waypoints:(!/)yrp`from`wp
waypoints route\`Paris
`Paris`Genoa`Milan`Vienna`Berlin
```

## Binary Values

### Binary Application

Syntax: `x v\y` or `x v/y`

First evaluation: `m[x;first y]`. Result becomes left argument for next evaluation.

```q
1000+\2 3 4
1002 1005 1009

m
1 6 4 4 2
2 7 2 0 5
7 5 6 7 0
2 1 8 1 0
7 3 3 6 8
2 3 8 9 0
1 1 9 6 9
7 8 4 3 0
4 5 8 0 4
9 8 0 3 9

c
4 1 3 3 1 4

7 m\c
0 6 6 6 1 5
```

### Unary Application

When the value is a function with a known identity element `I`, then `I` is the left argument of the first evaluation.

```q
(,\)2 3 4
,2
2 3
2 3 4

{x,y}\[2 3 4]
2
2 3
2 3 4

42{[x;y]x}\2 3 4
42 42 42

({[x;y]x}\)2 3 4
2 2 2

(m\)c
4 3 1 0 6 9

({count x,y}\)("The";"quick";"brown";"fox")
"The"
8
6
4
```

### Keywords: scan and over

Mnemonic keywords provide alternative syntax.

```q
(+) over til 5
10

(+) scan til 5
0 1 3 6 10

m scan c
4 3 1 0 6 9
```

## Ternary Values

Syntax: `v\[x;y;z]` or `v/[x;y;z]`

First evaluation: `v[x;first y;first z]`. Result becomes left argument of next evaluation.

```q
{x+y*z}\[1000;5 10 15 20;2 3 4 5]
1010 1040 1100 1200

{x+y*z}\[1000 2000;5 10 15 20;3]
1015 2015
1045 2045
1090 2090
1150 2150

s:"We are going to advance. Send reinforcements."
ssr\[s;("advance";"reinforcements");("a dance";"three and fourpence")]
"We are going to a dance. Send reinforcements."
"We are going to a dance. Send three and fourpence."

a:1000f;b:1 2 3 4f;c:5 6 7 8f
{z+x*y}\[a;b;c]
1005 2016 6055 24228f

a b\c
1005 2016 6055 24228f
```

## Empty Lists

> "In iterating through an empty list the value is not evaluated."

### Scan Behavior

Returns generic empty list for empty right arguments without evaluation.

```q
mt:0#0
type each (mt;*/[mt];{x*y}/[mt])
7 -7 0h

()~{x+y*z}\[`foo;mt;mt]
1b
```

### Over Behavior

For binary functions with known identity element `I`:

```q
(+/)mt
0

(*/)mt
1
```

For functions without known identity element:

```q
()~({x+y}/)mt
1b
```

For list values:

```q
type 1 0 3h/[til 0]
5h

type (3 4#til 12)/[0#0]
0h
```

Otherwise, returns the left argument:

```q
42+/mt
42

{x+y*z}/[42;mt;mt]
42

42 (3 4#til 12)/[0#0]
42

`foo+/mt
`foo

{x+y*z}/[`foo;mt;mt]
`foo
```
