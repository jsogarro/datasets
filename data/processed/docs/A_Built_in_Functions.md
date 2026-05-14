# A Built In Functions

# A. Built-in Functions¶

### A.0 Overview¶

The collection of built-in functions in q is rich and powerful. We include here a user guide for those built-ins that were not covered in the main text. For more details, see the Reference section at code.kx.com.

### A.1 acos¶

`acos`The atomic acos is the mathematical inverse of cos. For a float argument between -1 and 1, acos returns the float between 0 and π whose cosine is the argument.

`acos``cos``acos````q
q)acos 1
0f
q)acos 1.414213562373095
0n
q)acos -1
3.141593
q)acos 0
1.570796
```

### A.2 all¶

`all`The aggregate all is (&/), which applies & cumulatively across a numeric list and returns the boolean result.

`all``(&/)``&````q
q)all 100100b
0b
q)all 10 20 30
1b
```

### A.3 and¶

`and`The function and is the same as & for people who like typing extra characters.

`and``&````q
q)1b and 0b
0b
q)42 and 43
42
```

### A.4 any¶

`any`The aggregate any is (|/), which applies | cumulatively across a numeric list and returns the boolean result.

`any``(|/)``|````q
q)any 100100b
1b
q)any null til 10
0b
```

### A.5 asc¶

`asc`The uniform function asc returns its argument list of comparables sorted in ascending order with the `s# attribute applied. When evaluated on a dictionary, it reorders the key-value pairs (on a copy) so that the values are sorted.

`asc```s#````q
q)asc 3 7 2 8 1 9
`s#1 2 3 7 8 9
q)asc `b`c`a!3 2 1
a| 1
c| 2
b| 3
```

### A.6 asin¶

`asin`The atomic asin is the mathematical inverse of sin. For a float argument between -1 and 1, asin returns the float between –π/2 and π/2 whose sine is the argument.

`asin``sin``asin````q
q)asin 0
0f
q)asin 1.414213562373095%2
0.7853982
q)asin 1
1.570796
q)asin -1
-1.570796
```

### A.7 atan¶

`atan`The atomic atan is the mathematical inverse of tan. For a float argument, atan returns the float between –π/2 and π/2 whose tangent is the argument.

`atan``tan``atan````q
q)atan 0
0f
q)atan 1.414213562373095
0.9553166
q)atan 1
0.7853982
```

### A.8 attr¶

`attr`The function attr returns any attribute of its argument as a symbol. No attributes is the empty symbol

`attr````q
q)attr til 5
`
q)attr asc 30 20 40 10
`s
```

### A.9 avg¶

`avg`The aggregate avg returns the float average of a numeric list.

`avg````q
q)avg 1+til 1000
500.5
```

The function avg is equivalent to

`avg`{sum[x]%count x}

`{sum[x]%count x}`It is possible to apply avg to a nested list provided the sublists conform. In this context, the result conforms to the sublists and the average is calculated recursively on the sublists.

`avg````q
q)avg (1 2;100 200;1000 2000)
367 734f
```

### A.10 avgs¶

`avgs`The uniform avgs is (avg\), which computes the running averages of a numeric list.

`avgs``(avg\)````q
q)avgs 1+til 10
1 1.5 2 2.5 3 3.5 4 4.5 5 5.5
```

### A.11 bin¶

`bin`The binary bin takes a simple list of items (target) sorted in strictly increasing order as its first parameter. It is atomic in its second parameter (source). Loosely speaking, the result of bin is the position at which source would fall in target, looking from the left. The type of source must strictly match the type of target; no type promotion is performed.

`bin``bin`Note

While the items of the first argument bin should be in strictly increasing order for the result to meaningful, this condition is not enforced. The result of bin when the first argument is not strictly increasing is essentially undefined.

`bin``bin`More precisely, the result is -1 if source is less than the first item in target. Otherwise, the result is the index of the last item of target that is less than or equal to source; this is the found index if source is in target. If source is greater than the last item in target, the result is the last index of target

Tip

For large sorted lists, the binary search performed by bin can be much faster than the linear search performed by ?.

`bin``?`Here are some examples.

```q
q)L:1.0+til 50000000
q)L bin 25000000.
24999999
q)L?25000000.
24999999
q)\t L bin 25000000.
0
q)\t L?25000000.
33
q)L bin 25000000.5
24999999
62
q)L?25000000.5
50000000
q)\t L bin 25000000.5
0
q)\t L?25000000.5
63
q)L bin 50000000.5
49999999
q)L?50000000.5
50000000
```

### A.12 binr¶

`binr`The binary binr is closely related to bin. Whereas bin looks from the left, binr looks from the right. The result is the index of the first item of the first parameter that is greater than or equal to the second parameter.

`binr``bin``bin``binr````q
q)L:1.0+til 50000000
q)L bin 25000000.5
24999999
q)L binr 25000000.5
25000000
```

### A.13 Roll/Deal ?¶

`?`The binary operators Roll and Deal generate pseudo-random results.

In the case where the first parameter is a non-negative long and the second parameter is a non-negative numeric value (bound), ? returns a list of pseudo-random numbers with replacement between 0 and bound, not including bound.

`?````q
q)5?10
2 4 5 4 2
q)5?10
7 8 5 6 4
q)10?100.0
10.24432 86.71096 72.78528 16.27662 68.84756 81.77547 75.20102 10.86824 95.98..
q)10?100.0
64.30982 67.08738 67.89082 41.2317 98.77844 38.67353 72.6781 40.46546 83.5506..
```

In the case where the first parameter is a negative long and the second parameter is a non-negative long (bound), ? returns a list of pseudo-random numbers without replacement between 0 and bound, not including bound.

`?````q
q)-5?10
1 8 5 7 0
q)5?10
2 4 5 4 2
```

Zen Moment

The expression –n?n returns a random permutation of til n.

`–n?n``til n`In a case where the second parameter is a list (source) and the first parameter is a long (count), ? returns a list of count items randomly drawn from source, with replacement for positive count and without replacement for negative count.

`?````q
q)5?`a`b`c`d`e`f
`b`c`f`f`b
q)5?`a`b`c`d`e`f
`e`e`a`e`e
q)-5?`a`b`c`d`e`f
`c`a`d`f`e
q)-5?`a`b`c`d`e`f
`f`a`e`d`c
```

In a case where the first parameter is a positive long (count) and the second argument is a symbol of the form `n where n is a positive integer no greater than eight, the result is a random list of count symbols, each comprising exactly n characters. The symbols are distinct when count is negative.

```q
q)10?`1
`c`d`o`m`h`m`n`e`m`m
q)-10?`1
`h`p`j`n`e`o`a`k`l`i
q)-10?`3
`emb`agl`mho`ndm`gmj`egi`gek`hcc`mmb`kbh
```

### A.14 Enum Extend ?¶

`?`Enum Extend ? has as first parameter the symbolic name of a (presumably unique) list of symbols (target). It is atomic in the second parameter, which is a symbol (source). The result is the enumeration of source over target, where source is appended to target if it is not already contained therein. As a side effect of the function, symbols from source not in target are appended to target.

`?````q
q)sym:`a`b`c
q)`sym?`a`x`b`y`z
`sym$`a`x`b`y`z
q)sym
`a`b`c`x`y`z
```

### A.15 cor¶

`cor`The binary cor takes two numeric lists of the same count and returns the mathematical correlation between the items of the two lists as a float.

`cor````q
q)23 -11 35 0 cor 42 21 73 39
0.9070229
```

The function cor is equivalent to

`cor`{cov[x;y]%dev[x]*dev y}

`{cov[x;y]%dev[x]*dev y}`### A.16 count¶

`count`The aggregate count returns a long representing the number of items in its atom or list parameter. As with any function defined on lists, it also applies to the value list of a dictionary and hence to tables and keyed tables.

`count````q
q)count 42
1
q)count til 100
100
q)count `a`b`c`d!10 20 30 40
4
q)count ([] c1:10 20 30; c2:1.1 2.2 3.3)
3
q)count ([k:10 20] c:`one`two)
2
```

Tip

You cannot use count to determine whether an entity is an atom or list since atoms and singletons both have count 1. Instead test the sign of the type.

q)0>signum type 42
1b
q)0>signum type enlist 42
0b

`count````q
q)0>signum type 42
1b
q)0>signum type enlist 42
0b
```

Do you know why they call it count? Because it loves to count! Nyah, ha, ha, ha, ha. Vun, and two, and tree, and …

`count`### A.17 cov¶

`cov`The binary cov takes two numeric lists of the same length and returns a float equal to the mathematical covariance between the items of the two lists. If both arguments are lists, they must have the same count; atoms return 0f.

`cov``0f````q
q)23 -11 35 0 cov 42 21 73 39
411.25
```

The function cov is equivalent to,

`cov`{avg[x*y]-avg[x]*avg y}

`{avg[x*y]-avg[x]*avg y}`### A.18 cross¶

`cross`The binary cross takes atoms or lists as arguments and returns their Cartesian product – that is, the set of all pairs drawn from the two arguments.

`cross````q
q)1 2 cross `a`b`c
1 `a
1 `b
1 `c
2 `a
2 `b
2 `c
```

You can also apply cross to dictionaries and tables.

`cross`The function cross is equivalent to,

`cross`{raze x,\:/:y}

`{raze x,\:/:y}`### A.19 cut¶

`cut`The binary function cut can take a list of long (indices) as its first parameter and a list (source) as it right parameter. It returns the list obtained by splitting source at the positions in indices.

`cut`If the initial item of indices is not 0, the first sublist split out is dropped.

```q
q)L:10 20 30 40 50 60 70 80 90 100
q)0 2 5 7 cut L
10 20
30 40 50
60 70
80 90 100
q)2 5 7 cut L
30 40 50
60 70
80 90 100
```

When the first parameter is a non-negative integral atom (width), source is split at indices that are multiples of width.

