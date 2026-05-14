# Chapter 7: Transforming Data - Complete Content

## 7.1 Types

### 7.1.1 Basic DataTypes

The chapter presents a comprehensive type table with three equivalent representations:

| Type | Symbol | Char | Num |
|------|--------|------|-----|
| boolean | `boolean | B | 1h |
| guid | `guid | G | 2h |
| byte | `byte | X | 4h |
| short | `short | H | 5h |
| int | `int | I | 6h |
| long | `long | J | 7h |
| real | `real | E | 8h |
| float | `float | F | 9h |
| char | `char | C | 10h |
| symbol | ` | S | 11h |
| timestamp | `timestamp | P | 12h |
| month | `month | M | 13h |
| date | `date | D | 14h |
| datetime | `datetime | Z | 15h |
| timespan | `timespan | N | 16h |
| minute | `minute | U | 17h |
| second | `second | V | 18h |
| time | `time | T | 19h |

### 7.1.2 The `type` Operator

The binary function returns data type as a short integer. "A feature of q" is that atom types are negative while simple list types are positive:

```q
q)type 42
-7h
q)type 10 20 30
7h
q)type 98.6
-9h
q)type 1.1 2.2 3.3
9h
q)type `a
-11h
q)type `a`b`c
11h
q)type "z"
-10h
q)type "abc"
10h
```

Infinities and nulls retain their respective types:

```q
q)type 0W
-7h
q)type 0N
-7h
q)type -0w
-9h
q)type 0n
-9h
q)type `
-11h
```

General lists have type `0h`:

```q
q)type (42h; 42i; 42j)
0h
q)type (1 2 3; 10 20 30)
0h
q)type ()
0h
```

Dictionaries have type `99h`, keyed tables also `99h`:

```q
q)type (`a`b`c!10 20 30)
99h
q)type ([k:`a`b`c] v:10 20 30)
99h
```

Tables have type `98h`:

```q
q)type ([] c1:`a`b`c; c2:10 20 30)
98h
```

### 7.1.3 Type of a Variable

Since q is dynamically typed, variables have the type of their currently assigned value:

```q
q)a:42
q)type a
-7h
q)a:"abc"
q)type a
10h
```

Variables are associations in a global dictionary. The default global dictionary is named `` `. ``:

```q
q)get `.
a| 42
q)a:"abc"
q)get `.
a| "abc"
```

Global variables use ordinary q dictionary structures, illustrating that "q eats its own dog food."

---

## 7.2 Cast

The binary operator `$` transforms values between compatible types. It operates at runtime and is atomic in both operands. The left operand specifies target type using three formats:

- Positive numeric short value
- Char type value  
- Type name symbol

### 7.2.1 Casts that Widen

No information loss occurs when target type is wider:

```q
q)7h$42i
42
q)6h$42
42i
q)9h$42
42f
```

Using type char notation:

```q
q)"j"$42i
42
q)"i"$42
42i
q)"f"$42
42f
```

Using symbolic type names:

```q
q)`int$42
42i
q)`long$42i
42
q)`float$42
42f
```

### 7.2.2 Casts across Disparate Types

Underlying numeric values allow casts between seemingly different types:

Char values are ASCII positions (0-255):

```q
q)`char$42
"*"
q)`long$"\n"
10
```

Date values count days from the millennium:

```q
q)`date$0
2000.01.01
q)`int$2001.01.01
366
```

Timespan values count nanoseconds from midnight:

```q
q)`long$12:00:00.0000000000
43200000000000
q)`timespan$0
00:00:00.000000000
```

### 7.2.3 Casts that Narrow

Information loss occurs in narrowing conversions:

```q
q)`long$12.345
12
q)`short$123456789
32767h
```

Boolean casting follows C philosophy (zero is false, non-zero is true):

```q
q)`boolean$0
0b
q)`boolean$0.0
0b
q)`boolean$123
1b
q)`boolean$-12.345
1b
```

Extracting constituents from temporal types:

```q
q)`date$2015.01.02D10:20:30.123456789
2015.01.02
q)`year$2015.01.02
2015i
q)`month$2015.01.02
2015.01m
q)`mm$2015.01.02
1i
q)`dd$2015.01.02
2i
q)`hh$10:20:30.123456789
10i
q)`minute$10:20:30.123456789
10:20
q)`uu$10:20:30.123456789
20i
q)`second$10:20:30.123456789
10:20:30
q)`ss$10:20:30.123456789
30i
```

This approach is preferred over dot notation since the latter doesn't work inside functions.

### 7.2.4 Casting Integral Infinities

When integral infinities cast to wider integer types, their bit patterns reinterpret as finite values:

```q
q)`int$0Wh
32767i
q)`int$-0Wh
-32767i
q)`long$0Wi
2147483647
q)`long$-0Wi
-2147483647
```

