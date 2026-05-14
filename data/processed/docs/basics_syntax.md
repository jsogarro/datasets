# Syntax | Basics | kdb+ and q documentation

## Elements

The elements of q include:

- Functions: operators, keywords, lambdas, and extensions
- Data structures: atoms, lists, dictionaries, tables, expression lists, and parse trees
- Attributes of data structures
- Control words
- Scripts
- Environment variables

**Applicable values**: Lists, dictionaries, file and process handles, and all function types are applicable values—each represents a mapping.

## Tokens

A **token** is one or more characters forming a syntactic unit. ASCII symbols carry syntactic significance in q, denoting functions, nouns, iterators, names, constants, and punctuation that bound expressions.

Multi-character tokens include:
- `<=` (less-than-or-equal)
- `>=` (greater-than-or-equal)
- `<>` (not-equal)
- `::` (null, view, set global)
- `/:` (each-right)
- `\:` (each-left)
- `':` (each-prior, each-parallel)

## Nouns

All data are syntactically **nouns**, including:

- Atomic values: character, integer, floating-point, temporal values, symbols, functions, dictionaries, and `::` (null)
- Collections in lists
- Nested lists

### Numerical Constants

Integer and floating-point constants use standard notation with decimal and exponential forms. Negative constants use a minus sign immediately left of the positive value. Special atoms (`0W`, `0N`) denote infinities and null values.

### Temporal Constants

```q
2017.01              / month
2017.01.18           / date
00:00:00.000000000   / timespan
00:00                / minute
00:00:00             / second
00:00:00.000         / time
```

### Character Constants

An atomic character uses double quotes: `"a"`. Multiple characters or none between quotes denotes a character list.

### Symbol Constants

Symbol constants use a backtick prefix: `` `a.b_2 ``

## List Notation

Expressions separated by semicolons within parentheses denote a list:

```q
(3 + 4; a _ b; -20.45)
```

Empty list: `()`

At least one semicolon is required (except for empty lists). Parentheses around a single expression are grouping, not list notation:

```q
(a * b) + c
```

**One-item lists** require the `enlist` function:

```q
q)3           /atom
3
q)enlist 3    / 1-item list
,3
```

## Vector Notation

Typed vector constants use special syntax:

```q
01110001b                           / boolean
"abcdefg"                           / character
`ibm`aapl`msft                      / symbol
2018.05 2018.07 2019.01m            / month
2 3 4 5 6h                          / short integer
2 3 4 5 6i                          / integer
2 3 4 5 6                           / long integer
2 3 4 5 6j                          / long integer
2 3 4 5.6                           / float
2 3 4 5 6f                          / float
```

### Strings

Character vectors are called **strings**. Escape sequences within strings:

| Escape | Meaning |
|--------|---------|
| `\"` | double quote |
| `\NNN` | character with octal value NNN |
| `\\` | backslash |
| `\n` | new line |
| `\r` | carriage return |
| `\t` | horizontal tab |

## Table Notation

Tables are written as lists with an initial expression list indicating key columns:

```q
q)([]sym:`aapl`msft`goog;price:100 200 300)
sym  price
----------
aapl 100
msft 200
goog 300
```

Empty brackets indicate a simple table (no key). Column values must be lists of equal count or atoms:

```q
q)sym:`aapl`msft`goog
q)price:100 200 300
q)([] sym; price)
sym  price
----------
aapl 100
msft 200
goog 300
```

Atoms broadcast across rows:

```q
q)([] sym:`aapl`msft`goog; price: 300)
sym  price
----------
aapl 300
msft 300
goog 300
```

For single-row tables, enlist at least one value:

```q
q)([] sym:enlist`aapl; price:100)
sym  price
----------
aapl 100
```

Keyed tables use non-empty initial brackets:

```q
q)([names:`bob`carol`bob`alice;city:`NYC`CHI`SFO`SFO]; ages:42 39 51 44)
names city| ages
----------| ----
bob   NYC | 42
carol CHI | 39
bob   SFO | 51
alice SFO | 44
```

## Attributes

Attributes apply to lists of special form, often on dictionary domains or table columns, to reduce storage or speed retrieval.

## Bracket Notation

Expressions separated by semicolons within brackets denote indexes or function arguments:

```q
m[0;0]      / matrix element selection
f[a;b;c]    / function application with three arguments
```

Unlike list notation, bracket notation allows single expressions or none: `m[]` (all items), `f[]` (no-argument function).

Operators work with bracket notation:

```q
+[a;b]      / equivalent to a + b
```

### Indexing Tables

Tables index by row first, then column:

```q
q)t:([]name:`Tom`Dick`Harry;age:34 42 17)
q)t[1;`age]
42
```

Eliding indexes selects all values:

```q
q)t[;`age]
34 42 17