```q
q)2 cut L
10 20
30 40
50 60
70 80
90 100
q)3 cut L
10 20 30
40 50 60
70 80 90
,100
```

### A.20 cut _¶

`_`The binary function _ has several forms, depending on the types of its parameters. See also Drop and cut.

`_``cut`When _ is used infix, surrounding it with whitespace will avoid it getting mixed up with variable names, since _ is a valid q name character.

`_``_`When the first parameter is an integral atom (count) and the second parameters is a list (source), the result is source with count items dropped from the head if count is positive and from the tail when count is negative.

```q
q)2 _ 10 20 30 40 50 60 70 80 90 100
30 40 50 60 70 80 90 100
q)-2 _ 10 20 30 40 50 60 70 80 90 100
10 20 30 40 50 60 70 80
```

When the first parameter of _ is a list of long (indices) and the second parameter as a list (source) , it returns the list obtained by splitting source at the positions in indices. This is the same behavior as cut for this signature.

`_``cut`If the initial item of indices is not 0, the first sublist split out is dropped.

```q
q)L:10 20 30 40 50 60 70 80 90 100
q)0 2 5 7 _ L
10 20
30 40 50
60 70
80 90 100
q)2 5 7 _ L
30 40 50
60 70
80 90 100
```

When the second parameter of _ is a dictionary (source) and the first parameter is a list of key values whose type matches source, the result is a dictionary obtained by removing the specified key-value pairs from the target. Since tables and keyed tables are dictionaries, _ applies to them by extension.

`_``_````q
q)`a`c _ `a`b`c!10 20 30
b| 20
q)`c1`c2 _ ([] c1:`a`b; c2:10 20; c3:1.1 2.2)
c3
---
1.1
2.2
q)([] k:`a`c) _ ([k:`a`b`c] v:10 20 30)
k| v
-| --
b| 20
```

The first parameter must be a list, so a single value for the first parameter in this form must be enlisted.

```q
q)(enlist `b) _ `a`b`c!10 20 30
a| 10
c| 30
```

When the first parameter of _ is a list or a dictionary (source) and the second parameter is an atom representing an index or key, the result is obtained by deleting the specified item from (a copy of) source.

`_````q
q)101 102 103 104 105 _ 2
101 102 104 105
q)(`a`b`c!10 20 30) _ `b
a| 10
c| 30
q)([] c1:`a`b; c2:10 20; c3:1.1 2.2) _ 1
c1 c2 c3
---------
a 10 1.1
q)([k:101 102 103] c:`one`two`three) _ 102
k | c
---| -----
101| one
103| three
```

### A.21 deltas¶

`deltas`The uniform deltas is (-':), which returns the difference of each item in a numeric list with its predecessor.

`deltas``(-':)````q
q)deltas 1 2 3 4 5
1 1 1 1 1
q)deltas 96.25 93.25 58.25 73.25 89.50 84.00 84.25
96.25 -3 -35 15 16.25 -5.5 0.25
```

If you are looking to bound the absolute difference between successive elements, the second example shows that the initial item of the result will be troublesome. In this case, you can use,

deltas0:{first[x] –': x}

`deltas0:{first[x] –': x}`In our example above,

```q
q)deltas0 96.25 93.25 58.25 73.25 89.50 84.00 84.25
0 -3 -35 15 16.25 -5.5 0.25
```

### A.22 desc¶

`desc`The uniform desc returns (a copy of) its argument list of comparables sorted in descending order. When evaluated on a dictionary, it reorders the key-value pairs (on a copy) so that the values are sorted.

`desc````q
q)desc 3 7 2 8 1 9
9 8 7 3 2 1
q)desc `b`c`a!2 3 1
c| 3
b| 2
a| 1
```

### A.23 dev¶

`dev`The aggregate dev returns the float standard deviation of a numeric list.

`dev````q
q)dev 1000?100.
29.40271
```

The function dev is equivalent to,

`dev`{sqrt var x}

`{sqrt var x}`### A.24 differ¶

`differ`The uniform differ is

`differ`not (~':)

`not (~':)`It asks if each item in a list is not identical to its predecessor. The item at index 0 in the result is always 1b.

`1b````q
q)differ 0 1 1 2 3 2 2 2 4 1 1 3 4 4 4 4 5
11011100110110001b
q)differ "mississippi"
q)differ (1 2; 1 2; 3 4 5)
101b
```

One use of differ is to locate runs of repeated items in a list.

`differ````q
q)L:0 1 1 2 3 2 2 2 4 1 1 3 4 4 4 4 5
q)nd|next nd:not differ L
01100111011011110b
```

### A.25 distinct¶

`distinct`The function distinct returns the unique items in its list argument, in order of first occurrence. Note that it does not apply the `u# attribute.

`distinct```u#````q
q)distinct 1 2 3 2 3 4 6 4 3 5 6
1 2 3 4 6 5
```

Tip

Following is a test for existence of duplicates.
{count[x]=count distinct x}

`{count[x]=count distinct x}`Since a table is a list of records, distinct effectively removes duplicate rows. All fields’ records must be identical for them to be considered identical.

`distinct````q
q)distinct ([]a:1 2 3 2 1; b:`washington`adams`jefferson`adams`washington)
a b
------------
1 washington
2 adams
3 jefferson
q)distinct ([]a:1 2 3 2 1; b:`washington`adams`jefferson`adams`wasington)
_
```

### A.26 enlist¶

`enlist`The function enlist returns a list whose items comprise its arguments. The most common use is to create a singleton list from a single argument.

`enlist`Unlike user-defined functions, the number of arguments to enlist is not restricted to eight.

`enlist````q
q)enlist 42
,42
q)count enlist 10 20 30
1
q)enlist[1;2;3;4;5;6;7;8;9;10]
1 2 3 4 5 6 7 8 9 10
```

### A.27 eval¶

`eval`The unary eval evaluates a list that is a valid q parse tree; it is the same code used in the q interpreter. Such a list can be produced using parse on a string containing a valid q expression, or by hand – if you know what you're doing. A full discussion of parse trees is beyond the scope of this tutorial.

`eval``parse````q
q)eval parse "a:6*7"
42
q)a
42
q)eval (+;1;(*;6;7))
43
```

### A.28 except¶

`except`The binary except takes a list (or dictionary) as its first parameter (target) and an atom or list of items (or keys) of the same type as target for its second parameter (source) and returns those items in target that are not specified in source. The returned items are in the order of their first occurrence in target.

`except````q
q)1 2 3 4 3 2 except 2
1 3 4 3
q)string[2015.01.01] except "."
"20150101"
q)(`a`b`c`d!10 20 30 40) except `a`d!10 40
20 30
q)([] c1:`a`b`c`d; c2:10 20 30 40) except ([] c1:`a`d; c2:10 40)
c1 c2
-----
b 20
c 30
```

The result of except is always a list.

`except`### A.29 exit¶

`exit`The unary exit takes a long as its parameter (retval) and causes the q process to exit, returning retval to the OS.

`exit`There is no prompt for confirmation.

### A.30 Fill ^¶

`^`The binary fill ^ takes a list as its second parameter (target) and an atom of the same type as its first parameter (fillval). It returns (a copy of) target with null values filled with fillval.

`^````q
q)42^1 2 3 0N 5 0N
1 2 3 42 5 42
q)"_"^"Now is the time"
"Now_is_the_time"
q)`NA^`First`Second``Fourth
`First`Second`NA`Fourth
```

Fill is atomic in the second parameter.

```q
q)42^(1;0N;(100;200 0N))
1
42
(100;200 42)
```

Like all functions, fill operates on the values of a dictionary.

```q
q)42^`a`b`c`d!100 0N 200 0
a| 100
b| 42
c| 200
d| 0
q)0^([]c1:1.0 2.0 0n; c2:0N 2 0N)
c1 c2
-----
1 0
2 2
0 0
```

### A.31 Find ?¶

`?`The binary operator ? Find takes a list (target) as its first parameter and an atom of the same type (source) as its second parameter. It returns the index of the first occurrence of source in target or the count of target if it is not found. It is atomic in the second parameter.

`?`The simplest case is when source is an atom.

```q
q)100 99 98 87 96 98?98
2
q)`one`two`three?`four
3
q)"Now is the time"?"the"
7 8 9
```

Tip

The first example demonstrates that Find returns only the index of the first occurrence of source. To find the indices of all occurrences you could use,

q){where x=y}[100 99 98 87 96 98;98]
2 5

```q
q){where x=y}[100 99 98 87 96 98;98]
2 5
```

Find also works on general lists, although if you need this you should probably consider redesigning your program.

```q
q)(1 2; 3 4; `a`b)?`a`b
2
q)((0;1 2);3 4;5 6)?5 6
2
```

Find can also take a dictionary as its first parameter (target) and a dictionary value as its second parameter *value*. It returns the first key that maps to value. It is atomic in the second parameter.

`*value*````q
q)(`a`b`c`d!10 20 30 10)?10
`a
q)(`a`b`c`d!10 20 30 10)?10 30
`a`c
```

By extension, find applies to tables and keyed tables.

