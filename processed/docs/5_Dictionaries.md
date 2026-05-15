# 5 Dictionaries

# 5. Dictionaries¶

## 5.0 Overview¶

A dictionary is a mapping defined by an explicit association between a key list and value list. The two lists must have the same count and the key list should be a unique collection. While general lists can be used to create a dictionary, many dictionaries involve simple lists of keys.

## 5.1 Dictionary Basics¶

A dictionary is an association between a list of keys and a list of values. Logically it can also be considered as key-value pairs but it is stored physically as a pair of lists.

### 5.1.1 Definition¶

A dictionary, sometimes called an association, is a mapping defined by positional correspondence between a domain list of keys and a co-domain list of values. Dictionary creation uses the ! operator â read "bang" â in contrast with the syntactic form for lists.

`!`keys!values

All dictionaries have type 99h.

`99h`Observe that the q console displays a dictionary as an I/O table.

```q
q)10 20 30!1.1 2.2 3.3
10| 1.1
20| 2.2
30| 3.3
q)`a`b`c!100 200 300
_
```

Neither the keys nor values need be simple lists but the keys should conform else odd things will happen during operations.

```q
q)(`Arthur`Dent; `Zaphod`Beeblebrox; `Ford`Prefect)! 100 42 150
Arthur Dent      | 100
Zaphod Beeblebrox| 42
Ford Prefect     | 150
q)1001 1002 1003!(`Arthur`Dent; `Zaphod`Beeblebrox; `Ford`Prefect)
_
```

A dictionary can be decomposed into its key and value lists using the primitives key and value. The common number of keys or values is returned by count.

`key``value``count````q
q)d:`a`b`c!100 200 300
q)key d
_
q)value d
_
q)count d
_
```

Although q does not enforce uniqueness for keys (a historical accident) a dictionary does provide a unique output value for each input value. Indeed, only the first key occurrence is seen during lookup.

When you know that the keys are unique you can apply the `u# attribute to the keys.

``u#`This will effectively cause the dictionary to be a hash table with the attendant improvement in lookup speed over the default linear lookup.

q)(`u#`a`b`c)!10 20 30
_

```q
q)(`u#`a`b`c)!10 20 30
_
```

The order of the items in the key and value lists is significant, just as positional order is significant for lists. Although the I/O assignments and the associated mappings are equivalent regardless of order, differently ordered dictionaries are not identical.

```q
q)(`a`b`c!10 20 30)~`a`c`b!10 30 20
0b
```

### 5.1.2 Empty and Singleton Dictionaries¶

You can create a general empty dictionary using empty key and value lists.

```q
q)()!()
_
```

You can create a typed empty dictionary using typed empty key and value lists.

```q
q)(`symbol$())!`float$()
_
```

Because both the keys and values are required to be lists, you must enlist atoms for a singleton dictionary.

q)(enlist `x)!enlist 42
x| 42

```q
q)(enlist `x)!enlist 42
x| 42
```

The following is not a singleton dictionary. This is a common mistake of qbies.

```q
q)`x!42
`x!42i
```

For those who care, it is actually an enumerated value for a link column.

### 5.1.3 Lookup¶

To find the output value corresponding to an input key, we look up the key. Lookup uses the same notation as indexing into a list: brackets or prefix syntax.

```q
q)d:`a`b`c!10 20 30
q)d[`a]
10
q)d `b
_
```

Similar to list indexing, lookup of a value not in the key list of a dictionary results in a null whose type is that of the initial item in the value list.

```q
q)d[`x]
_
```

As with lists, key lookup in a dictionary is extended item-wise to a simple list of keys.

```q
q)d[`a`c]
_
q)ks:`a`c
q)d ks
_
```

### 5.1.4 Reverse Lookup with Find ?¶

`?`Recall that the Find  operator ? on a list returns the index of the first occurrence of the right operand.

`?````q
q)10 20 30 10 40?10
0
```

Essentially it inverts the positional retrieval mapping. Extending this concept to dictionaries means reversing the key-to-value mapping of lookup. That is, ? on a dictionary performs reverse lookup by mapping a value item to the first key associated to it.

`?````q
q)d:`a`b`c`a!10 20 30 10
q)d?10
`a
```

The result of Find on an argument not in the value list is a null whose type matches the initial item in the key list.

