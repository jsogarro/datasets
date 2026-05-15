# Apply (At), Index (At), Trap (At) - Complete Documentation

## Overview

The `.` and `@` operators provide three primary functions:
- Apply a function to a list of arguments
- Get items at depth in a list
- Trap errors during evaluation

## Syntax and Semantics

| Rank | Syntax | Function | Semantics |
|------|--------|----------|-----------|
| 2 | `v . vx` / `.[v;vx]` | Apply | Apply `v` to list `vx` of arguments |
| 2 | `v . vx` / `.[v;vx]` | Index | Get item/s `vx` at depth from `v` |
| 2 | `u @ ux` / `@[u;ux]` | Apply At | Apply unary `u` to argument `ux` |
| 2 | `u @ ux` / `@[u;ux]` | Index At | Get items `ux` from `u` |
| 3 | `.[g;gx;e]` | Trap | Try `g . gx`; catch with `e` |
| 3 | `@[f;fx;e]` | Trap At | Try `f@fx`; catch with `e` |

## Parameters

- `e`: An expression, typically a function
- `f`: A unary function with `fx` in its domain
- `g`: A function of rank *n* with `gx` as an atom or list of count *n*
- `v`: A value of rank *n* (or handle) with `vx` as a list of count *n*
- `u`: A unary value (or handle) with `ux` in its domain

## Amend, Amend At

The ternary and quaternary forms:
```q
.[d; i; u]      @[d; i; u]
.[d; i; v; vy]  @[d; i; v; vy]
```

Parameters:
- `d`: A list, dictionary, or handle to a list, dictionary, or datafile
- `i`: Indexes `d` as `d . i` or `d @ i` (must be a list for Amend)
- `u`: A unary with `d` in its domain
- `v`: A binary with `d` and `vy` in left and right domains

## Apply, Index

`v . vx` evaluates value `v` on the *n* arguments listed in `vx`.

```q
q)add               / addition 'table'
0 1 2 3
1 2 3 4
2 3 4 5
3 4 5 6
q)add . 2 3         / add[2;3] (Index)
5
q)(+) . 2 3         / +[2;3] (Apply)
5
q).[+;2 3]
5
q).[add;2 3]
5
```

If `v` has rank *n*, then `vx` has *n* items:
```q
v[vx[0]; vx[1]; …; vx[-1+count vx]]
```

For rank 2:
```q
v[vx[0];vx[1]]
```

For rank 1:
```q
v[vx[0]]
```

### Variadic Operators

Most binary operators have deprecated unary forms and are variadic. Parenthesize them when providing as the left argument:

```q
q).[+;2 2]
4
q)(+) . 2 2
4
```

## Nullaries

Nullaries (functions of rank 0) use `v . enlist[::]`:

```q
q)a: 2 3
q)b: 10 20
q){a + b} . enlist[::]
12 23
```

## Index

`d . i` returns an item from list or dictionary `d` as specified by successive items in list `i`. Since 4.1t 2022.03.25, `d` can be a persisted table.

The list `i` is composed of successive indexes into `d`. The evaluation follows:
```q
( (d@i[0]) @ i[1] ) @ i[2] …
```

```q
q)d
((1 2 3;4 5 6 7) ;(8 9;10;11 12) ;(13 14;15 16 17 18;19 20))
q)d . enlist 1      / select item 1, i.e. d@1
8 9
10
11 12
q)d . 1 2           / select item 2 of item 1
11 12
q)d . 1 2 0         / select item 0 of item 2 of item 1
11
```

A right argument of `enlist[::]` selects the entire left argument:

```q
q)d . enlist[::]
(1 2 3;4 5 6 7)
(8 9;10;11 12)
(13 14;15 16 17 18;19 20)
```

### Index At

Successive applications of Index At:

```q
q)((d @ 1) @ 2) @ 0         / selection in terms of a series of @s
11
q)d @/ 1 2 0                / selection in terms of @-Over
11
```

For any vector `i`, `d . i` is identical to `d@/i`.

### Cross Sections

Index is cross-sectional when items of `i` are lists. Selections are made for all combinations of atoms from `i[0]`, atoms from `i[1]`, etc.

```q
q)d . (2 0; 0 1)
13 14 15 16 17 18
1 2 3 4 5 6 7
q)count each d . (2 0; 0 1)
2 2
```

The first item selects from `d`, the second from each selected item:
```q
(d@i[0])@'i[1]
```

When items are vectors, the result is rectangular to at least depth `count i`, with shape determined by `count each i`.

### Nulls in i

Nulls in `i` mean "select all". If `i[0]` is null, continue with `d` and `1_i`. If `i[1]` is null, continue from each selection with `2_i`, etc.

```q
q)d
(1 2 3;4 5 6 7)
(8 9;10;11 12)
(13 14;15 16 17 18;19 20)
q)d . (::;0)
1 2 3
8 9
13 14
```

Equivalent expression:
```q
q)d . (0 1 2;0)
```

Another example with `i[1]` equal to null:

```q
q)d . (0 2;::;1 0)
(2 1;5 4)
(14 13;16 15;20 19)
```

### General Case of Non-Negative Integer List i