```q
q)([] c1:`a`b`c; c2:10 20 30)?`c1`c2!(`b;20)
1
q)([] c1:`a`b`c; c2:10 20 30)?(`b;20)
1
q)([k:1 2 3] c:100 101 102)?101
k| 2
```

Zen Moment

Viewing a list or dictionary as a mapping, Find is the inverse mapping.

### A.32 fills¶

`fills`The uniform fills is ^\, which fills forward, meaning that non-null items are filled over succeeding null items.

`fills``^\````q
q)fills 1 0N 3 0N 0N 5
1 1 3 3 3 5
q)fills `x``y```z
`x`x`y`y`y`z
q)update fills c2 from ([] `a`b`c`d`e`f; c2:1 0N 3 0N 0N 5)
x c2
----
a 1
b 1
c 3
d 3
e 3
f 5
```

If you need to fill initial nulls, use the binary form of ^\.

`^\````q
q)fills 0N 0N 3 0N 5
0N 0N 3 3 5
q)0 ^\ 0N 0N 3 0N 5
0 0 3 3 5
```

### A.33 flip¶

`flip`The unary function flip transposes a rectangular list, column dictionary or table (source).

`flip`When source is a rectangular list, the items are rearranged, effectively reversing the first two indices in indexing. For example,

```q
q)show m:(1 2 3;10 20 30)
1 2 3
10 20 30
q)flip m
1 10
2 20
3 30
```

When flip converts a column dictionary to a table and vice versa, no data is actually rearranged. Only column indexing is reversed.

`flip````q
q)dc:`c1`c2!(`a`b`c;10 20 30)
q)t:([] c1:`a`b`c; c2:10 20 30)
q)dc[`c1;2]~t[2;`c1]
```

### A.34 getenv¶

`getenv`The unary function getenv takes a symbol argument representing the name of an OS environment variable and returns the value if any of that environment variable as a string.

`getenv``if any````q
q)getenv `SHELL
"/bin/bash"
```

### A.35 group¶

`group`The unary function group takes a list (source) and returns a dictionary in which each distinct item in source is mapped to the indices of its occurrences in source. The keys of the result are in the order of their first appearance in source.

`group````q
q)group "i miss mississippi"
i| 0 3 8 11 14 17
 | 1 6
m| 2 7
s| 4 5 9 10 12 13
p| 15 16
```

### A.36 gtime¶

`gtime`The atomic gtime converts local time to UTC time.

`gtime````q
q).z.P
2015.04.12D12:10:20.685653000
q)gtime .z.P
2015.04.12D22:10:24.861692000
q).z.p
2015.04.12D22:10:28.597654000
```

### A.37 iasc¶

`iasc`The uniform iasc takes a list or a dictionary (source). Considering source as a mapping, the result of iasc is a list of the indices/keys of source in the order that would sort it ascending. Otherwise put, composing source with the result of iasc sorts in ascending order.

`iasc``iasc``iasc````q
q)L:30 70 20 80 10 90
q)iasc L
4 2 0 1 3 5

q)L iasc L
10 20 30 70 80 90

q)d:`c`a`b!30 10 20
q)iasc d
`a`b`c
q)d iasc d
10 20 30
```

This is useful when you want to use one list to control the order of others – e.g., manual sort on the serialized columns of a splayed table (see §14.3).

Zen Moment

The composite iasc iasc provides the list of indices that transforms the sorted entity back to the original.

`iasc iasc`### A.38 Identity ::¶

`::`The unary operator denoted :: is the identity function – i.e., it returns its argument.

`::````q
q)::[42]
42
q)(::) `Zaphod
`Zaphod
```

The identity function cannot be used naked with prefix syntax or @. You must either enclose it in parentheses or enclose its argument in square brackets.

`@`### A.39 idesc¶

`idesc`The unary function idesc takes a list or a dictionary (source). Considering source as a mapping, the result of idesc is a list of the indices/keys of source in the order that would sort it descending. Otherwise put, composing source with the result of idesc sorts in descending order.

`idesc``idesc``idesc````q
q)L:30 70 20 80 10 90
q)idesc L
5 3 1 0 2 4
q)L idesc L
90 80 70 30 20 10
q)d:`c`a`b!30 10 20
q)idesc d
`c`b`a
q)d idesc d
30 20 10
```

This is useful when you want to use one list to control the order of others – e.g., manual sort on the serialized columns of a splayed table (see §14.2).

Zen Moment

The composite idesc idesc provides the list of indices that transforms the sorted entity back to the original.

`idesc idesc`### A.40 in¶

`in`The binary function in is atomic in its first parameter (source) and takes an atom or list second parameter (target). It returns a boolean indicating whether source appears in target. The comparison is strict with regard to type.

`in````q
q)42 in 0 6 7 42 98
1b
q)4 in 42
0b
q)"4" in "42"
1b
q)"cat" in "abcdefg"
110b
```

### A.41 inter¶

`inter`The binary inter returns the items in its first list parameter that occur in its second list parameter.

`inter````q
q)1 2 3 4 inter 3 4 5
3 4
```

Lists differ from sets in that they are ordered and allow duplicates, so inter is not commutative in general.

`inter````q
q)1 2 3 1 inter 4 1
1 1
q)4 1 inter 1 2 3 1
,1
q)1 2 3 inter 4 3 2
2 3
q)4 3 2 inter 1 2 3
3 2
```

You can use inter to find common records in tables having identical schemas.

`inter````q
q)t1:([] c1:`a`b`c; c2:10 20 30)
q)t2:([] c1:`c`d; c2:30 40)
q)t1 inter t2
c1 c2
-----
c  30
```

### A.42 inv¶

`inv`The unary inv returns the inverse of a float matrix.

`inv````q
q)m:(1.1 2.1 3.1; 2.3 3.4 4.5; 5.6 7.8 9.8)
q)inv m
-8.165138 16.51376 -5
12.20183 -30.18349 10
-5.045872 14.58716 -5
```

An integer matrix must be cast to float.

### A.43 Join ,¶

`,`The binary operator Join , appends its second parameter to the first. When both operands are either lists or atoms, the result concatenates the second parameter to the first.

`,````q
q)1,2 3 4
1 2 3 4

q)1 2 3,4
1 2 3 4

q)1 2,3 4
1 2 3 4
```

The result is a general list unless all items are of homogeneous type.

```q
q)1 2 3,4.0
1
2
3
4f
```

When both parameters are dictionaries, the result is the merge of the second parameter into the first using upsert semantics. Otherwise put, assignments in the second parameter prevail over those in the first.

```q
q)(`a`b`c!10 20 30),`c`d!300 400
a| 10
b| 20
c| 300
d| 400
```

By extension, you can use , on tables and keyed tables with identical schemas.

`,`### A.44 key¶

`key`The key operator vies for the title of most overloaded q operator. Its eponymous action is to return the key of a dictionary, including the special case of the key table portion of a keyed table. It can be applied by value or name.

`key````q
q)key `a`b`c!10 20 30
_
q)kt:([k:`a`b`c] v:10 20 30)
q)key kt
_
q)key `kt
_
```

A context – i.e. the content of a namespace – is a dictionary, so key applied to the symbolic name will return the names of its constituents. Observe that contexts below the root all have an initial null name.

`key````q
q)key `.
_
q).jab.a:42
q)key `.jab
_
```

To list all contexts, apply key to the null symbol.

`key````q
q)key `
`q`Q`h`j`o`jab
```

Given a simple list, key returns the symbolic name of its type.

`key````q
q)key 10 20 30
`long
q)key "so long"
_
```

Given an enumerated entity, key returns the name of its enumeration domain. This applies to foreign key columns, where it returns the name of the target keyed table.

`key````q
q)sym:`c`b`a
q)key `sym$`a`b`a`c`a
_

q)kt:([k:`a`b`c] v:10 20 30)
q)t:([] k:`kt$`a`b`a`b`a; v: 10 20 30 49 59)
q)key t `k
_
```

Given an I/O handle corresponding to a directory, key returns a list of the symbolic names of entities in that directory, including hidden files.

`key````q
q)\ls q
"README.txt"
"m32"
"q.k"
"q.q"
"s.k"
"sp.q"
"trade.q"

q)key `:q
`.DS_Store`README.txt`m32`q.k`q.q`s.k`sp.q`trade.q
```

Tip

An empty directory returns an empty list of type symbol whereas a non-existent directory returns an empty general list.

This provides a simple test for the existence of a directory.

```q
q)\ls q/jab
ls: q/jab: No such file or directory
'os
q)()~key `:q/jab
1b
q)\mkdir q/jab
q)key `:q/jab
`symbol$()
```

Given an I/O handle corresponding to a file, key returns the symbolic file name if the file exists and the general empty list () if it does not. As above, this provides a test for existence of the file.

`key``()````q
q)key `:q/trade.q
`:q/trade.q
q)()~key `:q/quote.q
1b
```

Last, but not least, key also acts as a synonym of til.

`key``til````q
q)key 10
_
```

### A.45 like¶

`like`The binary like performs pattern matching on its first string parameter (source) according to the pattern in its string second parameter (pattern). It returns a boolean result indicating whether pattern is matched. The pattern is expressed as a mix of regular characters and special formatting characters. The special chars are ?,*, the pair[and], and a ^ that is enclosed in square brackets.

`like``?,``, the pair``and``^`The special char ? represents an arbitrary single character in the pattern.

`?````q
q)"fan" like "f?n"
1b
q)"fun" like "f?n"
1b
q)"foP" like "f?p"
0b
```

The special char * represents an arbitrary sequence of characters in the pattern.

`*`As of this writing (Sep 2015), only a single occurrence of * is allowed in the pattern, except that a * can occur at both the beginning and end of the pattern.

`*``*````q
q)"how" like "h*"
1b

q)"hercules" like "h*"
1b

q)"wealth" like "*h"
1b

q)"flight" like "*h*"
1b

q)"Jones" like "J?ne*"
1b

q)"Joynes" like "J*ne*"
'nyi
```

The special character pair [ and [ encloses a sequence of alternatives for a single character match.

`[``[````q
q)"flap" like "fl[ao]p"
1b

q)"flip" like "fl[ao]p"
0b

q)"459-0609" like "[0-9][0-9][0-9]-0[0-9][0-9][0-9]"
1b

q)"459-0609" like "[0-9][0-9][0-9]-1[0-9][0-9][0-9]"
0b
```

The special character ^ is used in conjunction with [ and ] to indicate that the enclosed sequence of characters is disallowed. For example, to test whether a string ends in a numeric character,

`^``[``]````q
q)"M26d" like "*[^0-9]"
1b

q)"Joe999" like "*[^0-9]"
0b
```

Enclose a special char in square brackets to escape it as a normal char.

```q
q)"a_c" like "a[*]"
0b