```q
q)d:`a`b`c`a!10 20 30 10
q)d?40
`
```

### 5.1.5 Dictionary vs. List¶

It is possible to create a dictionary that seemingly behaves very much like an equivalent list by making the positional retrieval explicit.

```q
q)L:10 20 30
q)d:0 1 2!10 20 30
q)L 0
10
q)d 0
10
q)L 1 2
20 30
q)d 1 2
20 30
d~L
_
```

The two are not the same. A major difference is that dictionaries can be extended via assignment and a list cannot.

This dictionary version of lists provides a compact and efficient representation of sparse lists.

q)d1:0 100 500000!10 20 30
q)d2:0 99 1000000!100 200 300
q)d1+d2
0      | 110
100    | 20
500000 | 30
99     | 200
1000000| 300

The implementation is left as an exercise for the reader.

```q
q)d1:0 100 500000!10 20 30
q)d2:0 99 1000000!100 200 300
q)d1+d2
0      | 110
100    | 20
500000 | 30
99     | 200
1000000| 300
```

### 5.1.6 Non-unique Keys and Values¶

As mentioned previously, non-unique keys are tolerated but lookup only sees the first occurrence.

```q
q)ddup:`a`b`a`c!10 20 30 20
q)ddup[`a]
10
```

Reverse lookup works properly for non-unique keys but will only see the first non-unique value.

```q
q)ddup?30
`a
q)ddup?20
`b
```

You can find all keys mapping to a given value as follows.

```q
q)where ddup=20
`b`c
```

### 5.1.7 Non-simple Keys and Values¶

Neither the key nor value lists of a dictionary are required to be atoms or uniform. Either can be nested lists.

```q
q)d:(`a`b; `c`d`e; enlist `f)!10 20 30
q)d `f
30
q)d?20
`c`d`e
q)d:`a`b`c!(10 20; 30 40 50; enlist 60)
q)d `b
30 40 50
q)d?30 40 50
`b
q)d?enlist 60
_
```

Be forewarned that an irregular key or value list â i.e., items do not conform â will confound lookup or reverse lookup, respectively.

```q
q)dwhackey:(1 2; 3 4 5; 6; 7 8)!10 20 30 40 / atom 6 is whack
q)dwhackey 1 2
10
q)dwhackey 6
0N
q)dwhackval:10 20 30 40!(1 2; 3 4 5; 6; 7 8) / atom 6 is whack
q)dwhackval?3 4 5
20
q)dwhackval?6
_
```

The observed behavior is that lookup in either direction fails at the first item of different shape.

## 5.2 Operations on Dictionaries¶

### 5.2.1 Amend and Upsert¶

As with lists, the items of a dictionary can be modified via assignment to a key.

```q
q)d:`a`b`c!10 20 30
q)d[`b]:42
q)d
_
```

In contrast to lists, dictionaries can be extended via assignment.

```q
q)d:`a`b`c!10 20 30
q)d[`x]:42
q)d
_
```

Letâs examine more closely this capability to modify or extend a dictionary via key-value assignment. Given a dictionary d, a value k that matches the type of the keys of d and a value v that matches the type of the values of d, consider the assignment

`d``k``d``v``d````q
d[k]:v
```

It updates the existing key-value association when k is in key d, or inserts (appends) a new key-value association when k is not in key d.

`k``d``k``d````q
q)d:`a`b`c!10 20 30
q)d[`b]:42 / update
q)d[`x]:100 / insert
q)d
_
```

This update/insert behavior is called upsert semantics. Because tables and keyed tables are dictionaries, upsert semantics pervade kdb+.

### 5.2.2 Extracting a Sub-Dictionary¶

Dictionary lookup on a key or a list of keys returns the associated values. It is possible to extract both keys and values using an overload of the Take operator #. The left operand is a list of keys, the right operand is the source dictionary and the result is a sub-dictionary for the specified keys. Viewed as a mapping, the result is the original restricted to the specified keys.

`#````q
q)d:`a`b`c!10 20 30
q)d `a`c
_
q)`a`c#d
a| 10
c| 30
q)(enlist `c)#d
_
```

In the event of duplicate keys, only the first is extracted.

```q
q)ddup:`a`b`a`c!10 20 30 20
q)`a`c#ddup
a| 10
c| 20
```

This operation works with non-simple keys.

```q
q)dns:(`arthur`dent; `ford`prefect; `zaphod`beebelbrox)!100 150 42
q)(`arthur`dent; `zaphod`beebelbrox)#dns
arthur dent      | 100
zaphod beebelbrox| 42
```

### 5.2.3 Removing Entries¶

The adjoint operation to # is _, which removes key-value pairs. Its syntax is the same â i.e., a list of keys as the left operand and a dictionary on the right â and it returns the dictionary obtained by removing the key-value pairs for the specified keys.

`#``_`Whitespace is required to the left of _ if the left operand is a variable since _ is a valid terminal name character.

`_``_`q)d:`a`b`c!10 20 30
q)`a`c _ d
b| 20
q)(enlist `b) _ d
_

