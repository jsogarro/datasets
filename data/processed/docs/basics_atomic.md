# Atomic Functions and Implicit Iteration

## Overview

Atomic functions in q are those that recursively iterate through list or dictionary arguments down to their constituent atoms. A function qualifies as atomic when `.[f;x]~.[f';x]` holds true—meaning the function produces identical results whether applied directly or with an explicit iterator.

## Formal Definition

The formal criterion states that a function `f` is atomic if applying it directly equals applying it with the Each iterator. For unary functions, this means `f[x]~f'[x]`.

```q
q).[+;(2;(3 4;5))]
5 6
7
q).[+';(2;(3 4;5))]
5 6
7
```

## Informal Definition

A unary function is atomic when it applies to both atoms and lists, processing each atom in a list independently. The negation function `neg` exemplifies this:

```q
q)neg 3 4 5
-3 -4 -5
q)neg (5 2; 3; -8 0 2)
-5 -2
-3
8 0 -2
```

An atomic function can be defined recursively:

```q
neg:{$[0>type x; 0-x; neg'[x]]}
```

For binary functions, atomicity requires these rules:
- Defined for atomic `x` and `y`
- Atom `x` with list `y` produces a list where item `i` equals `f[x;y[i]]`
- List `x` with atom `y` produces a list where item `i` equals `f[x[i];y]`
- Lists `x` and `y` produce a list where item `i` equals `f[x[i];y[i]]`

The Add operator demonstrates this behavior:

```q
q)2 + 3
5
q)2 6 + 3
5 9
q)2 + 3 -8
5 -6
q)2 6 + 3 -8
5 -2
q)(2; 3 4) + ((5 6; 7 8 9); (10; 11 12))
7 8 9 10 11
13  15 16
```

## Length and Type

Arguments must be [conformable](../conformable/). Length mismatches produce errors:

```q
q)1 2 3 + 4 5
'length
```

Type errors can occur at any depth:

```q
q)1 2 3 + (4;"a";5)
'type
```

## Rank

Atomic functions aren't limited to unary or binary forms. Ternary and higher-rank atomic functions exist, such as `{x+y xexp z}` (x plus y to the power z).

## Left- and Right-Atomic

Functions need not be atomic in all arguments. The Index At operator `@[x;y]` is right-atomic: for every left argument `x`, the projection `x@` is atomic. This selects items from `x` using atoms in `y`:

```q
q)2 4 -23 8 7 @ (0 4 ; 2)
2 7
-23
```

## String-Atomic

Since q lacks a dedicated string type (strings are character vectors), string-atomic functions recurse until finding strings or individual characters:

```q
q)upper ("quick";("brown";"fox");"x")
"QUICK"
("BROWN";"FOX")
"X"
```

---

*Reference: Q for Mortals §6.6*
