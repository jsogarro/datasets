# `asc`, `iasc`, `xasc`¶

_Sort and grade: ascending_

## `asc`¶

_Ascending sort_
    
    
```q
asc x     asc[x]
```
Where `x` is a

  * **vector** , returns its items in ascending order of value, with the [sorted attribute](../set-attribute/) set, indicating the list is sorted.
  * **mixed list** , returns the items sorted within datatype and with the sorted attribute set;
  * **nested list** , returns the items sorted lexicographically and with the sorted attribute set;
  * **dictionary** , returns it sorted by the values;
  * **table** , returns it sorted lexicographically by the non-key columns, and
```q
* if there is only one non-key column, setting the sorted attribute set on that column,
* otherwise, setting the parted attribute on the first non-key column.
```
Q chooses from a variety of sorting algorithms depending on the datatype and data distribution.

The sort is stable: it preserves order between equals. If the list was already sorted, this means that the only thing that happens is the attribute being set, and therefore that also happens **in place** (``s` is the only attribute that behaves this way).

`asc` is a uniform function.

### Vector¶
    
    
```q
q)asc 2 1 3 4 2 1 2
`s#1 1 2 2 2 3 4

q)a:0 1
q)b:a
q)asc b  / result has sorted attribute applied
`s#0 1
q)b      / argument was already in ascending order, so the application happened in place
`s#0 1
q)a      / b was a shallow copy of a
`s#0 1
```
### Mixed list¶

In the example below, the boolean is returned first, then the sorted integers, the sorted characters, and then the date.
    
    
```q
q)show l:asc (1;1b;"b";2009.01.01;"a";0)
1b
0
1
"a"
"b"
2009.01.01

q)type each l
-1 -7 -7 -10 -10 -14h     / datatypes are sorted by their type number
```
Warning

Because of this functionality, it is essential to ensure that a list has matching datatypes if we want to sort it (unless we want exactly this to happen).

### Nested list¶
    
    
```q
q)l:("bat";"dog";"cow";"cat")
q)asc l
"bat"
"cat"
"cow"
"dog"
q)attr asc l  / the result has the sorted attribute, but the console doesn't show that with nested lists
`s
```
### Dictionary¶
    
    
```q
q)asc `a`b`c!2 1 3
b| 1
a| 2
c| 3
```
### Table¶
    
    
```q
q)/ Simple table
q)asc ([]a:4 3 4;b:`s`a`d)
a b
---
3 a
4 d
4 s
q)meta asc ([]a:3 4 1;b:`a`d`s)     / sets parted attribute
c| t f a
-| -----
a| j   p
b| s
q)meta asc([]a:3 4 1)               / sets sorted attribute
c| t f a
-| -----
a| j   s

q)/ Keyed table
q)meta asc ([c1:`a`b] c2:2 1; c3:01b) / sets parted attribute
c | t f a
--| -----
c1| s
c2| j   p
c3| b 
q)meta asc ([c1:`a`b] c2:2 1)       / sets sorted attribute
c | t f a
--| -----
c1| s 
c2| j   s



domain: B G X H I J E F C S P M D Z N U V T
range:  B G X H I J E F C S P M D Z N U V T
```
## `iasc`¶

_Ascending grade_
    
    
```q
iasc x    iasc[x]
```
Where `x` is a list or dictionary, returns the indices needed to sort the list `x` in ascending order.
    
    
```q
q)L:2 1 3 4 2 1 2
q)iasc L
1 5 0 4 6 2 3
q)L iasc L
1 1 2 2 2 3 4
q)(asc L)~L iasc L
1b
q)iasc `a`b`c!2 3 1
`c`a`b
```
Tip

You can invert an ordering with `iasc`:
    
    
```q
q)show is:0N?til 5
4 2 1 3 0
q)b:`a`b`c`d`e
q)b is
`e`b`c`d`a
q)b[is] iasc is
`a`b`c`d`e


domain: B G X H I J E F C S P M D Z N U V T
range:  J J J J J J J J J J J J J J J J J J
```
## `xasc`¶

_Sort a table in ascending order of specified columns._
    
    
```q
x xasc y     xasc[x;y]
```
Where `x` is a symbol vector of column names defined in table `y`, which is passed by

  * value, returns
  * [reference](../../basics/glossary/#reference-pass-by), updates



`y` sorted in ascending order by `x`. The sort is by the first column specified, then by the second column within the first, and so on.

The sorted attribute is set on the first column given (if possible). The sort is stable, i.e. it preserves order amongst equals.
    
    
```q
q)show t:0N?([]sym:raze 2#/:`a`b`c; date:6#2025.01.01+til 2; val:50+6?10f)
sym date       val
-----------------------
c   2025.01.01 51.95847
a   2025.01.02 53.40721
b   2025.01.01 50.54001
b   2025.01.02 55.49794
a   2025.01.01 53.83946
c   2025.01.02 55.61526
q)`date xasc t
sym date       val
-----------------------
c   2025.01.01 51.95847
b   2025.01.01 50.54001
a   2025.01.01 53.83946
a   2025.01.02 53.40721
b   2025.01.02 55.49794
c   2025.01.02 55.61526
q)`sym`date xasc t
sym date       val
-----------------------
a   2025.01.01 53.83946
a   2025.01.02 53.40721
b   2025.01.01 50.54001
b   2025.01.02 55.49794
c   2025.01.01 51.95847
c   2025.01.02 55.61526
q)`sym`date xasc `t
`t
q)meta t                      / sym column has sorted attribute
c   | t f a
----| -----
sym | s   s
date| d
val | f
```
**Duplicate column names**  
`xasc` signals `'dup` and the duplicate column name if it finds duplicate columns in the right argument. (Since V3.6 2019.02.19.)

[`.Q.id` (sanitize)](../dotq/#id-sanitize)

### Sorting data on disk¶

`xasc` can sort data on disk directly, without loading the entire table into memory.
    
    
```q
q)t:([]b:`s`g`a`s`a;c:30 10 43 13 24;g:til 5)
q)`:dat/t/ set .Q.en[`:dat]t     / write splayed table
`:dat/t/
q)\ls dat/t                      / splayed columns
,"b"
,"c"
,"g"
q)`c xasc `:dat/t                / sort table on disk by column c
`:dat/t
q)t                              / in-memory table is unsorted
b c  g
------
s 30 0
g 10 1
a 43 2
s 13 3
a 24 4
q)\l dat/t                       / load table from disk
`t 
q)t                              / table is sorted
b c  g
------
g 10 1
s 13 3
a 24 4
s 30 0
a 43 2
```
Duplicate keys in a dictionary or duplicate column names in a table cause sorts and grades to return unpredictable results.

* * *

[`attr`](../attr/); [`desc`, `idesc`, `xdesc`](../desc/); [Set Attribute](../set-attribute/)

[Dictionaries & tables](../../basics/dictsandtables/), [Metadata](../../basics/metadata/), [Sorting](../../basics/by-topic/#sort)   


_Q for Mortals_ [§8.9 Attributes](/q4m3/8_Tables/#88-attributes)

Back to top 