### 7.2.5 Coercing Types

Casting enables type-safe assignment to simple lists. Direct assignment fails if types don't match:

```q
q)L:10 20 30 40
q)L[1]:42h
'type
q)L,:43h
'type
```

Cast the value to match the list's type:

```q
q)L[1]:(type L)$42h
q)L,:(type L)$43h
q)L
10 20 30 40 42 43
```

Simple list types are positive, enabling this idiom.

### 7.2.6 Cast is Atomic

Cast is atomic in the right operand:

```q
q)"i"$10 20 30
10 20 30i
q)`float$(42j; 42i; 42j)
42 42 42f
```

Cast is atomic in the left operand:

```q
q)`short`int`long$42
42h
42i
42
q)"ijf"$98.6
99i
99
98.6
```

Cast is atomic in both operands simultaneously:

```q
q)"ijf"$10 20 30
10i
20
30f
```

---

## 7.3 Data to and from Text

A q string is a simple list of char. The `$` operator handles conversion between values and text representations, though technically this differs from true casting since values are only implicit in text.

### 7.3.1 Data to Strings

The `string` function converts any q entity to a text representation:

```q
q)string 42
"42"
q)string 4
,"4"
q)string 42i
"42"
q)a:2.0
q)string a
,"2"
q)f:{x*x}
q)string f
"{x*x}"
```

Key features: results are always char lists (never single char); results contain no type indicators; applying to actual strings may not yield expected results.

The function is "pseudo-atomic"—it recurses into arguments:

```q
q)string 1 2 3
,"1"
,"2"
,"3"
q)string "string"
,"s"
,"t"
,"r"
,"i"
,"n"
,"g"
q)string (1 2 3; 10 20 30)
,"1" ,"2" ,"3"
"10" "20" "30"
```

Converting symbol lists to strings:

```q
q)string `Life`the`Universe`and`Everything
,"L" ,"t" ,"U" ,"a" ,"E"
"i"  "h"  "n"  "n"  "v"
"f"  "e"  "i"  "d"  "e"
"e"  ""   "v"  ""   "r"
```

### 7.3.2 Creating Symbols from Strings

Cast char or string to symbol using `` `$ ``. This is the **only** method for symbols with embedded blanks or special characters:

```q
q)`$"abc"
`abc
q)`$"Hello World"
`Hello World
```

The `` `symbol$ `` form generates an error.

Including special characters requires escaping:

```q
q)`$"Zaphod \"Z\""
`Zaphod "Z"
q)`$"Zaphod \n"
`Zaphod`
```

Leading and trailing whitespace are trimmed:

```q
q)string `$" abc "
"abc"
```

The unary `` `$ `` is atomic on lists:

```q
q)`$("Life";"the";"Universe";"and";"Everything")
`Life`the`Universe`and`Everything
```

### 7.3.3 Parsing Data from Strings

Using **uppercase** type char as the left operand invokes parse mode. If parsing fails, a null of the target type is returned instead of an exception:

```q
q)"J"$"42"
42
q)"F"$"42"
42f
q)"F"$"42.0"
42f
q)"I"$"42.0"
0Ni
q)"I"$" "
0Ni
```

Date parsing is flexible regarding format:

```q
q)"D"$"12.31.2014"
2014.12.31
q)"D"$"12-31-2014"
2014.12.31
q)"D"$"12/31/2014"
2014.12.31
q)"D"$"12/1/2014"
2014.12.01
q)"D"$"2014/12/31"
2014.12.31
```

Creating functions from strings uses `value` (the q interpreter) or `parse`:

```q
q)value "{x*x}"
{x*x}
q)parse "{x*x}"
(enlist(:{x*x}))
```

---

## 7.4 Creating Typed Empty Lists

The general empty list has type `0h`:

```q
q)type ()
0h
```

Appending an atom to a general empty list creates a simple list of that atom's type:

```q
q)L:()
q)type L
0h
q)L,:42
q)type L
7h
```

This can cause problems in tables. If the first appended value is the wrong type, the column type is fixed incorrectly:

```q
q)c1:()
q)c1,:42
q)c1,:98.6
'type
```

Cast the empty list to create a typed empty list:

```q
q)c1:`float$()
q)c1,:42
'type
q)c1,:98.6
q)c1
98.6
```

Operations on typed empty lists preserve the type:

```q
q)0#10 20 30
0#0
```

An idiom for creating typed empty lists:

```q
q)0#0
0#0
q)0#0.0
`float$()
q)0#`
`symbol$()
```

Note: There is no method to type nested empty lists.

---

## 7.5 Enumerations

The `$` operator extends to user-defined target domains, providing enumerated types. The binary form uses a variable name holding unique values as the left operand.