```q
q)d:`a`b`c!10 20 30
q)`a`c _ d
b| 20
q)(enlist `b) _ d
_
```

Observe that all occurrences of a key are removed and that attempting to remove a key that does not exist has no effect.

```q
q)d:`a`b`c`a!10 20 30 40
q
q)`a`c _ d
_
q)`x`a _ d
_
```

Removing all the entries in a dictionary leaves a dictionary with empty key and value lists of the appropriate types. The console will not display the empty dictionary but you can force the display with .Q.s1. In this case we get typed empty lists.

`.Q.s1````q
q)d:`a`b`c!10 20 30
q)`a`b`c _ d
q).Q.s1 `a`b`c _ d
"(`symbol$())!`long$()"
```

The binary keyword cut is the same as this overload of _ on a dictionary.

`cut``_````q
q)`a`c cut d
_
```

There is another overload of _ with a dictionary on the left and a single key on the right that deletes just that key, which is seldom used.

`_````q
q)d _ `b
a| 10
c| 30
```

### 5.2.4 Basic Operations on Dictionaries¶

Because a dictionary is a map, it can be composed with another map to obtain a dictionary whose key-value association is the composite of the original and the new map. Another way to say this is that applying a function to a dictionary effectively applies it to the value list. Unary functions are straightforward.

d:`a`b`c!10 20 30 composed with neg:

`d:`a`b`c!10 20 30``neg``a      |->  10    |->     -10
`b      |->  20    |->     -20
`c      |->  30    |->     -30
q)d:`a`b`c!10 20 30
q)neg d
a| -10
b| -20
c| -30

```q
`a      |->  10    |->     -10
`b      |->  20    |->     -20
`c      |->  30    |->     -30
```

```q
q)d:`a`b`c!10 20 30
q)neg d
a| -10
b| -20
c| -30
```

And similarly for other unary maps.

```q
q)2*d
_
q)d=20
_
q)f:{x*x}
q)f d
_
```

When the domains of two dictionary maps are identical, performing a binary operation is also straightforward: do it value-with-value along the keys. For example, to add two dictionaries with a common list of keys, add their corresponding values.

```q
q)d1:`a`b`c!1 2 3
q)d2:`a`b`c!10 20 30
q)d1+d2
_
```

What should happen when the domains lists are not identical? First, the domain of the resulting dictionary is the union of the domains â i.e., the union of the key lists. For items in the common domain â i.e., the intersection of the keys lists, we already know we should apply the operation value-with-value.

So the question becomes: what to do on non-common key items? The answer: Nothing! Since there really isnât anything to do where there is no matching key on the other side, simply carry the value to the result. In essence, this implies using the identity element for the operation in place of the missing value.

```q
q)d1:`a`b`c!1 2 3
q)d2:`b`c`d!20 30 40
q)d1+d2
a| 1
b| 22
c| 33
d| 40
```

### 5.2.5 Join ,¶

`,`The Join operator , merges two dictionaries. Clearly on disjoint keys the corresponding values should be carried over.

`,````q
q)d1:`a`b`c!10 20 30
q)d2:`x`y!40 50
q)d1,d2
a| 10
b| 20
c| 30
x| 40
y| 50
```

On common keys, since q is right-to-left, the value of the right operand prevails over that of the left.

```q
q)d1:`a`b`c!10 20 30
q)d2:`a`b`c!100 200 300
q)d1,d2
a| 100
b| 200
c| 300
```

Lo and behold, the entire right operand has been upserted into the left operand.

```q
q)d1:`a`b`c!10 20 30
q)d2:`c`d!300 400
q)d1,d2
a| 10
b| 20
c| 300
d| 400
```

This is equivalent to each key-value association being upserted into the left