q)"a*c" like "a[*]c"
1b
```

### A.46 lower¶

`lower`The atomic lower takes a char, string or symbol argument and returns the result of converting all alpha text to lower case.

`lower````q
q)lower `A
`a

q)lower "a Bc42De"
"a bc42de"
```

### A.47 lsq¶

`lsq`The binary matrix function lsq takes float matrix parameters A and B and returns the matrix X that solves the following matrix equation, where · is matrix multiplication.

`lsq`A = X·B

For example,

```q
q)A:(1.1 2.2 3.3;4.4 5.5 6.6;7.7 8.8 9.9)
q)B:(1.1 2.1 3.1; 2.3 3.4 4.5; 5.6 7.8 9.8)
q)A lsq B
1.211009 -0.1009174 2.993439e-12
-2.119266 2.926606 -3.996803e-12
-5.449541 5.954128 -1.758593e-11
```

Observe that lsq is equivalent to,

`lsq````q
q)A mmu inv B
1.211009 -0.1009174 1.77991e-12
-2.119266 2.926606 -5.81224e-12
-5.449541 5.954128 -1.337952e-11
```

Integer matrices must be cast to float.

### A.48 ltime¶

`ltime`The atomic ltime converts UTC time to local time.

`ltime````q
q).z.P
2015.04.12D12:12:42.475837000

q)ltime .z.P
2015.04.12D02:12:45.987860000

q).z.p
2015.04.12D22:12:47.859567000
```

### A.49 ltrim¶

`ltrim`The unary ltrim takes a string parameter and returns the result of removing leading blanks.

`ltrim````q
q)ltrim " abc "
"abc " "
q)ltrim " "
""
```

You can also apply ltrim to a non-blank char but it fails for a blank char.

`ltrim````q
q)ltrim "a"
"a"
q)ltrim " "
k){$[~t&77h>t:@x;.z.s'x;" "=*x;(+/&\" "=x)_x;x]}
'type
_
1b
" "
```

### A.50 mavg¶

`mavg`The binary mavg takes first parameter a long (length) and returns the length-wise moving average of its numeric list second parameter (source). At each position in source, it computes the average of the length preceding items, or as many as are available up to length, ignoring internal nulls. It is uniform in its second argument.

`mavg`In the following example, the first item in the result is the average of itself only; the second result item is the average of the first two source items; all other items reflect the average of the item at the position along with its two predecessors.

```q
q)3 mavg 10 20 30 40 50
10 15 20 30 40f

q)3 mavg 0N 10 20 30 40 50
0n 10 15 20 30 40
```

For length 1, the result is the source converted to float.

### A.51 max¶

`max`The aggregate max is (|/), which applies | cumulatively across a list of items of underlying numeric type and returns the final result. It is the same as any on a boolean list.

`max``(|/)``|``any````q
q)max 100100b
1b

q)max 10?2015.01.01
2013.09.09
```

### A.52 maxs¶

`maxs`The uniform maxs is (|\), which applies | across a list of comparable items and returns the intermediate results.

`maxs``(|\)``|````q
q)maxs 1 2 5 4 10
1 2 5 5 10

q)maxs "Beeblebrox"
"Beeelllrrx"
```

### A.53 mcount¶

`mcount`The uniform binary mcount takes as first parameter a long (length) and returns a long equal to the length-wise moving count of its list second parameter (source). At each position in source, it counts the number of non-null preceding items up to length. This function is useful in computing other moving quantities, such as moving average, since it correctly reports the number of predecessors at the head of the list and also ignores internal nulls.

`mcount````q
q)3 mcount 10 20 30 40 50
1 2 3 3 3i

q)3 mcount 10 20 0N 40 50 60 0N
1 2 2 2 2 3 2i
```

### A.54 md5¶

`md5`The unary function md5 computes the MD5 (Message Digest Algorithm 5) hash of a string as a list of 16 bytes.

`md5````q
q)md5 "Life the Universe and Everything"
0x0e9a5631e4db880d43808504d05348df
```

### A.55 mdev¶

`mdev`The uniform binary mdev takes first parameter a long (length) and returns the length-wise moving standard deviation of its numeric list second parameter (source). At each position in source, it computes the standard deviation of the length preceding items, or as many as are available at that position up to length. It excludes internal nulls from the calculation.

`mdev````q
q)3 mdev 10 20 30 40 50
0 5 8.164966 8.164966 8.164966

q)3 mdev 10 20 30 0N 50
0 5 8.164966 5 10
```

### A.56 med¶

`med`The aggregate med returns the float median of a numeric list.

`med`For lists and dictionaries, the result is a float.

```q
q)med 1000?1000.
499.1908
```

The function med is equivalent to,

`med````q
{avg x (iasc x)@floor .5*-1 0+count x,:()}
```

### A.57 min¶

`min`The aggregate min is (&/), which applies & cumulatively across a list of underlying numeric type and returns the final result. It is the same as and for a boolean list.

`min``(&/)``&``and````q
q)min 100?100
2

q)min 100100b
0b
```

### A.58 mins¶

`mins`The uniform mins is (&\), which computes the cumulative minimums of a list of comparables and returns the intermediate results.

`mins``(&\)````q
q)mins 10 4 5 1 2
10 4 4 1 1

q)mins "the cat"
"theecaa"
```

### A.59 mmax¶

`mmax`The uniform binary mmax takes as first parameter a long (length) and returns the length-wise moving maximum of its second parameter list of comparables (source). At each position in source, it computes the maximum of the length preceding items, or as many as are available at that position up to length.

`mmax`In the following example, the first item in the result is the max of itself only; the second result item is the max of the first two source items; all other items reflect the max of the item at the position along with its two predecessors.

`max``max``max````q
q)3 mmax 20 10 30 50 40
20 20 30 50 50

q)3 mmax 20 10 30 0N 40
20 20 30 30 40
```

### A.60 mmin¶

`mmin`The uniform binary mmin takes first parameter a long (length) and returns the length-wise moving minimum of its second parameter list of comparables (source). At each position in source, it computes the minimum of the length preceding items, or as many as are available at that position up to length.

`mmin`In the following example, the first item in the result is the min of itself only; the second result item is the min of the first two source items; all other items reflect the min of the item at the position along with its two predecessors.

`min``min``min````q
q)3 mmin 20 10 30 50 40
20 10 10 10 30
```

For length less than or equal to 0 the result is source.

### A.61 mmu¶

`mmu`The binary matrix multiplication mmu returns the matrix product of its two float vector or matrix parameters, which must be of the correct shape. It reduces to dot product on vectors. Integer matrices must be cast to float.

`mmu````q
q)m1:(1.1 2.2 3.3;4.4 5.5 6.6;7.7 8.8 9.9)
q)m1 mmu flip m1
16.94 38.72 60.5
38.72 93.17 147.62
60.5 147.62 234.74

q)m2:`float$(0 0 1; 0 1 0; 1 0 0)
q)m2 mmu m2
1 0 0
0 1 0
0 0 1

q)1 2 3f mmu 1 2 3f
14f
```

The $ operator is overloaded to yield matrix multiplication when its parameters are float vectors or matrices.

`$````q
q)m2 $ m2
1 0 0
0 1 0
0 0 1
```

### A.62 msum¶

`msum`The uniform binary msum takes first parameter a long (length) and returns the length-wise moving sum of its numeric list second parameter (source). At each position in source, it computes the sum of the length preceding items, or as many as are available at that position up to length. It excludes internal nulls from the sum.

`msum`In the following example, the first item in the result is the sum of itself only; the second result item is the sum of the first two source items; all other items reflect the sum of the item at the position along with its two predecessors.

```q
q)3 msum 10 20 30 40 50
10 30 60 90 120
```

### A.63 next¶

`next`The uniform next returns its list parameter shifted one position to the left with a null (or empty) in the last item. Otherwise put, each item in the original is replaced with its successor.

`next````q
q)next 1 2 3 4 5
2 3 4 5 0N

q)next (1 2; 3 4 5; 6 7)
3 4 5
6 7
`long$()
```

### A.64 null¶

`null`The atomic function null tests its argument for null value. It is preferred to testing for equality as it works for all types.

`null````q
q)null 42
0b

q)null 0n
1b

q)null "Now is the time"
000100100010000b
```

A common idiom combines where with null to obtain the positions of the null items in a list.

`where``null````q
q)where null 1 2 3 0N 5 0N
3 5
```

Because null is atomic, it picks out null values in all fields.

`null````q
q)null ([] c1:`a``b; c2:0N 20 30)
c1 c2
-----
0 1
1 0
0 0
```

This can be used to identify null columns.

```q
q)where all null ([] c1:`a`b`c; c2:0n 0n 0n; c3:10 0N 30)
,`c2
```

### A.65 or¶

`or`The function or is the same as | for people who like typing extra characters.

`or``|````q
q)1b or 0b
1b

q)42 or 43
43
```

### A.66 over¶

`over`The function over is a convenience function that renames the unary form of / for those allergic to k.

`over``/````q
q){x+2*y} over 2 3 5 7
32

q)(+) over 2 3 5 7
17
```

### A.67 parse¶

`parse`The unary parse returns the q parse tree for a string representing a q expression. Applying eval to the result of parse effectively evaluates the expression. The composition of eval after parse is essentially the q interpreter. A discussion of q parse trees is beyond the scope of this tutorial.

`parse``eval``parse``eval``parse````q
q)parse "7*4+2"
*
7
(+;4;2)
q)eval parse "7*4+2"
42
```

It is useful to apply parse to a string containing a query template for the purpose of expressing the query in functional form. The result will often include k code but it is usually recognizable and you can use it in functional form.

`parse`The constraint portion of the resulting parse tree will contain an extra level of nesting displayed in the k form ,,.

`,,`You must remove one level for the functional form.

