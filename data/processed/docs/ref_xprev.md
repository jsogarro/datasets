# next, prev, xprev – Immediate or Near Neighbors of List Items

## next

**Next item/s in a list**

```q
next x      next[x]
```

For each item in list `x`, returns the subsequent item. The last item returns null (for vectors) or an empty list `()` (otherwise).

```q
q)next 2 3 5 7 11
3 5 7 11 0N
q)next (1 2;"abc";`ibm)
"abc"
`ibm
`long$()
```

Example use case—calculating quote duration:

```q
q)update (next time)-time by sym from quote
```

`next` is a uniform function.

## prev

**Immediately preceding item/s in a list**

```q
prev x     prev[x]
```

For each item in list `x`, returns the preceding item. The first item returns null (for vectors) or an empty list `()` (otherwise).

```q
q)prev 2 3 5 7 11
0N 2 3 5 7
q)prev (1 2;"abc";`ibm)
`long$()
1 2
"abc"
```

Example use case—shifting times in a table:

```q
q)update time:prev time by sym from t
```

`prev` is a uniform function.

## xprev

**Nearby items in a list**

```q
x xprev y     xprev[x;y]
```

Where `x` is a long atom and `y` is a list, returns for each item of `y` the item `x` indices before it. The first `x` items are null, empty, or blank as appropriate.

Note: There is no `xnext` function, but negative values on the left achieve forward lookups.

```q
q)2 xprev 2 7 5 3 11
0N 0N 2 7 5
q)-2 xprev 2 7 5 3 11
5 3 11 0N 0N
q)1 xprev "abcde"
" abcd"
```

`xprev` is a right-uniform function.