### 7.5.1 Traditional Enumerations

Traditional enumerations associate descriptive names with integral values. They serve multiple purposes:

- Enable descriptive names instead of arbitrary numbers
- Enable type checking for permissible values
- Provide namespacing for reused names across domains

A subtler, more powerful use is data normalization.

### 7.5.2 Data Normalization

Data normalization eliminates duplication while retaining minimum required data. For example, a list of text entries drawn from a fixed set of values (such as stock ticker symbols) presents problems:

- Variable length values complicate storage and retrieval
- Repetition duplicates data and creates synchronization issues

An enumeration solves both. Given a list `v` of symbols from unique list `u`:

```q
q)u:`g`aapl`msft`ibm
q)v:1000000?u
q)v
`g`g`msft`aapl`msft`aapl`msft`ibm`msft`aapl`g`ibm`aapl`msft`msft`aapl...
```

Or determine `u` from `v`:

```q
q)v
`jha`jha`fna`fed`fna`fna`jha`jha`jgc`pkh`pkh`pkh`fna`fed`jha`cpi...
q)u:distinct v
q)u
`jha`fna`fed`jgc`pkh`cpi`igb`hln`mjh`ooj
```

Create an index list `k` showing each item of `v`'s position in `u`:

```q
q)u:`g`aapl`msft`ibm
q)v:1000000?u
q)k:u?v
q)k
2 1 1 3 3 1 0 0 0 3 0 2 2 1 2 3 1 0 1 1 2 1 2 0 2 1 1 0 1 1 3 0...
```

The key observation: `u` and `k` carry precisely the same information as `v` and can reconstitute it:

```q
q)u[k]
`msft`aapl`aapl`ibm`ibm`aapl`g`g`g`ibm`g`msft`msft`aapl`msft`ibm...
q)v~u[k]
1b
```

`u` and `k` constitute a traditional enumeration where `u` is the name list and indices are associated values.

The trade benefits: speed and compactness. Variable-length text searching is slow compared to uniform integer traversal. Moreover, `u` and `k` normalize `v`'s data. While `v` has many repetitions, `u` stores each symbol once. Reading/writing index list `k` from disk is very fast.

Storage requirement for symbol list: with count `a`, maximum text width `b`, and variable count `x`:

Denormalized storage: `b*x`

Indexed storage: `a*b + 4*x`

When `a` is small and `b` is moderately large, factorization significantly reduces storage.

Example comparison:

```q
v:`ccccccc`bbbbbbb`aaaaaaa`ccccccc`ccccccc`bbbbbbb
u:distinct v
u
`ccccccc`bbbbbbb`aaaaaaa
k:u?v
k
0 1 2 0 0 1
```

### 7.5.3 Enumerating Symbols

The process of converting a symbol list to equivalent indices is called _enumeration_ in q. It uses another `$` overload with the variable name holding unique symbols as left operand and a list drawn from that domain on the right.

Under the covers, `$` performs the indexing operation, replacing each symbol with its index. Q hides this and displays enumerated symbols in reconstituted form with domain name annotation:

```q
q)`u$v
`u$`msft`aapl`aapl`ibm`ibm`aapl`g`g`g`ibm`g`msft`msft`aapl`msft...
```

Recover underlying integer values by casting to integer:

```q
q)ev:`u$v
q)`int$ev
2 1 1 3 3 1 0 0 0 3 0 2 2 1 2 3 1 0 1 1 2 1 2 0 2 1 1 0 1 1 3...
```

The basic enumerated symbol form is `` `u$v `` where `u` is a simple list of unique symbols and `v` is an atom from `u` or a (possibly nested) list of such. We call `u` the _domain_ of the enumeration and `` `u$ `` the enumeration over `u`. Under the covers, applying `` `u$ `` to vector `v` produces index list `k`.

All potential values must be in `u`; otherwise a `'cast` error results:

```q
q)u:`a`b`c
q)`u$`d
'cast
```

By convention in kdb+ tables, all symbol columns are enumerated over a common domain _sym_.

Note: Although integers are 64-bit in q3+, enumerations are 32-bit.

### 7.5.4 Using Enumerated Symbols

Continuing with the standard sym domain:

```q
q)sym:`g`aapl`msft`ibm
q)v:1000000?sym
q)ev:`sym$v
```

Enumerated `ev` substitutes for original `v` in nearly all situations:

```q
q)v[3]
`aapl
q)ev[3]
`sym$`aapl
q)v[3]:`ibm
q)ev[3]:`ibm
q)v=`ibm
000100010010011101000010010100000000100100000001000000001100001001011...
q)ev=`ibm
000100010010011101000010010100000000100100000001000000001100001001011...
q)where v=`aapl
4 5 19 20 21 31 33 34 41 42 43 49 58 59 61 74 81 83 90 94 95 98 114...
q)where ev=`aapl
4 5 19 20 21 31 33 34 41 42 43 49 58 59 61 74 81 83 90 94 95 98 114...
q)v?`aapl
4
q)ev?`aapl
4
q)v in `ibm`aapl
000111010010011101011110010100010110100101110001010000001111011001011...
q)ev in `ibm`aapl
000111010010011101011110010100010110100101110001010000001111011001011...
```

While enumerated version is item-wise equal, the entities are not identical:

```q
q)all v=ev
1b
q)v~ev
0b
```

Type matters with `~`.

### 7.5.5 Type of Enumerations

Each enumeration receives a new numeric data type beginning with `20h`. Starting with q version 3.2, type `20h` is reserved for the conventional enumeration domain sym (whether used or not). Other enumeration types begin with `21h` and proceed sequentially. The negative type for atoms and positive type for lists convention still applies. In a fresh q session:

```q
q)sym1:`g`aapl`msft`ibm
q)type `sym1$1000000?sym1
21h
q)sym2:`a`b`c
q)type `sym2$`c
-22h
```

The above was true in kdb+ V3.2. In later versions the type remains `20h`.

The sym domain has type `20h` even if created after another enumeration:

```q
q)sym:`b`c`a
q)type `sym$100?sym
20h
```

Enumerations with different domains are distinct even with identical constituents:

```q
q)sym1:`c`b`a
q)sym2:`c`b`a
q)ev1:`sym1$`a`b`a`c`a
q)ev2:`sym2$`a`b`a`c`a
q)ev1=ev2
11111b
q)ev1~ev2
0b
```

### 7.5.6 Updating an Enumerated List

Normalization reduces updating all occurrences of a value to a single operation, with significant performance implications for large repetitive lists:

```q
q)sym:`g`aapl`msft`ibm
q)ev:`sym$`g`g`msft`ibm`aapl`aapl`msft`ibm`msft`g`ibm`g...
q)sym[0]:`twit
q)sym
`twit`aapl`msft`ibm
q)ev
`sym$`twit`twit`msft`ibm`aapl`aapl`msft`ibm`msft`twit`ibm`twit...
```

In contrast, updating denormalized data requires changing **every** occurrence:

```q
q)v
`g`g`msft`ibm`aapl`aapl`msft`ibm`msft`g`ibm`g…
q)@[v; where v=`g; :; `twit]
`twit`twit`msft`ibm`aapl`aapl`msft`ibm`msft`twit`ibm`twit...
```

**Caution**: Manually modifying the sym list is extremely risky. Corrupting sym scrambles the entire database. Make a persistent backup before modifying, else update your resume.

### 7.5.7 Dynamically Appending to an Enumeration Domain

When the full enumeration domain extent is unknown in advance, appending new values is complicated. Appending to ordinary symbol lists is simple:

```q
q)sym:`g`aapl`msft`ibm
q)v:1000000?sym
q)ev:`sym$v
q)v,:`twtr
q)ev,:`twtr
'cast
```

New values **must** first be added to the unique list:

```q
q)sym,:`twtr
q)ev,:`twtr
```

For dynamically generated values, test if the value is in the enumeration domain and append to it if absent. Q anticipates this using another `?` overload. The syntax matches enumeration `$`—variable name as left operand and source symbol(s) as right. This application of `?` checks if source symbols are in the domain, appends any missing ones, and returns the enumerated version:

Build from scratch:

```q
q)sym:()
q)`sym$`g
'cast
q)`sym?`g
`sym$`g
q)sym
,`g
q)`sym?`ibm`aapl
`sym$`ibm`aapl
q)sym
`g`ibm`aapl
q)`sym?`g`msft
`sym$`g`msft
q)sym
`g`ibm`aapl`msft
```

The previous example now works with `?` replacing `$`:

```q
q)ev,:`sym?`twtr
```

### 7.5.8 Resolving an Enumeration

Enumerated symbols substitute for equivalent symbol values in most expressions. However, some situations require non-enumerated values, such as converting between enumeration domains or merging databases.

Recover un-enumerated values using the built-in `value`:

```q
q)sym:`g`aapl`msft`ibm
q)v:1000000?sym
q)ev:`sym$v
q)value ev
`aapl`g`msft`msft`ibm`msft`msft`msft`msft`msft`g`ibm`ibm`ibm...
q)v~value ev
1b
```

This is another `value` overload, the function implementing the q interpreter.

---

## Navigation

**Previous**: [6. Functions](../6_Functions/)  
**Next**: [8. Tables](../8_Tables/)

© 2021 Kx Systems, Inc. KX and kdb+ are registered trademarks of Kx Systems, Inc., a subsidiary of FD Technologies plc.