```q
q)d1:`a`b`c!10 20 30
q)d1[`c]:300
q)d1[`d]:400
q)d1
_
```

Observe that Join is not commutative: order matters.

```q
q)d1:`a`b`c!10 20 30
q)d2:`c`d!300 400
q)d1,d2
_
q)d2,d1
_
```

### 5.2.6 Coalesce ^¶

`^`The Coalesce operator is related to , in that it employs upsert semantics to merge two dictionaries. The difference is that right values prevail over left except for nulls in the right.

`,````q
q)d1:`a`b`c!10 0N 30
q)d2:`b`c`d!200 0N 400
q)d1^d2
a| 10
b| 200
c| 30
d| 400
```

### 5.2.7 Arithmetic And Relational Operations¶

Following the general pattern, when an arithmetic operation is performed on dictionaries, the operation is performed on the common domain elements and the identity element implied elsewhere. Of course the value lists must have proper types for the operation.

```q
q)d1:`a`b`c!10 20 30
q)d2:`b`c`d!200 300 400
q)d1+d2
a| 10
b| 220
c| 330
d| 400
```

For equality and comparison operations on dictionaries, the indicated operation is performed over the common keys. On disjoint keys, the appropriate null is effectively substituted for missing values. Due to all nulls being equal, a null value on a disjoint key will report 1b under equality test.

`1b````q
q)(`a`b`c!10 20 30)=`b`c`d!20 300 400
a| 0
b| 1
c| 0
d| 0
q)(`a`b`c!0N 20 30)=`b`c`d!20 300 0N
a| 1
b| 1
c| 0
d| 1
q)(`a`b`c!10 20 30)<`b`c`d!20 300 400
a| 0
b| 0
c| 1
d| 1
```

## 5.3 Column Dictionaries¶

Column dictionaries are the foundation for tables.

### 5.3.1 Definition and Terminology¶

A very useful type of dictionary maps a simple list of symbols to a rectangular list of lists.

```q
q)`c1`c2!(`a`b`c; 10 20 30)
c1| a b c
c2| 10 20 30
```

Interpreting each symbol as a name and the corresponding list as a vector of column values, we call such a list a column dictionary. A general column dictionary has the form,

c1...cn!(v1;...;vn)

where each ci is a symbol and the vi are lists with common length (count). Such a dictionary associates the name ci with the value list vi.

The type of the column named by ci is the type of its associated value list vi. Often the vi are all simple lists â i.e., each column is a vector of atoms of uniform type.

### 5.3.2 Simple Example¶

Letâs make a column dictionary that holds the names of famous galactic travelers and their IQ scores.

```q
q)travelers:`name`iq!(`Dent`Beeblebrox`Prefect;42 98 126)
q)travelers
name| Dent Beeblebrox Prefect
iq  | 42   98         126
```

In this dictionary, the values for the name column are,

```q
q)travelers[`name]
`Dent`Beeblebrox`Prefect
```

Since this column is a list, we can index into it.

```q
q)travelers[`name][1]
`Beeblebrox
q)travelers[`iq][2]
126
```

In general, whenever we encounter repeated indexing in q we can alternately write it as indexing at depth.

```q
q)travelers[`name; 1]
`Beeblebrox
q)travelers[`iq; 2]
126
```

Thus travelers can be considered a two-dimensional entity indexed by name in the first slot.

### 5.3.3 General Case¶

For a general column dictionary defined as,

dc:c1...cn!(v1;...;vn)

the ith element of column cj is retrieved by,

dc[cj][i]

Or equivalently,

dc[cj;i]

The indexing-at-depth notation suggests thinking of the column dictionary as a two-dimensional data structure, indexed by column name and column position. Alternatively, it is a positional map of two variables with symbolic name in the first parameter and column index in the second.

Specifying just an index in the second column is interesting. It essentially retrieves a 'slice' of the original dictionary across that index. This slice is itself a dictionary. In our simple example of the previous section,

```q
q)travelers[; 2]
name| `Prefect
iq  | 126
```

Letâs summarize.

- A column dictionary can be viewed as a two-dimensional entity with retrieval by name in the first dimension and by position within column in the second.
- Specifying only a name retrieves the corresponding column, which can then be indexed by position.
- Specifying only a column index retrieves a slice dictionary that maps the names to the associated values across that column index.

This data structure has potential but there is a hitch: the indexing is backwards. That is, columns are retrieved in the first slot. Normally in two-dimensional data structures, columns are retrieved in the second slot. We shall see shortly how to fix this.

### 5.3.4 Column Dictionary with a Single Column¶

Recall that every dictionary is constructed from a list of keys and a list of values. A column dictionary is further required to have a rectangular list of values â i.e., a list of lists of the same length.

Thus in order to make a column dictionary with a single column, we need to enlist the single key and enlist the single column list.

```q
q)dc1:(enlist `c)!enlist 10 20 30
q)dc1
c| 10 20 30
```

As noted previously, the following isnât even a valid dictionary, let alone a column dictionary.

q)`c!1 2 3
_

```q
q)`c!1 2 3
_
```

