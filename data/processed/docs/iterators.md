# Iterators

# Iterators¶

--------- maps --------     --------- accumulators ----------
'  Each each      / Over over Converge, Do, While
': Each Parallel peach     \ Scan scan Converge, Do, While
': Each Prior prior
\: Each Left
/: Each Right
'  Case

The iterators (once known as adverbs) are native higher-order operators: they take applicable values as arguments and return derived functions.
They are the primary means of iterating in q.

Iteration in q

Iterators

Applicable value

An applicable value is a q object that can be indexed or applied to arguments: a function (operator, keyword, lambda, or derived function), a list (vector, mixed list, matrix, or table), a file- or process handle, or a dictionary.

For example, the iterator Over (written /) uses a value to reduce a list or dictionary.

`/````q
q)+/[2 3 4]      /reduce 2 3 4 with +
9
q)*/[2 3 4]      /reduce 2 3 4 with *
24
```

Over is applied here postfix, with + as its argument. 
The derived function +/ returns the sum of a list; */ returns its product.
(Compare map-reduce in some other languages.)

`+``+/``*/`## Variadic syntax¶

Each Prior, Over, and Scan applied to binary values derive functions with both unary and binary forms.

```q
q)+/[2 3 4]           / unary
9
q)+/[1000000;2 3 4]   / binary
1000009
```

Variadic syntax

## Postfix application¶

Like all functions, the iterators can be applied with Apply or with bracket notation. 
But unlike any other functions, they can also be applied postfix. They  almost always are.

```q
q)'[count][("The";"quick";"brown";"fox")]   / ' applied with brackets
3 5 5 3
q)count'[("The";"quick";"brown";"fox")]     / ' applied postfix
3 5 5 3
```

Only iterators can be applied postfix.

Regardless of its rank, a function derived by postfix application is always an infix.

To apply an infix derived function in any way besides infix, you can use bracket notation, as you can with any function.

```q
q)1000000+/2 3 4       / variadic function applied infix
1000009
q)+/[100000;2 3 4]     / variadic function applied binary with brackets
1000009
q)+/[2 3 4]            / variadic function applied unary with brackets
9
q)txt:("the";"quick";"brown";"fox")
q)count'[txt]          / unary function applied with brackets
3 5 5 4
```

If the derived function is unary or variadic, you can also parenthesize it and apply it prefix.

```q
q)(count')txt          / unary function applied prefix
3 5 5 4
q)(+/)2 3 4            / variadic function applied prefix
9
```

## Glyphs¶

Six glyphs are used to denote iterators. Some are overloaded.

Iterators

- in bold type derive uniform functions;
- in italic type, variadic functions.

Subscripts indicate the rank of the value; superscripts, the rank of the derived function. (Ranks 4-8 follow the same rule as rank 3.)

`'``\:``/:``':``/``\`Over and Scan, with values of rank >2, derive functions of the same rank as the value.

The overloads are resolved according to the following table of syntactic forms.

## Two groups of iterators¶

There are two kinds of iterators: maps and accumulators.

distribute the application of their values across the items of a list or dictionary. They are implicitly parallel.

apply their values successively: first to the entire (left) argument, then to the result of that evaluation, and so on. With values of rank â¥2 they correspond to forms of map reduce and fold in other languages.

## Application¶

A derived function, like any function, can be applied by bracket notation. 
Binary derived functions can also be applied infix. 
Unary derived functions can also be applied prefix. 
Some derived functions are variadic and can be applied as either unary or binary functions.

This gives rise to multiple equivalent forms, tabulated here.
Any function can be applied with bracket notation or with Apply.
So to simplify, such forms are omitted here in favour of prefix or infix application. 
For example, u'[x] and @[u';x] are valid, but only (u')x is shown here.
(Iterators are applied here postfix only.)

`u'[x]``@[u';x]``(u')x`The mnemonic keywords each, over, peach, prior and scan are also shown.

`each``over``peach``prior``scan``(u')x``u each x``x b'y``v'[x;y;z;â¦]``u``x``g``x``y``v``x``y``z``x b\:d``b``d``x``d b/:y``b``d``y``(u':)x``u peach x``u``x``(b':)y``b prior y``d b':y``b``d``y``int'[x;y;â¦]``[x;y;â¦]``(u/)d``(u\)d``u``d``n u/d``n u\d``u``d``n``t u/d``t u\d``u``d``t``(b/)y``b over y``d b/y``vv/[d;y;z;â¦]``(g\)y``g scan y``d g\y``vv\[d;y;z;â¦]`Key:

```q
d:   data                 
int: int vector         n: int atom â¥0 
v:   value              t: test value
u:   unary value        y: list
b:   binary value       x: list
```