```q
q)t:([]c1:`a`b`c; c2:10 20 30)
q)parse "select c2:2*c2 from t where c1=`c"
?
`t
,,(=;`c1;,`c)
0b
(,`c2)!,(*;2;`c2)
q)?[`t; enlist (=;`c1;enlist `c); 0b; (enlist `c2)!enlist (*;2;`c2)]
c2
--
60
```

### A.68 peach¶

`peach`A full discussion of how q handles concurrency and distributed processing is beyond the scope of this text. We provide a description of the essential features in q3.2 as of this writing (Sep 2015) and refer the reader to the KX site for up-to-date details.

Concurrency in q uses two basic constructs: peach and slaves. Slaves can be either threads spawned by the main q process or independent q processes. How many and which type of slaves are declared to the q process at startup. The function peach is a “parallel” version of each that applies a function over a list by distributing the items of the list across slaves. This is q’s implementation of the “map” portion of the distributed map-reduce paradigm, which is much-ballyhooed in other environments.

`peach``peach``each`If q is not started with slaves, peach reverts to each, meaning function application across the list occurs sequentially in the main q thread.

`peach``each`Use the command line parameter -s to start a slave-enabled q session. In classic Arthurian fashion, the interpretation of –s depends on the sign of the integer that follows it. A positive integer n indicates that the q process should start with n slave threads to be used by peach. A negative integer –n means that you will provide n independent slave q processes for peach to use. The physical implementation of work distribution is different in these two cases, but the logical result is the same.

`-s``–s``peach``peach`#### A.68.1 Threads¶

When q is started with -s n, the q process automatically spawns n slave threads in addition to the main thread at startup. When you subsequently use peach to apply a function – i.e., map it – across a list, the application is automatically distributed across the slave threads for you. The result of the application is the same as if you had used each but if q is running on a machine with multiple cores, you should expect a speedup, depending on what the function does, how slave threads are allocated to cores and the load on your machine. For example, on the author’s laptop that has 4 cores and hyperthreading, we find that the benefit of multiple slaves for evaluating a compute-intensive calculation on a list of 8 items plateaus at 4 slaves.

`-s n``peach``each`$ rlwrap q/m32/q -s 2
q)\t {sum exp x?1.0} each 8#1000000
134
q)\t {sum exp x?1.0} peach 8#1000000
72
$ rlwrap q/m32/q -s 4
q)\t {sum exp x?1.0} each 8#1000000
136
q)\t {sum exp x?1.0} peach 8#1000000
46
$ rlwrap q/m32/q -s 8
q)\t {sum exp x?1.0} each 8#1000000
135
q)\t {sum exp x?1.0} peach 8#1000000
43

```q
$ rlwrap q/m32/q -s 2
```

```q
q)\t {sum exp x?1.0} each 8#1000000
134
q)\t {sum exp x?1.0} peach 8#1000000
72
```

```q
$ rlwrap q/m32/q -s 4
```

```q
q)\t {sum exp x?1.0} each 8#1000000
136
q)\t {sum exp x?1.0} peach 8#1000000
46
```

```q
$ rlwrap q/m32/q -s 8
```

```q
q)\t {sum exp x?1.0} each 8#1000000
135
q)\t {sum exp x?1.0} peach 8#1000000
43
```

In order to maintain consistency during concurrent application, the following restriction is placed on function evaluation within slave threads. Code executed in a slave thread cannot update global variables – i.e., a function can only update local variables.

Tip

If you use a timer on the main thread or other tricks to update globals that are read by slave threads, you are cruising for a bruising. While these updates are locked, there is no notion of a transaction with attendant rollback. It is unlikely that your function executing on the timer is atomic at the hardware level. Should it fail mid-flight, you should expect the workspace would be in an inconsistent state with no built-in recovery mechanism.

Now we describe how the slave thread execution is implemented. The items in the list are pre-assigned to slaves in a round-robin fashion. For example, if you peach a function on a list with 9 items in a session with 4 slaves: items 0, 4 and 8 will be executed sequentially on slave 0; items 1, 5 and 9 will be executed sequentially on slave 1; items 2 and 6 will be executed sequentially on slave 2; and items 3 and 7 will be executed sequentially on slave 3. The sequential execution within slaves occurs concurrently across slaves.

`peach`The slaves use thread-local heaps and the (partial) results of application within each slave are serialized and copied to the main thread where they are deserialized and assembled into the final result. Since there is overhead associated with serialization/deserialization, we have:

Tip

A function applied with peach should do enough computation to outweigh the serialization overhead. You can estimate the serialization time and space of a q entity with
\ts:100 -9!-8!entity

`peach``\ts:100 -9!-8!entity`We collect here observations on execution within a slave. See the KX site for more details.

- If the list has a single item, peach application occurs in the main q thread only.
- If the function applied with peach performs grouping on a symbol list – e.g., a select that groups on a ticker symbol – this will be significantly slower in the slave because the optimized algorithm used in the main thread is not available in the slave.
- In q3.*, a socket can be used from the main thread only. As there is no locking around a socket descriptor, a communication handle shared between threads would result in garbage due to message interleaving.
- Each slave thread has its own heap, a minimum of 64MB. Executing .Q.gc[] in the main thread triggers garbage collection in the slave threads too. Automatic garbage collection within each thread is only executed for that particular thread, not across all threads.
- Symbols are internalized in a single memory area accessible to all threads.

If the list has a single item, peach application occurs in the main q thread only.

`peach`If the function applied with peach performs grouping on a symbol list – e.g., a select that groups on a ticker symbol – this will be significantly slower in the slave because the optimized algorithm used in the main thread is not available in the slave.

`peach`In q3.*, a socket can be used from the main thread only. As there is no locking around a socket descriptor, a communication handle shared between threads would result in garbage due to message interleaving.

Each slave thread has its own heap, a minimum of 64MB. Executing .Q.gc[] in the main thread triggers garbage collection in the slave threads too. Automatic garbage collection within each thread is only executed for that particular thread, not across all threads.

`.Q.gc[]`Symbols are internalized in a single memory area accessible to all threads.

#### A.68.2 Distributed peach¶

`peach`Starting q with –s –n indicates:

`–s –n`- In addition to the main q process, you will instantiate n workers, where a worker is an independent q process with an open port
- You will connect the master q process to each worker
- You will provide the master process a unique’d list of open handles to the workers

In addition to the main q process, you will instantiate n workers, where a worker is an independent q process with an open port

You will connect the master q process to each worker

You will provide the master process a unique’d list of open handles to the workers

The list of handles to the workers is obtained from the system variable .z.pd upon each invocation of peach. You can set .z.pd either to an integer list of open handles with the `u# attribute applied, or a function that returns the same.

`.z.pd``peach``.z.pd```u#`Tip

You must apply the `u# attribute or you will get the error:

``u#`'.z.pd - expected unique vector of int handles

```q
'.z.pd - expected unique vector of int handles
```

In contrast to the situation with slave threads, where the threads are created for you automatically, with distributed peach it is your responsibility to instantiate the separate worker processes and make them available to the main q process. In production environments this may entail meeting security requirements – e.g., Kerberos. You can start the workers externally to your main process or you can start them from within the process using system.

`peach``system`The workers can be on the same machine as the master process or on any machine on a network so long as it is accessible from the master process. Since they are independent q processes that share no data with the main process, the distributed function is not restricted as to what it can do on the worker. This provides a very flexible framework for distributed computing.

Once you have started the worker processes, you must open a connection to each one, keeping track of the open handles.

You cannot use these handles for other messaging, as peach will close any handle that presents a message not in its expected format.

`peach`You can assign the unique’d list of the open connections to .z.pd or you can return such a list from a function that you assign to .z.pd. In contrast to the slave thread implementation, where the assignment to slaves is determined in advance, individual computations are automatically distributed to workers that become free. Thus we get a simple form of dynamic load balancing.

`.z.pd``.z.pd`Following is an example that starts four external workers on the same machine from within the master process, wires the workers directly into .z.pd and then uses peach to distribute work to them.

`.z.pd``peach`~$q -s -4
..
q){system "q -q -p ",string[x]," &"} each 20000+til 4
q).z.pd:`u#hopen each `$"::",/:string 20000+til 4
q)\t {sum exp x?1.0} each 8#1000000
134
q)\t {sum exp x?1.0} peach 8#1000000
46

```q
~$q -s -4
..
```

```q
q){system "q -q -p ",string[x]," &"} each 20000+til 4
q).z.pd:`u#hopen each `$"::",/:string 20000+til 4
q)\t {sum exp x?1.0} each 8#1000000
134
q)\t {sum exp x?1.0} peach 8#1000000
46
```

You can also assign to .z.pd a function that manages the connections. The code below monitors closing connections via .z.pc, which removes the handle of any closed connection from its self-maintained list of active handles. The function assigned to .z.pd checks to see if all expected handles are open and, if not, closes any open handles and then reopens all the specified handles.

`.z.pd``.z.pc``.z.pd````q
q)handles:`u#`int$()
q).z.pd:{
 if[0>=n:neg system "s";
 '"must start q with -s -n"];
 if[n<>count handles; hclose each handles;
 `handles set `u#hopen each 20000+til n];
 handles}
q).z.pc:{`handles set `u#handles except x;}
```

### A.69 prd¶

`prd`The aggregate prd is (*/), which applies * cumulatively across a numeric list and returns the final result.

`prd``(*/)``*`Note the missing ‘o’ in the name.

```q
q)prd 1+til 5

q)prd 1+til 5
120

q)prd 10?100.
2.667807e+13
```

It is possible to apply prd to a nested list provided the sublists conform. In this case, the result conforms to the sublists and the product is calculated recursively on the sublists.

`prd````q
q)prd (1 2; 100 200; 1000 2000)
100000 800000
```

### A.70 prds¶

`prds`The uniform prds is (*\), which applies * cumulatively across a list of numeric type and returns the intermediate results.

`prds``(*\)``*`Note the missing ‘o’ in the name.

```q
q)prds 1+til 5
1 2 6 24 120
```

### A.71 prev¶

`prev`The uniform prev returns a list argument shifted one position to the right with a null (or empty) in the first item. Otherwise put, each item in the original is replaced with its predecessor.

`prev````q
q)prev 1 2 3 4 5
0N 1 2 3 4