## 5.4 Flipping a Column Dictionary¶

No, this isnât what you do to a dictionary that cuts you off in traffic. Recall that a rectangular nested list can be transposed with the flip operator.

`flip````q
q)L:(10 20 30; 100 200 300)
q)L
10 20 30
100 200 300
q)flip L
10 100
20 200
30 300
```

A side effect of transposing the nested list data is that it reverses the slots for indexing at depth.

```q
q)L:(10 20 30; 100 200 300)
q)M:flip L
q)L[0;0]
10
q)L[0;1]
20
q)L[0;2]
30
q)M[0;0]
10
q)M[1;0]
20
q)M[2;0]
30
```

Recall that applying flip to a nested list creates a copy of the original list in which the data has been reorganized. When a column dictionary is considered as a two-dimensional entity it is rectangular, since its column lists all have the same count. The console display shows this.

`flip````q
q)`c1`c2!(`a`b`c; 10 20 30)
c1| a b c
c2| 10 20 30
```

Thus it makes sense to transpose this rectangular data structure. Observe that the console display is exactly transposed.

```q
q)dc:`c1`c2!(`a`b`c; 10 20 30)
q)dc
c1| a b c
c2| 10 20 30
q)t:flip dc
q)t
c1 c2
-----
a 10
b 20
c 30
```

We expect the transposed entity to have the slots reversed for indexing at depth.

```q
q)dc[`c1; 0]
`a
q)dc[`c1; 1]
`b
q)dc[`c1; 2]
`c
q)t[0; `c1]
`a
q)t[1; `c1]
`b
q)t[2; `c1]
`c
```

VoilÃ ! We have fixed the indexing issue with our column dictionaries. Columns are now indexed in the second parameter. Letâs translate the other aspects of the column dictionary retrieval into the transposed setting.

```q
q)dc[`c1;]
`a`b`c
q)t[;`c1]
`a`b`c
q)dc[; 0]
c1| `a
c2| 10
q)t[0;]
c1| `a
c2| 10
q)dc[`c2; 1]
20
q)t[1; `c2]
20
```

For the moment, forget the origin of t as a transposed column dictionary and examine it in isolation.

`t````q
q)t[;`c1]
`a`b`c
q)t[0;]
c1| `a
c2| 10
q)t[1; `c2]
20
```

Letâs look more closely at the section dictionaries that are now indexed in the first slot. Recall that we are permitted (although not encouraged) to omit trailing semi-colons for elided indices.

```q
q)t[0]
c1| `a
c2| 10
q)t[1]
c1| `b
c2| 20
q)t[2]
c1| `c
c2| 30
```

If you were to meet such a beast out of context, you would say that it is a list of these funky section dictionaries.

We summarize our findings about the transpose of a column dictionary,

- It is a two-dimensional data structure that uses an integer index in the first slot and a column name symbol in the second slot.
- Specifying only an integer in the first slot retrieves a section dictionary across that index
- Specifying only a column name in the second slot retrieves that column.
- Specifying both an integer and a column name retrieves the "field" at that index in that column.
- It can be viewed as a list of section dictionaries.

We make two more observations about transposed column dictionaries. In general, if you transpose something twice, you get back the original.

```q
q)dc~flip flip dc
1b
q)dc~flip t
1b
```

We restate this as:

- The flip of a transposed column dictionary is a (the original!) column dictionary

Unlike the case of transposing rectangular lists, transposing a column dictionary does not physically re-arrange data.

A transposed column dictionary is stored in column order.

Instead, the result is a logical adjustment â meaning that the slots in indexing at depth are reversed.

