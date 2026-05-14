# Count the Items of a List or Dictionary

## count

**Number of items**

```q
count x     count[x]
```

Where `x` is:
- a list, returns the number of its items
- a dictionary, the number of items in its value
- anything else, 1

### Examples

```q
q)count 0                            / atom
1
q)count "zero"                       / vector
4
q)count (2;3 5;"eight")              / mixed list
3
q)count each (2;3 5;"eight")
1 2 5
q)count `a`b`c!2 3 5                 / dictionary
3
q)/ The items of a table are its rows
q)count ([]city:`London`Paris`Berlin; country:`England`France`Germany)
3
q)count each ([]city:`London`Paris`Berlin; country:`England`France`Germany)
2 2 2

q)count {x+y}
1
q)count (+/)
1
```

Use with `each` to count items at each level of a list or dictionary.

```q
q)raggedArray:(1 2 3;4 5;6 7 8 9;0)
q)count raggedArray
4
q)count each raggedArray
3 2 4 1
q)raggedDict:`a`b`c!(1 2;3 4 5;"hello")
q)count raggedDict
3
q)count each raggedDict
a| 2
b| 3
c| 5
```

## mcount

**Moving counts**

```q
x mcount y     mcount[x;y]
```

Where:
- `x` is a positive int atom
- `y` is a numeric list

Returns the `x`-item moving counts of the non-null items of `y`. The first `x` items of the result are the counts so far; thereafter the result is the moving count.

### Examples

```q
q)3 mcount 0 1 2 3 4 5
1 2 3 3 3 3
q)3 mcount 0N 1 2 3 0N 5
0 1 2 3 2 2
```

`mcount` is a uniform function.

### Implicit Iteration

`mcount` applies to dictionaries and tables.

```q
q)kt:`k xkey update k:`abc`def`ghi from t:flip d:`a`b!(10 21 3;4 5 6)

q)2 mcount d
a| 1 1 1
b| 2 2 2

q)2 mcount t
a b
---
1 1
2 2
2 2

q)2 mcount kt
k  | a b
---| ---
abc| 1 1
def| 2 2
ghi| 2 2
```