q)t[1;]
name| `Dick
age | 42
```

Trailing indexes can be elided:

```q
q)t[1]
name| `Dick
age | 42
```

Shorthand for column selection:

```q
q)t[`age]    / same as t[;`age]
34 42 17
q)t`age
34 42 17
```

## Conditional Evaluation and Control Statements

A `$` prefix before bracket notation denotes conditional evaluation:

```q
$[a;b;c]
```

Control words precede bracket notation:

```q
do[a;b;c]
if[a;b;c]
while[a;b;c]
```

Control words are not functions and do not return results.

## Function Notation

Expressions separated by semicolons within braces denote a function (lambda):

```q
{x + 2}
{[x;y] x * y}
```

The first expression may be a **signature** naming arguments: `[name1;name2;…;nameN]`

Functions may span multiple lines in scripts.

## Prefix, Infix, Postfix

Functions apply in multiple notations:

```q
f[x]         / bracket notation
f x          / prefix
x + y        / infix
f\           / postfix (iterators only)
```

Apply lists to indexes similarly:

```q
q)"abcdef" 1 0 3
"bad"
```

### Infix and Prefix Have Long Right Scope

The right argument of unary or binary infix functions extends rightward consuming all expressions (within parentheses):

```q
q)count first (2 3 4;5 6)
3
```

Here, `count` receives `first (2 3 4;5 6)`, which is `2 3 4`.

```q
q)2 3 * 4 5 - 6 7
-4 -6
```

Left argument of Multiply is `2 3`; right argument is `4 5-6 7` (evaluating to `-2 -2`).

### Postfix Yields Infix

Iterators applied postfix to applicable values derive functions with infix syntax, regardless of rank:

```q
count'     / unary but infix syntax
```

Derived functions often require parentheses for postfix application.

### Prefix and Vector Notation

Prefix notation applies unary functions: `til 3`, `{x - 2} 5 3`

Item selection uses prefix: `(1; "a"; 3.5; `xyz) 2` returns `3.5`

Vector notation binds tightly:

```q
{x - 2} 5 6     / function applied to vector 5 6, not two separate applications
```

### Parentheses Around Infix Functions

Parentheses capture infix functions as values, preventing infix parsing:

```q
q)+\[1 2 3 4 5]                 / unary
1 3 6 10 15
q)+\[1000;1 2 3 4 5]            / unary
1001 1003 1006 1010 1015
q)1000+\1 2 3 4 5               / binary, infix
1001 1003 1006 1010 1015

q)(+\)[1000;1 2 3 4 5]          / binary, captured parentheses
1001 1003 1006 1010 1015
q)(+\)1 2 3 4 5                 / unary, postfix
1 3 6 10 15
```

Captured functions pass as arguments:

```q
q)(*) scan 1 2 3 4 5
1 2 6 24 120
q)n:("the ";("quick ";"brown ";("fox ";"jumps ";"over ");"the ");("lazy ";"dog."))
q)(,/) over n
"the quick brown fox jumps over the lazy dog."
```

Functions without infix syntax don't require parentheses:

```q
q)raze over n
"the quick brown fox jumps over the lazy dog."
q){,/[x]}over n
"the quick brown fox jumps over the lazy dog."
```

## Compound Expressions

Function expressions, index expressions, argument expressions, and list expressions are **compound expressions**.

## Empty Expressions

An empty expression represents the special atomic value **null**:

```q
(a+b;;c-d;)     / second and fourth expressions are empty
```

## Colon

### Assign

Colon names values:

```q
x: 10
```

### Explicit Return

Within a lambda, colon-value terminates evaluation and returns that value:

```q
f: {
  if[type[x]<0; :x];     / if atom, return it
  ...
}
```

### Colons in Names

I/O and interprocess communication functions use digit-colon notation: `0:`, `1:`

Unary operator forms use colon suffix (deprecated in q): `#:`

## Colon Colon

Double colon (`::`) with a name and expression performs:

- **Within functions**: global assignment `{… ; x::3 ; …}`
- **Outside functions**: view definition