q)prev (1 2; 3 4 5; 6 7)
`long$()
1 2
3 4 5
```

### A.72 prior¶

`prior`The function prior is a convenience function that renames the unary form of ': for those allergic to k.

`prior``':````q
q)(-) prior 10 11 12 13 14
10 1 1 1 1
```

### A.73 rank¶

`rank`The uniform rank returns the sort order of each item in a list of comparables.

`rank````q
q)rank 50 20 30 10 40
4 1 2 0 3

q)rank `x`a`b`z`c
3 0 1 4 2
```

### A.74 ratios¶

`ratios`The uniform ratios is (%':), which applies % across a list of numeric type and returns the intermediate results. That is, it returns the ratio of each item with its predecessor. The initial item in the result is 1.0.

`ratios``(%':)``%````q
q)ratios 10 20 30 40 50
10 2 1.5 1.333333 1.25
```

If you are looking to bound the relative difference between successive elements, this example shows that the initial item of the result will be troublesome.

In this case, you can use,

```q
ratios1:{first[x] %': x}
```

For our example above,

```q
q)ratios1 10 20 30 40 50
1 2 1.5 1.333333 1.25
```

### A.75 raze¶

`raze`The unary raze is (,/). It effectively eliminates the top-most level of nesting from a list by concatenating across it.

`raze``(,/)````q
q)raze (1 2 3;3; 4 5)
1 2 3 3 4 5
```

Observe that raze only removes the top-most level of nesting but you can use each to reach deeper levels.

`raze``each````q
q)raze ((1 2;3 4);(5;(6 7;8 9)))
1 2
3 4
5
(6 7;8 9)
q)raze each ((1 2;3 4);(5;(6 7;8 9)))
1 2 3 4
(5;6 7;8 9)a
```

### A.76 reval¶

`reval`Introduced in q3.3, reval is a version of eval that behaves as if the command line option -b had been set. This means that all updates on the server are blocked for the duration of the evaluation. See §A.27 and §13.2.1.

`reval``eval``-b`### A.77 reverse¶

`reverse`The uniform function reverse inverts the item order of its list argument.

`reverse````q
q)reverse 1 2 3 4 5
5 4 3 2 1

q)reverse `a`b`c!10 20 30
c| 30
b| 20
a| 10

q)reverse ([] c1:`a`b`c; c2:10 20 30)
c1 c2
-----
c 30
b 20
a 10

q)reverse ([k:`a`b`c] v:10 20 30)
k| v
-| --
c| 30
b| 20
a| 10
```

For nested entities, the reversal takes place only at the topmost level but you can use each to reach lower levels.

`each````q
q)reverse (1 2 3;"abc";`Four`Score`and`Seven)
`Four`Score`and`Seven
"abc"
1 2 3

q)reverse each (1 2 3;"abc";`Four`Score`and`Seven)
3 2 1
"cba"
`Seven`and`Score`Four
```

### A.78 rload¶

`rload`The function rload is a convenience function that loads a splayed table into a variable with the same name as the splayed directory. The usual way to do this is with get, whose result can be assigned to an arbitrary variable.

`rload``get````q
q)`:db/t/ set ([] c1:10 20 30; c2:1.2 2.2 3.3)
`:db/t/
q)rload `:db/t
`t
q)count t
3

q)t1:get `:db/t
q)count t1
3
```

### A.79 rotate¶

`rotate`The uniform binary rotate takes as its first parameter a long (length) and returns the result of rotating its second parameter by length positions. The rotation is to the left for positive length and to the right for negative length. For length 0, it returns the source.

`rotate````q
q)2 rotate 1 2 3 4 5
3 4 5 1 2

q)22 rotate 1 2 3 4 5
3 4 5 1 2
```

### A.80 rsave¶

`rsave`The function rsave is a convenience function that splays a table variable to a directory with the same name. The preferred way to do this is with set, which allows the directory name to be specified.

`rsave``set````q
q)t:([] c1:10 20 30; c2:1.2 2.2 3.3)
q)rsave `:db/t
`:db/t/
q)`:db/t1/ set t
`:db/t1/
```

### A.81 rtrim¶

`rtrim`The unary rtrim takes a string argument and returns the result of removing trailing blanks.

`rtrim````q
q)rtrim " abc "
" abc"
q)rtrim " "
""
```

You can apply rtrim to a non-blank char but it fails for a blank char.

`rtrim````q
q)rtrim "a"
"a"
q)rtrim " "
k){$[~t&77h>t:@x;.z.s'x;" "=*x;(+/&\" "=x)_x;x]}
'type
_
1b
" "
```

### A.82 scan¶

`scan`The function scan is a convenience function that renames the unary form of \ for those allergic to k.

`scan``\````q
q){x+2*y} scan 2 3 5 7
2 8 18 32
q)(+) scan 2 3 5 7
2 5 10 17
```

### A.83 scov¶

`scov`The binary scov returns the statistical covariance of its numeric list arguments of the same length.

`scov````q
q)2 3 5 7 scov 4 3 0 2
-2.416667
```

It is equivalent to,

```q
{cov[x;y]*count[x]%-1+count x}
```

### A.84 sdev¶

`sdev`The unary sdev returns the statistical standard deviation of its numeric list argument.

`sdev````q
q)sdev 10 343 232 55
155.1322
```

It is equivalent to,

```q
{sqrt var[x]*count[x]%-1+count x}
```

### A.85 setenv¶

`setenv`The binary setenv takes as first parameter a symbol representing the name of an OS environment variable and a string second parameter. It calls the underlying OS to set the named environment variable to the specified string value.

`setenv````q
q) `FOO setenv "test"
q)getenv `FOO
_
```

### A.86 sin¶

`sin`The atomic sin takes a float and returns its mathematical sine.

`sin````q
q)sin 0f
0f

q)pi:3.141592653589793
q)sin pi
1.224647e-16

q)pi%2
1.570796

q)sin pi%4
0.7071068
```

### A.87 ss¶

`ss`The binary ss, for "string search", performs similar pattern matching as like against its first string parameter (source), looking for matches to its string second parameter (pattern). However, the result of ss is a list containing the position(s) of the matches of the pattern in source.

`ss``like``ss````q
q)ss["Now is the time for all good men to come to";"me"]
13 29 38

q)"fun" ss "f?n"
,0
```

If no matches are found, an empty list of long is returned.

```q
q)ss["ab";"z"]
`long$()
```

You cannot use * to match with ss.

`*``ss````q
q)"fun" ss "f*n"
'length
```

### A.88 ssr¶

`ssr`The triadic ssr, for "string search and replace", extends the capability of ss with replacement. The result is a string based on the first string parameter (source) in which all occurrences of the second string parameter (pattern) are replaced with the third string argument.

`ssr``ss````q
q)ssr["suffering succotash";"s";"th"]
"thuffering thuccotathh"
```

You cannot use * to match with ssr.

`*``ssr`You can use the over iterator with ssr to replace multiple items.

`over``ssr````q
q)(ssr/)["results_%div_%dept.csv"; ("%div";"%dept"); ("banking";"m&a")]
"results_banking_m&a.csv"
```

### A.89 string¶

`string`The unary string can be applied to any q entity to produce a textual representation. For atoms, lists and functions, the result of string is a list of char that does not contain any q formatting characters. Following are some examples.

`string``string````q
q)string 42
"42"

q)string 6*7
"42"

q)string 424224242i
"424224242"

q)string `Zaphod
"Zaphod"

q)string {[a] a*a}
"{[a] a*a}"
```

The first example demonstrates that string is not atomic, because the result of applying it to an atom is a list of char.

`string`Although string is not atomic, it is pseudo-atomic, in that it recurses through a list.

`string````q
q)string 42 98
"42"
"98"
q)("42";"98")
"42"
"98"
```

This accounts for the un-intuitive behavior of string on an actual string.

`string````q
q)string "Beeblebrox"
,"B"
,"e"
,"e"
,"b"
,"l"
,"e"
,"b"
,"r"
,"o"
```

Considering a list as a mapping, string acts on the range of the mapping. Viewing a dictionary as a generalized list, we conclude that the action of string on a dictionary should also apply to its range.

`string``string````q
q)string 1 2 3!100 101 102
1| "100"
2| "101"
3| "102"
```

A table is the flip of a column dictionary, so we expect string to operate on the range of the column dictionary.

`string````q
q)string ([] a:1 2 3; b:`a`b`c)
a    b
---------
,"1" ,"a"
,"2" ,"b"
,"3" ,"c"
```

Finally, a keyed table is a dictionary, so we expect string to operate on the value table.

`string````q
q)string ([k:1 2 3] c:100 101 102)
k| c
-| -----
1| "100"
2| "101"
3| "102"
```

### A.90 sublist¶

`sublist`The binary sublist retrieves a sub-list of contiguous items from a list. The first parameter is a simple list of two non-negative integers: the first item is the starting index (start); the second item is the number of items to retrieve (count). The second parameter (target) is a list or dictionary.

`sublist````q
q)0 3 sublist 10 20 30 40 50
10 20 30
q)0 10 sublist "abcdef"
"abcdef"
q)0 2 sublist `a`b`c!10 20 30
a| 10
b| 20
```

Tip

The construct 0 n sublist … is preferred to n# … for extracting the first n items from a list since the latter will repeat the initial items if necessary to reach the specified count. This is ignored surprisingly often in practice.

`0 n sublist …``n# …``n`### A.91 sum¶

`sum`The aggregate sum is (+/), which applies + cumulatively across a numeric list and returns the final result.

`sum``(+/)``+````q
q)sum 1+til 100
5050

q)sum 100?100.
5387.56
```

It is possible to apply sum to a nested list provided the sublists conform. In this case, the result conforms to the sublists and the sum is calculated recursively on the sublists.

`sum````q
q)sum (1 2;100 200;1000 2000)
1101 2202
```

### A.92 sums¶

`sums`The uniform sums is (+\), which applies + cumulatively across a list of numeric type and returns the intermediate results.

`sums``(+\)``+````q
q)sums 1+til 10
1 3 6 10 15 21 28 36 45 55
```

### A.93 sv¶

`sv`The binary sv – "scalar from vector" – has several forms. It is usually written infix since that is easier to read.

`sv`The first form takes a char as its first parameter and a list of strings (source) as its second. It returns a string that is the concatenation of the strings in source, separated by the specified char.

```q
q)"," sv ("Now";"is";"the";"time";"")
"Now,is,the,time,"
```

When sv is used with an empty symbol as its first parameter and a list of symbols as its second (source), the result is a symbol in which the items in source are concatenated with a separating dot. This is useful for q context names.

`sv````q
q)` sv `qalib`stat
`qalib.stat
```

When sv is used with an empty symbol as its first parameter and a symbol second parameter (source) whose first item is a file handle, the result is a symbol in which the items in source are concatenated with a separating /. This is useful for fully qualified path names.

`sv``/````q
q)` sv `:`q`tutorial`draft`3
`:/q/tutorial/draft/3
```

When sv is used with an empty symbol as its first parameter and a list of strings as the second parameter, it concatenates the strings, inserting a new line character after each.

`sv````q
q)` sv ("abc";"de")
"abc\nde\n"
```