When items of `i` are non-negative integer atoms/lists or null, the result structure cascades through `i`. With nulls aside, the result is structurally like `i[0]`, except atoms in `i[0]` are replaced by structures like `i[1]`, and so on.

Recursive definition using Index At:

```q
Index:{[d;F;R]
  $[ F~::; Index[d; first R; 1 _ R];
     0 =count R; d @ F;
     0>type F; Index[d @ F; first R; 1 _ R]
     Index[d;; R]'F ]}
```

Where `d . i` is `Index[d;first i;1_i]`.

At each recursion step:
- If `F` is null, select all of `d` and continue
- If remainder is empty, apply Index At and finish
- If `F` is an atom, apply Index At to select that item and continue
- Otherwise, apply Index with fixed arguments `d` and `R` to items of `F`

### Dictionaries and Symbolic Indexing

If `i` is a symbol atom, `d` must be a dictionary or handle of a directory, and `d . i` selects the value with that key:

```q
dir:`a`b!(2 3 4;"abcdefg")
```

Access:
```q
`dir . enlist`b     / "abcdefg"
`dir . (`b;1 3 5)   / "bdf"
```

Mixed indexing with integers and symbols:

```q
q)(1;`a`b!(2 3 4;10 20 30 40)) . (1; `b; 2)
30
```

Rules:
- Each item of `i` must contain entirely non-negative integer atoms or entirely symbol atoms
- If the *k*th item is symbols, all items at depth *k* selected must be dictionaries
- Symbols can only select from dictionaries/directories; integers cannot

Dictionary with all values:
```q
d . enlist key d
```

### Step Dictionaries

A step dictionary has the sorted attribute. For keys outside the domain, `d@i` returns nulls of the same type as keys:

```q
q)d:`cat`cow`dog`sheep!`chat`vache`chien`mouton
q)d
cat  | chat
cow  | vache
dog  | chien
sheep| mouton
q)d `sheep`snake`cat`ant
`mouton``chat`
```

Numeric step dictionary:

```q
q)e:(10*til 10)!til 10
q)e
0 | 0
10| 1
20| 2
30| 3
40| 4
50| 5
60| 6
70| 7
80| 8
90| 9
q)e 80 35 20 -10
8 0N 2 0N
```

For a step dictionary `s`, values for keys outside the domain are the values for the highest keys lower than the requested key:

```q
q)ds:`s#d
q)ds~d
1b
q)ds `sheep`snake`cat`ant
`mouton`mouton`chat`
```

Numeric example:

```q
q)es:`s#e
q)es~e
1b
q)es 80 35 20 -10
8 3 2 0N
```

## Apply At, Index At

`@` is syntactic sugar for unary `u` and 1-item list `ux`. `u@ux` is equivalent to `u . enlist ux`.

Brackets are syntactic sugar for `.`:

```q
{`o`h`l`c!(first;max;min;last)@\:x}1 2 3 4 22  / open, high, low, close
o| 1
h| 22
l| 1
c| 22
```

Use derived function `@\:` to apply a list of unary values to the same argument.

## Composition

A sequence of unaries `u`, `v`, `w`… can be composed with Apply At. All but the last `@` may be elided:

```q
q)tc:til count@  / indexes of a list
q)tc "abc"
"0 1 2"
```

The last value can have higher rank if projected as a unary by Apply:

```q
q)di:reciprocal(%).  / divide into
q)di 2 3             / divide 2 into 3
1.5
```

## Trap

In the ternary form, if evaluation fails, the expression is evaluated:

```q
q).[+;"ab";`ouch]
`ouch
```

If the expression is a function, it evaluates on the error text:

```q
q).[+;"ab";{"Wrong ",x}]
"Wrong type"
```

For successful evaluation, the ternary returns the same result as the binary:

```q
q).[+;2 3;{"Wrong ",x}]
5
```

### Trap At

`@[f;fx;e]` is equivalent to `.[f;enlist fx;e]`. Use this simpler form for unary values:

```q
@[2+;"42";`err]
`err
```

### Limit of the Trap

Trap catches only errors in applications of `f` or `g`. Errors in evaluation of `fx` or `gx` themselves are not caught:

```q
q)@[2+;"42";`err]
`err
q)@[2+;"42"+3;`err]
'type
  [0]  @[2+;"42"+3;`err]
                ^
```

### When e is not a function

If `e` is a function, it evaluates _only_ if `f` or `g` fails and is parsed before other expressions. If `e` is any other expression, it _always_ evaluates first in right-to-left sequence:

```q
q)@[string;42;a:100] / expression not a function
"42"
q)a // but a was assigned anyway
100
q)@[string;42;{b::99}] / expression is a function
"42"
q)b // not evaluated
'b
  [0]  b
       ^
```

For most purposes, `e` should be a function.

## Errors Signalled

- `index`: An atom in `vx` or `ux` is not an index to an item-at-depth in `d`
- `rank`: The count of `vx` is greater than the rank of `v`
- `type`: `v` or `u` is a symbol atom but not a handle to a value
- `type`: An atom of `vx` or `ux` is not an integer, symbol, or null