## Iterators

Iterators are higher-order operators. Arguments are applicable values (functions, handles, lists, dictionaries), and results are derived functions.

Symbols denoting iterators:

| Token | Semantics |
|-------|-----------|
| `'` | Case and Each |
| `':` | Each Prior, Each Parallel |
| `/:`, `\:` | Each Right and Each Left |
| `/`, `\` | Converge, Do, While, Reduce |

Derived functions combine an iterator with a value:

```q
(+/)1 2 3 4     / sum
10
16 +/ 1 2 3 4   / sum with starting value
26
```

Notation for derived functions without arguments denotes a constant function atom: `+/`

## Names and Namespaces

Names consist of alphabetic characters, digits, dot (`.`), and underscore (`_`). First character cannot be numeric or underscore.

**Underscores in names are strongly deprecated** due to confusion with Drop.

A kdb+ session has a default namespace and nested child namespaces (the **K-tree**). Namespaces begin with a dot: `.h`, `.j`, `.q`, `.Q`, `.z` (reserved for KX).

**Compound names** contain dots separating simple names. All simple names have meaning relative to the K-tree.

- **Absolute names**: begin with a dot (e.g., `.q.type`)
- **Relative names**: all others

Two consecutive dots are invalid.

## Iterator Composition

A derived function is **composed** by a string of iterators with no spaces between them or between the value and the leftmost iterator:

```q
+\/:\:
```

Parse from left to right: the leftmost iterator modifies the operator, creating a function; the next iterator modifies that function, and so on.

## Projecting the Left Argument of an Operator

If the left argument of an operator is present but the right is absent, the combination denotes a **projection**:

```q
(3 +) 4         / "3 plus" applied to 4
7
```

## Precedence and Order of Evaluation

All functions in expressions have equal precedence. Evaluation is **strictly right to left**, except within specific compound expressions.

```q
a * b + c       / evaluates as a * (b + c)
```

In compound expressions:

- **Index and argument expressions**: evaluated right to left
- **Function, conditional, and control expressions**: evaluated left to right

Example:

```q
q)x: 10
q)(x + 5; x: 20; x - 5)
25 20 5
```

Rightmost expression evaluates first using original `x`; middle assigns new value; leftmost uses updated value.

Function expressions evaluate top to bottom:

```q
q)f:{a : 10; : x + a; a : 20}
q)f[5]
15
```

The explicit return (`:`) stops execution before the final assignment.

## Multiline Expressions

Individual expressions span multiple lines in scripts. Break after semicolons and indent continuations with spaces:

```q
(a + b;
  ;
  c - d)
```

This is equivalent to `(a+b;;c-d)`.

When expressions evaluate left to right (function, conditional, control), multiline evaluation follows top to bottom.

## Spaces

Spaces between tokens are usually optional, with exceptions:

- **No spaces** between:
  - `'` and `:` (iterator `':`)
  - `\` and `:` (iterator `\:`)
  - `/` and `:` (iterator `/:`)
  - digit and `:` (functions like `0:`)
  - `:` and `:` (global assignment `::`)

- **No spaces** between iterator glyphs and the value or iterator to the left

- **No spaces** between operator and colon for assignment

- **`/` as comment start** must be preceded by blank/newline; otherwise interpreted as iterator start

- **Underscore and dot** denote operators and appear in names; space needed to disambiguate operator from name

- **Neighboring numeric constants** in vector notation require space: `3.5 -1` is a two-element list; `3.5-1` is subtraction

- **Minus sign** is part of a negative constant unless the token to the left is a name, constant, `)`, or `]` with no space:

```q
x-1            / x minus 1
x -1           / x applied to -1
3.5-1          / 3.5 minus 1
3.5 -1         / list: 3.5 and -1
x[1]-1         / x[1] minus 1
(a+b)- 1       / (a+b) minus 1
```

## Comments

Line and trailing comments use `/`:

```q
q)/Oh what a lovely day
q)2+2  /I know this one
4
```

(Not within strings or after system commands.)

Multiline comments use matching singleton `/` and `\`:

```q
/
    Oh what a beautiful morning
    Oh what a wonderful day
\
```

Singleton `\` exits the script:

```q
a:42
\
ignore this and what follows
```

## Special Constructs

Backslash, colon, and single-quote (`/ \ : '`) carry special meanings outside ordinary expressions, denoting system commands and debugging controls.
