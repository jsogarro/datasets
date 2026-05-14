# Map Iterators

## Overview

Map iterators are functions that apply values item-wise to dictionaries, lists, or conforming lists. They derive uniform functions from their values.

| Map | Rank | Syntax | Description |
|-----|------|--------|-------------|
| Each | same as v | `v'x` or `v each x` | Apply value to each item |
| Each Left | 2 | `x v2\: y` | Apply binary value between one arg and each item of other |
| Each Right | 2 | `x v2/: y` | Apply binary value between each item and one arg |
| Each Parallel | 1 | `v1': x` or `v1 peach x` | Apply across secondary tasks |
| Each Prior | variadic | `v2': x` or `(v2) prior x` | Apply between each item and preceding item |
| Case | 1+max i | `i'[a;b;c;…]` | Pick items from multiple arguments by selector vector |

---

## Each

*Apply a value item-wise to a dictionary, list, or conforming lists and/or dictionaries.*

```q
(v1')x    v1'[x]       v1 each x
x v2'y    v2'[x;y]
          v3'[x;y;z]
```

Where `v` is an applicable value, `v'` applies `v` to each item of a list, dictionary or to corresponding items of conforming lists. The derived function has the same rank as `v`.

```q
q)(count')`a`b`c!(1 2 3;4 5;6 7 8 9)        / unary
a| 3
b| 2
c| 4
```

Each applied to a binary value is sometimes called *each both* and can be applied infix.

```q
q)1 2 3 in'(1 0 1;til 100;5 6 7)  / in' is binary, infix
110b
```

Iterations of ternary and higher-rank values are applied with brackets.

```q
q){x+y*z}'[1000000;1 0 1;5000 6000 7000]    / ternary
1005000 1000000 1007000
```

Each is redundant with atomic functions.

### `each` keyword

The mnemonic keyword `each` can be used to apply a unary value without parentheses or brackets.

```q
q)(count')string `Clash`Fixx`The`Who
5 4 3 3
q)count'[string `Clash`Fixx`The`Who]
5 4 3 3
q)count each string `Clash`Fixx`The`Who
5 4 3 3
```

---

## Each Left and Each Right

*Apply a binary value between one argument and each item of the other.*

```q
Each Left     x v2\: y    v2\:[x;y]   |->   v2[;y] each x
Each Right    x v2/: y    v2/:[x;y]   |->   v2[x;] each y
```

The maps Each Left and Each Right take binary values and derive binary functions that pair one argument to each item of the other. Effectively, the map projects its value on one argument and applies Each.

```q
q)"abcde",\:"XY"             / Each Left
"aXY"
"bXY"
"cXY"
"dXY"
"eXY"
q)"abcde",/:"XY"             / Each Right
"abcdeX"
"abcdeY"
q)m                          / binary map
"abcd"
"efgh"
"ijkl"
q)m[0 1;2 3] ~ 0 1 m\:2 3
1b
q)0 1 m/:2 3
"cg"
"dh"
q)(flip m[0 1;2 3]) ~ 0 1 m/:2 3
1b
```

### Left, right, `cross`

Each Left combined with Each Right resembles the result obtained by `cross`.

```q
q)show a:{x,/:\:x}til 3
0 0 0 1 0 2
1 0 1 1 1 2
2 0 2 1 2 2
q)show b:{x cross x}til 3
0 0
0 1
0 2
1 0
1 1
1 2
2 0
2 1
2 2
q){}0N!a
((0 0;0 1;0 2);(1 0;1 1;1 2);(2 0;2 1;2 2))
q){}0N!b
(0 0;0 1;0 2;1 0;1 1;1 2;2 0;2 1;2 2)
q)raze[a] ~ b
1b
```

The domains of `\:` and `/:` extend to include atoms and lists beyond binary values:

```q
q)(", "/:)("quick";"brown";"foxes")
"quick, brown, foxes"
q)(0x0\:)3.14156
0x400921ea35935fc4
```

---

## Each Parallel

*Assign sublists of the argument list to secondary tasks, in which the unary value is applied to each item of the sublist.*

```q
(v1':)x   v1':[x]   v1 peach x
```

The Each Parallel map takes a unary value as argument and derives a unary function. The iteration `v1':` divides its list or dictionary argument `x` between available secondary tasks. Each secondary task applies `v1` to each item of its sublist.

```q
❯ q -s 2
kdb+ 5.0.20251113 2025.11.13 Copyright (C) 1993-2025 Kx Systems
...

q)\s
2i
q)\t inv each 2 1000 1000#2000000?1f
2601
q)\t inv peach 2 1000 1000#2000000?1f
1462
```

### `peach` keyword

The binary keyword `peach` can be used as a mnemonic alternative. The following are equivalent.

```q
v1':[list]
(v1':)list
v1 peach list
```

To parallelize a value of rank >1, use Apply to evaluate it on a list of arguments, or define the value as a function that takes a parameter dictionary as argument and pass it a table of parameters to evaluate.

---

## Each Prior

*Apply a binary value between each item of a list and its preceding item.*

```q
(v2':)x    v2':[x]      (v2)prior x
x v2':y    v2':[x;y]
```

The Each Prior map takes a binary value and derives a variadic function. The derived function applies the value between each item of a list or dictionary and the item prior to it.

```q
q)(-':)1 1 2 3 5 8 13
1 0 1 1 2 3 5
```

The first item of a list has no prior item. If the derived function is applied as a binary, its left argument is the 'seed' – the value preceding the first item.

```q
q)1950 -': `S`J`C!1952 1954 1960
S| 2
J| 2
C| 6
```

If applied as a unary and the value is an operator with a known identity element, that identity is used as the seed.

```q
q)(*':)2 3 4                        / 1 is I for *
2 6 12
q)(,':)2 3 4                        / () is I for ,
2
3 2
4 3
q)(-':) `S`J`C!1952 1954 1960       / 0 is I for -
S| 1952
J| 2
C| 6
```

If applied as a unary and the value is not an operator with a known identity element, a null of the same type as the argument (`first 0#x`) is used as the seed.

```q
q){x+2*y}':[2 3 4]
0N 7 10
```

### `prior` keyword

The mnemonic keyword `prior` can be used as an alternative to `':`.

```q
q)(-':) 5 16 42 103
5 11 26 61
q)(-) prior 5 16 42 103
5 11 26 61
q)deltas 5 16 42 103
5 11 26 61
```

---

## Case

*Pick successive items from multiple list arguments: the left argument of the iterator determines from which of the arguments each item is picked.*

```q
int'[a;b;c;…]
```

Where `int` is an integer vector and `[a;b;c;…]` are the arguments, the derived function `int'` returns `r` such that `r_i` is `(args_{int_i})_i`.

The derived function has rank `max[int]+1`.

Atom arguments are treated as infinitely-repeated values.

```q
q)0 1 0'["abc";"xyz"]
"ayc"
q)e:`one`two`three`four`five
q)f:`un`deux`trois`quatre`cinq
q)g:`eins`zwei`drei`vier`funf
q)l:`English`French`German
q)l?`German`English`French`French`German
2 0 1 1 2
q)(l?`German`English`French`French`German)'[e;f;g]
`eins`two`trois`quatre`funf
q)0 2 0'["abc";"xyz";"123";"789"]
"a2c"
q)0 1 0'["a";"xyz"]  /atom "a" repeated as needed
"aya"
```

Use Case to select between record fields according to a test on some other field.

```q
q)([]pref: p;home: h; office: o; call: (`home`office?p)'[h;o])
pref   home             office           call
---------------------------------------------------------
home   "(973)-902-8196" "(431)-158-8403" "(973)-902-8196"
office "(448)-242-6173" "(123)-993-9804" "(123)-993-9804"
office "(649)-678-6937" "(577)-671-6744" "(577)-671-6744"
home   "(677)-200-5231" "(546)-864-5636" "(677)-200-5231"
home   "(463)-653-5120" "(636)-437-2336" "(463)-653-5120"
```

Case is a map. Consider iteration arguments as a matrix where each row corresponds to an argument.

```q
q)a:`Kuh`Hund`Katte`Fisch
q)b:`vache`chien`chat`poisson
q)c:`cow`dog`cat`fish
q)show m:(a;b;c)
Kuh   Hund  Katte Fisch
vache chien chat  poisson
cow   dog   cat   fish
q)i:0 1 0 2
q)i,'til count i
0 0
1 1
0 2
2 3
q)m ./:i,'til count i
`Kuh`chien`Katte`fish
q)i'[a;b;c]
`Kuh`chien`Katte`fish
```

---

## Empty lists

A map's derived function is uniform. Applied to an empty right argument it returns an empty list without evaluation.

```q
q)()~{x+y*z}'[`foo;mt;mt]    / generic empty list ()
1b
```

Watch out for type changes when evaluating lists of unknown length.

```q
q)type (2*')til 5
7h
q)type (2*')til 0
0h
q)type (2*)til 0
7h
```