When sv is used with a long first parameter (base) that is greater than 1, together with a second parameter of a simple list of place values expressed in base, the result is a long representing the converted base 10 value.

`sv````q
q)2 sv 101010b
42

q)10 sv 1 2 3 4 2
12342

q)256 sv 0x001092
4242
```

More precisely, the last version of sv evaluates the polynomial,

`sv`(d[n-1]*b exp n-1) + ... +d[0]

where d is the list of digits, n is the count of d, and b is the base.

Thus, we find,

```q
q)10 sv 1 2 3 11 2
12412

q)-10 sv 2 1 5
195
```

### A.94 svar¶

`svar`The unary svar returns the statistical standard variance of its numeric list argument.

`svar````q
q)svar 2 3 5 7
4.916667
```

It is equivalent to,

```q
{var[x]*count[x]%-1+count x}
```

Warning

The svar function was added in version 3.2 meaning that finance apps can no longer use a variable name svar for stress value at risk in this and later versions.

`svar``svar`### A.95 system¶

`system`The unary system takes a string argument and executes it as a q command, if recognized, or an OS command otherwise. This is convenient to execute q commands programmatically. The normal result of the command becomes the return value of the evaluation.

`system`Do not include the initial \ as you would when executing the command from the q session prompt.

`\````q
q)system "c"
25 80i
q)system "c 40 400"
q)system "c"
40 400i

q)system "cd /data"
q)system "pwd"
"/data"
```

### A.96 Take/Reshape #¶

`#`The binary operator # has several forms.

`#`When the first parameter of # is an integer atom count, it returns count items from the atom or list second parameter (source). The items are extracted from the head when source is positive and from the tail when count is negative. When count is zero, the result is an empty list of the same type as the first item in source. When count is greater than the count of source, items are repeatedly drawn from source as described until count items are obtained.

`#`The result of Take is always a list.

```q
q)2#10 20 30 40 50
10 20

q)-3#10 20 30 40 50
30 40 50

q)0#10 20 30 40 50
`long$()

q)3#42
42 42 42

q)10#10 20 30 40 50
10 20 30 40 50 10 20 30 40 50
```

The idiom 0# atom is a common way to create an empty list of the type of atom.

`0#````q
q)meta ([] a:0#0; b:0#`)
c| t f a
-| -----
a| j
b| s
```

Since a dictionary is ordered, take also applies to dictionaries and, by extension, to tables and keyed tables.

```q
q)2#`a`b`c!10 20 30
a| 10
b| 20

q)-2#([] c1:`a`b`c; c2:10 20 30)
c1 c2
-----
b 20
c 30

q)2#([k:`a`b`c] v:10 20 30)
k| v
-| --
a| 10
b| 20
```

Another form of Take has first parameter a list of keys and the second parameter a dictionary. The result is the sub-dictionary for these keys.

```q
q)`a`c#`a`b`c!10 20 30
a| 10
c| 30
```

Since a table is a column dictionary this form also applies to a table with a list of column names.

```q
q)`c1`c2#([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
c1 c2
-----
a 10
b 20
c 30
```

Using an anonymous table is a nifty way to generate a list of keys for takes with a keyed table.

```q
q)([] k:`a`c)#([k:`a`b`c] v:10 20 30)
k| v
-| --
a| 10
c| 30
```

Another form of Take has first parameter a list (r; c) of two non-negative integers and second parameter a list or atom (source). The result is a nested list with r items, each having c items, drawn sequentially from the beginning of source, repeatedly if necessary. Otherwise put, the result is an r by c array obtained by reshaping source.

```q
q)2 3#10 20 30 40 50
10 20 30
40 50 10
```

If in the previous form either r or c is null, the result is a nested list obtained by reshaping the second parameter into the specified number of rows or columns. This is generally a ragged array.

```q
q)2 0N#10 20 30 40 50
10 20 30
40 50
q)0N 3#10 20 30 40 50
10 20 30
40 5
```

### A.97 tan¶

`tan`The atomic tan takes a float and returns its mathematical tangent.

`tan````q
q)tan 0
0f

q)pi:3.141592653589793
q)tan pi
-1.224647e-16

q)tan pi%2
1.633124e+16

q)tan pi%4
1f

q)tan pi%8
0.4142136
```

The function tan is equivalent to

`tan````q
(sin x)%cos x
```

### A.98 til¶

`til`The unary til takes a non-negative long parameter (n) and returns a list of n consecutive longs starting with 0.

`til`The argument is not included in the result.

```q
q)til 4
0 1 2 3
q)count til 1000000
1000000
```

The result of til is always a list.

`til````q
q)til 0
`long$()
q)til 1
,0
```

To generate other regular numeric sequences, perform vector operations on the result of til.

`til````q
q)1+til 10
1 2 3 4 5 6 7 8 9 10
q)2*til 10
0 2 4 6 8 10 12 14 16 18
q)1+2*til 10
1 3 5 7 9 11 13 15 17 19
q).5*til 10
0 0.5 1 1.5 2 2.5 3 3.5 4 4.5
q)2015.01.01+til 5
2015.01.01 2015.01.02 2015.01.03 2015.01.04 2015.01.05
```

### A.99 trim¶

`trim`The unary trim takes a string argument and returns the result of removing leading and trailing blanks.

`trim````q
q)trim " abc "
"abc"
```

The function trim is equivalent to,

`trim````q
{ltrim rtrim x}
```

### A.100 ungroup¶

`ungroup`The unary ungroup flattens tables with nested columns that are the result of a select query that groups without aggregation, or of xgroup. Of course you can also apply it to tables with nested columns of the same form that are otherwise generated.

`ungroup``xgroup`The action of ungroup expands each key group, resulting in one row for each item in the nested columns. The fields in a result row are drawn from corresponding positions across the nested columns.

`ungroup`We use the distribution example.

```q
q)\l sp.q
+`p`city!(`p$`p1`p2`p3`p4`p5`p6`p1`p2;`london`london`london`london`london`lon..
(`s#+(,`color)!,`s#`blue`green`red)!+(,`qty)!,900 1000 1200
+`s`p`qty!(`s$`s1`s1`s1`s2`s3`s4;`p$`p1`p4`p6`p2`p2`p4;300 200 100 400 200 300)

q)sp
s  p  qty
---------
s1 p1 300
s1 p2 200
s1 p3 400
s1 p4 200
s4 p5 100
s1 p6 100
s2 p1 300
s2 p2 400
s3 p2 200

q)select s, qty by p from sp
p | s               qty
--| -------------------------------
p1| `s$`s1`s2       300 300
p2| `s$`s1`s2`s3`s4 200 400 200 200
p3| `s$,`s1         ,400
p4| `s$`s1`s4       200 300
p5| `s$`s4`s1       100 400
p6| `s$,`s1         ,100

q)ungroup select s, qty by p from sp
p  s  qty
---------
p1 s1 300
p1 s2 300
p2 s1 200
p2 s2 400
p2 s3 200
p2 s4 200
p3 s1 400
p4 s1 200
p4 s4 300
p5 s4 100
p5 s1 400
p6 s1 100

q)`p xgroup sp
p | s               qty
--| -------------------------------
p1| `s$`s1`s2       300 300
p2| `s$`s1`s2`s3`s4 200 400 200 200
p3| `s$,`s1         ,400
p4| `s$`s1`s4       200 300
p5| `s$`s4`s1       100 400
p6| `s$,`s1         ,100

q)ungroup `p xgroup sp
p  s  qty
---------
p1 s1 300
p1 s2 300
p2 s1 200
p2 s2 400
p2 s3 200
p2 s4 200
p3 s1 400
p4 s1 200
p4 s4 300
p5 s4 100
p5 s1 400
p6 s1 100
```

Tip

Observe that ungroup is not quite the inverse of a grouping operation. Since grouping sorts on the keys, a subsequent ungroup returns the original records sorted by the grouped column(s).

`ungroup``ungroup`### A.101 union¶

`union`The binary union returns the set theoretic union of its two list parameters. The result has the distinct items of the two parameters in order of their first appearance.

`union````q
q)1 1 2 3 union 2 2 3 4
1 2 3 4
q)"a good time" union "was had by all"
"a godtimewshbyl"
```

Since a table is a list of records, union can be used to perform the analogue of SQL UNION for tables having the same schemas.

`union``UNION````q
q)([] c1:`a`b`c; c2:10 20 30) union ([] c1:`c`d; c2:300 400)
c1 c2
------
a  10
b  20
c  30
c  300
d  400
```

Use uj for tables that do not have matching schemas – see §9.9.7.

`uj`### A.102 upper¶

`upper`The atomic upper takes a char, string or symbol argument and returns the result of converting any alpha characters to upper case.

`upper````q
q)upper `a
`A

q)upper `a`b
`A`B

q)upper "a Bc42De"
"A BC42DE"
```

### A.103 value¶

`value`The function value also vies for the title as the most overloaded q operator.

`value`The functions value and get are identical. Conventionally get is used with file I/O.

`value``get``get`When value is applied to a symbolic variable name, it returns the associated value.

`value````q
q)a:42
q)value `a
42
```

When applied to a dictionary, value returns the values of the dictionary. This includes the special case of the value table of a keyed table.

`value````q
q)value `a`b`c!10 20 30
10 20 30

q)kt:([k:`a`b`c] v:10 20 30)
q)value kt
_
```

A common idiom uses value to extract the columns lists from a table.

`value````q
q)value flip ([] c1:`a`b`c; c2:10 20 30)
a b c
10 20 30
```

When value is applied to an enumerated value, it de-enumerates it. This is handy when switching between kdb+ databases having different sym domains.

`value````q
q)sym:()
q)ev:`sym?`a`x`b`y`a`c`b
q)ev
`sym$`a`x`b`y`a`c`b
q)value ev
`a`x`b`y`a`c`b
```

Given a function, value returns the following list,

`value`(bytecode;parms;locals;(context;globals);constants[0];...;constants[n];defn)

```q
q)f:{[a;b]d::neg c:a*b+42;c+e}
q)value f
0xa0794178430316220b048100028276410004
`a`b
,`c
``d`e
42
"{[a;b]d::neg c:a*b+42;c+e}"q)sym:()
```

Given an alias (aka “view”), value returns the list

`value`(cached value;parse tree;dependencies;definition)

When the evaluation of the expression in the alias is as yet deferred, the cached value is ::.

`::````q
q)a:42
q)b::a+1            / evaluation is deferred
q)value `.[`b]      / inspect global context
::
(+;`a;1)
,`a
"a+1"

q)b                 / force evaluation
43
q)value `.[`b]
43
(+;`a;1)
,`a
"a+1"
```

Given a projection, value returns a list containing the function body followed by the supplied arguments.

`value````q
q)f:{x+y+z}
q)value f[2;3;]
{x+y+z}
2
3
::
q)value +[2]
_
```

Given a function that is the result of applying a k iterator, value returns the argument to the iterator, viewing the latter as a higher-order function.

`value````q
q)value (+/)
+
q)value (+')
+
q)f:{x+y}
q)value (f')
{x+y}
```

The final and most powerful form of value is that it is the q interpreter itself. It will evaluate a string as if it had been typed at the console.

`value````q
q)value "6*7"
42
q)value "{x*x} til 10"
0 1 4 9 16 25 36 49 64 81
```

It will also evaluate a parse tree – i.e., a (potentially) nested list of functions followed by arguments. If the function is specified by name, that name is resolved first.

```q
q)value (*; 6; 7)
42
q)f:{x*y}
q)value (`f; 6; 7)
42
```

Warning

This use of the value function is a powerful feature that allows q code to be written and executed on the fly. This can expose your program to potential attack unless done very carefully. If abused, it can quickly lead to unmaintainable code. (The spellchecker suggests "unmentionable" instead of "unmaintainable." How did it know?)

`value`### A.104 var¶

`var`The aggregate var takes a numeric list and returns the mathematical variance of the items as a float.

`var````q
q)var 42 45 37 38
13.66667
```

The function var is equivalent to

`var````q
{(avg[x*x]) - (avg[x])*(avg[x])}
```

### A.105 vs¶

`vs`The binary vs – ”vector from scalar” – has several versions. It is usually written infix for ease of reading.

`vs`The first form takes a char as its first parameter and a string (source) as its second parameter. It returns a list of strings containing the tokens of source as delimited by the specified char.

```q
q)" " vs "Now is the time "
"Now"
"is"
"the"
"time"
""
```

You probably want to trim the input in the above example.

When vs is used with an empty symbol as its first parameter and a symbol second parameter (source) containing dots, it returns a simple symbol list obtained by splitting source along the dots.

`vs````q
q)` vs `qalib.stat
`qalib`stat
```

When vs is used with an empty symbol as its first parameter and a symbol representing a fully qualified file name as the second parameter, it returns a simple list of symbols in which the first item is the path and the second item is the file name. Note that in this usage, vs is not quite the inverse of sv.

`vs``vs``sv````q
q)` vs `qalib.stat
`qalib`stat
```

When vs is used with a null of binary type as the first parameter and an value of integer type as the second parameter (source), it returns a simple list whose items comprise the digits of the corresponding binary representation of source.

`vs````q
q)0x00 vs 4242
0x0000000000001092

q)10h$0x00 vs 8151631268726338926j
"q is fun"

q)0b vs 42
0000000000000000000000000000000000000000000000000000000000101010b
```

The 0b version can be used to display the internal representation of special values.

`0b````q
q)0b vs 0W
0111111111111111111111111111111111111111111111111111111111111111b
q)0b vs -0W
1000000000000000000000000000000000000000000000000000000000000001b
```

### A.106 wavg¶

`wavg`The binary wavg takes two numeric lists of the same count and returns the float average of the items in the second parameter, weighted by the items of the first.

`wavg````q
q)1 2 3 4 wavg 500 400 300 200
300f
```

The function wavg is equivalent to,

`wavg````q
{(sum x*y)%sum x}
```

It is possible to apply wavg to a nested list provided all sublists of both parameters conform. In this context, the result conforms to the sublists and the weighted average is calculated recursively across the sublists.

`wavg````q
q)(1 2;3 4) wavg (500 400; 300 200)
350 266.6667
```

### A.107 where¶

`where`The unary where has two forms.

`where`The first form returns the indices of 1b in a boolean list, or the keys associated to 1b in a dictionary with boolean values.

`1b``1b````q
q)where 101010b
0 2 4
q)where `a`b`c`d`e`f!101010b
`a`c`e
```

This is useful when the boolean mapping is the result of a predicate.

```q
q)where ","="First, second, third"
5 13

q)@[L; where L=","; :; "\t"]
"First\t second\t third"

q)where null `a`b`c`d`e`f!1 0n 3 4 0n 6
`b`e
```

When the argument of where is a list of non-negative long (c), the result is a list obtained by catenating c[i] copies of i, for each item in c.

`where````q
q)where 2 1 3
0 0 1 2 2 2
q)where 2 1 0 3 2
0 0 1 3 3 3 4 4
q)where 4#1
0 1 2 3
```

Zen Moment

The second form of where generalizes the form on a boolean list.

`where`### A.108 within¶

`within`The binary within is atomic in its first parameter (source) and takes a second parameter that is a list (b;e) that are comparable. It returns a boolean representing whether source is greater than or equal to b and less than or equal to e. It will do type promotion on arguments.

`within````q
q)3 within 2 5
1b

q)(til 7)within 2 5
0011110b

q)"c" within "az"
1b

q)2015.03.14 within 2015.02.01 2015
1b

q)`ab within `a`b
1b

q)100 within "aj"
1b
```

Ensure the items in the second argument are in increasing order or you will get no matches.

### A.109 wsum¶

`wsum`The binary wsum takes two numeric lists of the same count and returns the float sum of the items of second parameter weighted by the items of the first.

`wsum````q
q)1 2 3 4 wsum 500 400 300 200
3000f
```

The function wsum is equivalent to,

`wsum````q
{sum x*y}
```

It is possible to apply wsum to a nested list provided all sublists of both arguments conform. In this case, the result conforms to the sublists and the weighted sum is calculated recursively across the sublists.

`wsum````q
q)(1 2;3 4) wsum (500 400;300 200)
1400 1600
```

### A.110 xbar¶

`xbar`The uniform binary xbar takes first parameter a non-negative numeric atom (width) and returns the largest integral multiple of width that is less than or equal to its second parameter. Visually, the result is the left endpoint of the band of width width that contains the second parameter. Since xbar is atomic in the second parameter, this is an effective way to bin the items in a numeric list.

`xbar``xbar````q
q)1 xbar 0 .5 1 1.5 2 2.5 3 3.5
0 0 1 1 2 2 3 3f

q)1.5 xbar 0 .5 1 1.5 2 2.5 3 3.5
0 0 0 1.5 1.5 1.5 3 3

q)2 xbar 0 .5 1 1.5 2 2.5 3 3.5
0 0 0 0 2 2 2 2f
```

Since a q month is actually the count of months since the millennium, you can use xbar to determine quarters. Recall that a month is equal to the first day of the month.

`xbar````q
q)`date$3 xbar `month$2015.11.19 / beginning of that quarter
2015.10.01
q)`date$3+3 xbar `month$2015.11.19 / beginning of next quarter
_
q)-1+`date$3+3 xbar `month$2015.11.19 / end of that quarter
_
```

### A.111 xprev¶

`xprev`The binary xprev takes a long as its first parameter (shift) and shifts the list in its second parameter by shift positions. When shift is 0 or positive, the shift is forward; otherwise it is backward. The beginning – respectively end – of the list is filled with shift null (or empty) items.

`xprev````q
q)2 xprev 10 20 30 40 50
0N 0N 10 20 30

q)-2 xprev 10 20 30 40 50
30 40 50 0N 0N

q)2 xprev (1 2 3; 4 5; enlist 6; 7 8)
`long$()
`long$()
1 2 3
4 5
```

### A.112 xrank¶

`xrank`The binary xrank has first parameter a positive long (n) and is uniform in its second parameter (source). It returns a list of long containing the n-quantile into which each item of source falls, considering source as a distribution. For example, choosing n to be 4 returns quartiles and 100 yields percentiles.

`xrank````q
q)4 xrank 30 10 40 20 90
1 0 2 0 3

q)100 xrank 200?1000.
35 14 30 96 19 68 39 21 91 73 27 79 30 66 27 79 75 8 99 84 84 77 ..
```

