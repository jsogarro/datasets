# 4 Operators

# 4. Operators¶

## 4.0 Operators and Keywords Are Functions¶

### 4.0.1 Function Notation¶

Operators are built-in functions which can be used with in-fix notation. We examine functions in depth in Chapter 5, but cover some salient points here. There are two main differences between the functions we can write and built-in functions.

- Our functions must have alphanumeric names whereas q functions can have purely symbolic names.
- Our functions can only be used in prefix notation whereas q functions can be used prefix or infix.

Function application in q uses square brackets to enclose the arguments, and semicolons to separate multiple arguments. Thus the output value of a unary function f for the input x is written f[x] We can omit the brackets for unary application and write f x. Application of a binary function g on arguments x and y is written g[x;y] in prefix or x g y in infix.

`f``x``f[x]``f x``g``x``y``g[x;y]``x g y`An atomic function acts recursively on data structures. For example, applying it to a list is the same as applying it to each item in the list.

### 4.0.2 Primitives, Verbs and Functional Notation¶

The normal way of writing addition in mathematics and most programming 
languages uses an operator with infix notation â e.g., addition is written with a plus symbol between the two operands.

```q
2+3
```

In q, we can write addition this way and read it right-to-left as âadd 3 to 2.â.

```q
q)2+3
_
```

The primitive operators are built-in functions, including the basic arithmetic, relation and comparison operators. Some are represented by a single ASCII symbol such as +, -, =, and <. Others use compound symbols, such as <=, >=, and <>. Keywords have names such as not or neg.

`+``-``=``<``<=``>=``<>``not``neg`Operators and keywords can also be used with ordinary function notation. For example, we can also use + as a binary function that takes two numeric arguments and returns a numeric result. You probably wouldn't think twice at seeing sum[a;b] but you might blink at the following perfectly logical equivalent.

`+``sum[a;b]````q
q)+[2;3]
_
```

Qbies will definitely need to get accustomed to,

```q
q)=[2;3]
_
```

It is even possible to apply an operator using a combination of infix and functional notation. This may look strange, even to initiates.

```q
q)(2+)[3]
5
q)(2+)3
_
```

### 4.0.3 Extension of Atomic Functions¶

A fundamental feature of atomic functions is that their action extends automatically to the items in a list. Of course, if you want to combine two lists they must be of the same length.

```q
q)neg 1 2 3
-1 -2 -3
q)1 2 3+10 20 30
11 22 33
q)1 2 3+10 20 30 40
'length
```

This applies to nested lists as well, provided they conform in shape for multivalent functions.

```q
q)neg (1 2 3; 4 5)
-1 -2 -3
-4 -5
q)(1 2 3; 4 5)+(100 200 300; 400 500)
101 202 303
404 505
```

Another fundamental property of atomic operators is that they implicitly extend atom arguments to match lists.

```q
q)100+1 2 3
101 102 103
q)1 2 3+100
_
```

Atom extension also applies with nested lists.

```q
q)100+(1 2 3; 4 5)
101 102 103
104 105
q)(1 2 3; 4 5)+100
_
```

## 4.1 Operator Precedence¶

There is none.

### 4.1.1 Traditional Operator Precedence¶

Mathematical operators and most programming languages have a concept of operator precedence, which attempts to resolve ambiguities in the evaluation of arithmetic and logical operations in expressions. The arithmetic precedence rules were drummed into you in elementary school: multiplication and division are equal and come before addition and subtraction, etc. There are similar precedence rules for =, <, >, âandâ and âorâ.

### 4.1.2 Left-of-Right Evaluation¶

Although the traditional notion of operator precedence has the weight of incumbency (not to mention the imprecations of your fifth grade math teacher), itâs time to throw the bum out. As mentioned in Chapter 1, q has no rules for operator precedence. Instead, it has one simple rule for evaluating any expression:

Expressions are evaluated left-of-right

which equates to

Expressions are evaluated right-to-left

Please review the Mathematics Refresher if the notion of left-of-right is new to you. The short version is that the composite f(g(x)), read âf of g of xâ is evaluated by first substituting x into g and then substituting that result into f. Evaluating the inner function first actually becomes right-to-left. Thinking functionally makes âofâ a paradigm not just a preposition.

The adoption of left-of-right evaluation frees q to treat expression evaluation simply and uniformly without redundant parentheses getting in the way. Infix or prefix notation can be used as suits the occasion. Left-of-right expression evaluation also means that there is no ambiguity in any expression â from the compilerâs perspective if not yours. Parentheses can always be used to group terms and override the default evaluation order but there will be far fewer once you abandon old (bad) habits.

Arrange your expressions with the goal of placing parentheses and brackets on the endangered species list. Most are just programming noise unless you insist on writing Lisp.

### 4.1.3 The Gotcha of Left-of-Right Evaluation¶

Due to left-of-right evaluation, parentheses are needed to isolate the result of an expression that is the left operand of an infix operator. Letâs take a closer look at this.

In any language you might have seen, the following expression evaluates to 10 but not in q. You can parenthesize or rearrange to get 10.

```q
q)2*3+4
14
q)(2*3)+4
10
q)4+2*3
10
```

In some cases parentheses are simply unavoidable.

```q
q)(2+3)*3+4
35
```

This is the one (and only) situation in which parentheses are necessary in q.

If the left operand of an operator is an expression it must be parenthesized, otherwise the operator will bind to the right-most element of the expression.

Redundant parentheses

These parentheses on the left are often not needed in traditional programming and their omission in an overzealous extermination campaign is a common error for qbies. Please donât overreact by putting parentheses around all operands âto be safe.â Spend five seconds and think about it. Eventually it will be second nature.

### 4.1.4 Rationale for No Operator Precedence¶

Operator precedence as normally encountered in programming languages is feeble. It requires all the components of an expression to be analyzed before anything can be evaluated. Moreover, it often results in the use of parentheses to override the very rules that are purportedly there to help.

Even more damning is that operator precedence forces complexity. Some programming languages (not q) allow user-written binary functions to be operators. This would entail the extension of precedence levels to cover user functions, even those as yet unborn. If youâve worked in such a language, you eventually run out of precedence levels (and patience) and end up needing parentheses with operators of the same precedence!

## 4.2 Match ~¶

`~`The non-atomic binary Match operator ~ applies to any two q entities, returning the boolean result 1b if they are identical and 0b otherwise. For two entities to match, they must have the same shape, the same type and the same value(s), but they may occupy separate storage locations. Colloquially, clones are considered identical in q.

`~``1b``0b`This differs from the notion of identity in many traditional languages having pointers or objects. For example, in OO languages of C ancestry, objects are equal if and only if their underlying pointers address the same memory location. Identical twins are not equal. You must write your own method to determine if one object is a deep copy of another.

There are no restrictions as to the type or shape of the two operands for Match. Try to predict each of the following results of Match as you enter them into your console session.

```q
q)42~40+2
_
q)42~42h
_
q)42f~42.0
_
q)42~`42
_
q)`42~"42"
_
q)4 2~2 4
_
q)42~(4 2;(1 0))
_
q)(4 2)~(4;2*1)
_
q)(())~enlist ()
_
q)(1; 2 3 4)~(1; (2; 3; 4))
_
q)(1 2;3 4)~(1;2 3 4)
_
```

Use Match to test

While learning or debugging q (except for the q gods who write perfect q code every time), applying Match can be an effective way to determine if you have what you intended. For example, qbies often trip over the following, thinking the latter is a singleton list.

q)42~(42)
1b

```q
q)42~(42)
1b
```

## 4.3 Equality and Relational Operators¶

It comes as a surprise to many who are new to vector programming that relational operators are atomic functions that return boolean values. Relational operations do not require the types of their operands to match, but they must be compatible.

### 4.3.1 Equality = and Disequality <>¶

`=`The equality operator = differs from Match ~ in that it is atomic in both operands, meaning it tests its operands atom-wise instead of in entirety. All atoms of numeric, temporal or char type are mutually compatible for equality, but symbols are compatible only with symbols.

`=``~`Equality tests whether two compatible atoms represent the same value, without regard to type.

```q
q)42=42i
_
q)42=42.0
_
q)42=0x42
_
q)42="*"
_
```

That last one may come as a surprise. It simply reflects that the underlying bit pattern of the ASCII char * is the same as the underlying bit pattern for the integer 42.

`*`For temporal types the comparison is between the points on the calendar/clock rather than the underlying counts.

```q
q)2000.01.01=2000.01.01D00:00:00.000000000
1b
q)2015.01.01<2015.02m
1b
q)12:00:00=12:00:00.000
1b
```

A symbol and a character are not compatible and an error results from the test,

```q
q)`a="a"
'type
```

The not-equal primitive is <>.

`<>````q
q)42<>0x42
_
```

The test ânot equalâ can also be achieved by applying not to the result of testing with =. This what <> actually does.

`not``=``<>`q)not 42=98.6
_

```q
q)not 42=98.6
_
```

When comparing floats, q uses multiplicative tolerance for non-zero values, which makes floating point arithmetic give reasonable results. At the time of this writing (Sep 2015) the tolerance is \(10^{-14}\).

```q
q)r:1%3
q)r
0.3333333
q)2=r+r+r+r+r+r
1b
```

### 4.3.2 Not Zero not¶

`not`The unary, atomic keyword not differs from its equivalent in some traditional languages. It returns a boolean result and has domain of all numeric, temporal and character types; it is not defined for symbols. The not keyword generalizes the reversal of true and false bits to any entity having an underlying numeric value. It answers the Hamletonian question: to be, or not to be, zero.

`not``not`The test against zero yields the expected results for boolean arguments.

```q
q)not 0b
_
q)not 1b
_
```

The test against zero applies for any type with underlying numeric value.

```q
q)not 0b
_
q)not 1b
_
q)not 42
_
q)not 0
_
q)not 0xff
_
q)not 98.6
_
```

For char values, not returns 0b except for the character representing the underlying value of 0.

`not``0b````q
q)not "*"
_
q)not " "
_
q)not "\000"
_
```

For temporal values, an underlying 0 corresponds to the stroke of midnight at the millennium for types including a date and simply midnight for time-only types.

```q
q)not 2000.01.01
_
q)not 2014.01.01
_
q)not 2000.01.01T00:00:00.000000000
_
q)not 2000.01m
_
q)not 00:00:00
_
q)not 12:00:00.000000000
_
```

### 4.3.3 Order: <, <=, >, >=¶

Less Than <, Greater Than > Up To <= and At Least >= are atomic and are defined for all compatible atom types. Numeric and char types are mutually compatible, but symbols are only compatible with symbols. As with equality, comparison for numeric and char types is based on underlying numeric value, independent of type.

`<``>``<=``>=````q
q)4<42
_
q)4h>0x2a
_
q)-1.4142<99i
_
```

As with equality, the comparison for temporal types is between the points on the calendar/clock rather than the underlying counts.

```q
q)2000.01.01<2000.01.01D00:00:00.000000001
1b
q)2015.01.01<2015.02m
_
q)12:00:01>12:00:00.000
_
```

For char atoms, comparing the underlying numeric value follows the ASCII collation sequence.

```q
q)"A"<"Z"
_
q)"a"<"Z"
_
q)"A"<"O"
_
q)"?"<"?"
_
```

To see the entire ASCII collation sequence in compact form

q)16 16#"c"$til 256
_

```q
q)16 16#"c"$til 256
_
```

Symbol comparison is based on lexicographic order.

```q
q)`a<`b
_
q)`abc<`aba
_
```

Now that we are familiar with relational operations on atoms, letâs examine their item-wise extensions to simple lists. Notice the simple boolean list returned.

```q
q)2 1 3=1 2 3
_
q)10 20 30<=30 20 10
_
q)2=1 2 3
_
q)"zaphod"="Arthur"
_
q)`a`b`a`d=`a`d`a`b
_
```

## 4.4 Basic Arithmetic: +, -, *, %¶

`+``-``*``%`The arithmetic operators are atomic and come in binary and unary flavors. We begin with the four operations of elementary arithmetic. Arithmetic operations are defined for all numeric and temporal types, and all numeric types are compatible.

`+``42+67``-``42.0-5.3456``*``2h*3h``%``42%6`Arithmetic looks pretty much like other programming languages, except that division is represented by % since / is used to delimit comments. Simon Garland of KX points out that this is actually closer to Ã· (division) the way God meant it to be written.

`%``/``Ã·````q
q)2+3
_
q)a:6
q)b:7
q)b-a
_
q)a*b
_
q)4%2
2f
```

The result of division is always a float.

The major learning adjustment in q arithmetic expressions is due to left-of-right evaluation and the absence of precedence.

```q
q)6*3+4
42
```

Type promotion for arithmetic operators follows two rules:

- Binary types are promoted to int
- The result type of an operation is the narrowest type that will accommodate both operands.

Here are examples of binary data promotion. Note that arithmetic on booleans is not performed modulo 2.

```q
q)1b+1b
2i
q)42*1b
42
q)5i*0x2a
210i
```

Overflow and underflow are not trapped on arithmetic operations on integer types.

q)9223372036854775806+4
-9223372036854775806
q)2*5223372036854775800
-8000000000000000016
q)-9223372036854775806-4
9223372036854775806

```q
q)9223372036854775806+4
-9223372036854775806
q)2*5223372036854775800
-8000000000000000016
q)-9223372036854775806-4
9223372036854775806
```

When a floating-point type occurs in an expression, the result is a float.

```q
q)6+7.0
13f
q)1.0+1b
2f
q)6.0*7.0e
42f
```

The symbols for arithmetic operators are  binary.

In particular, while - is used as a lexical marker to denote a negative number, there is no unary function - to negate a numeric value. Its attempted use for such generates an error. Use the operator neg instead

q)42
42
q)-42
-42
q)a:42
q)-a / error
'-
q)neg a
-42

`-``-``neg````q
q)42
42
q)-42
-42
q)a:42
q)-a / error
'-
q)neg a
-42
```

Being atomic, arithmetic operators and their type promotion are performed atom-wise on lists.

```q
q)1.0+10 20 30
_
q)10 20 30%1 2 3
_
q)100 200 300+1b
_
q)1+(100 200;1000 2000)
_
```

## 4.5 Greater | and Lesser &¶

`|`These atomic binary operators follow the same type promotion and compatibility rules as arithmetic operators. They are defined for all values with underlying numeric values but are not defined for symbols and GUIDs.

The Greater operator | returns the larger of its operands; this reduces to logical âorâ for binary operands. The Lesser operator & returns the smaller of its operands, which reduces to logical âandâ for binary operands.

`|``&````q
q)42|43
_
q)98.6&101.9
9_
q)0b|1b
_
q)1b&0b
_
q)42|0x2b
_
q)"a"|"z"
"_
q)`a|`z / error
_
```

Being atomic they operate item-wise on lists.

```q
q)2|0 1 2 3 4
_
q)11010101b&01100101b
_
q)"zaphod"|"arthur"
_
```

For readability of logical operations on binary data, | can also be written as or and & can be written as and.

`|``or``&``and````q
q)1b or 0b
_
q)1b and 0b
_
q)42 or 43
_
```

## 4.6 Amend :¶

`:`An overload of : that is âassign in place.â

`:`### 4.6.1 Amend in C Language¶

We are familiar with the basic form of assignment.

```q
q)a:42
```

Programmers from languages with C heritage are familiar with expressions such as,

```q
x += 2; // C expression that assigns in place
```

This has the same effect as the following but can be more efficiently implemented at the machine instruction level.

```q
x = x + 2; // C expression
```

Reading the first statement succinctly as add 2 to x in place motivates the interpretation of the operation as âamendâ. To wit, the value assigned to x is amended by applying the operation + with the supplied operand 2.

`x``+`### 4.6.2 Simple q Amend¶

Transliterating the above C expression to q yields the +: operation to amend a variable in place.

`+:````q
q)x:42
q)x+:1
q)x
_
```

There is nothing special about + here. Amend can be used with any symbolic operator having compatible signature.

`+````q
q)a:43
q)a-:1
q)a
_
q)a&:21
q)a
_
```

In spite of the linguistic dissonance, a q variable can be amended even if it has not been previously assigned.

In a fresh q session:

q)x
'x
q)x+:42
q)x
_

```q
q)x
'x
q)x+:42
q)x
_
```

### 4.6.3 Amend with Lists¶

The capability to modify in place extends to lists and indexing.

```q
q)L:100 200 300 400
q)L[1]+:99
q)L
_
q)L[1 3]-:1
q)L
_
q)L1:(1 2 3; 10 20 30)
q)L1[;2]+:100
q)L1
_
```

A very useful idiom is ,: which appends to a list in place.

`,:````q
q)L:1 2 3
q)L,:4
q)L
_
q)L,:100 200
q)L
_
```

Amend does type promotion based on the operator it is combined with, except for ,: which requires exact type match.

`,:`q)L:1.1 2 2 3.3
q)L[1]+:100
q)L,:100
'type

```q
q)L:1.1 2 2 3.3
q)L[1]+:100
q)L,:100
'type
```

## 4.7 Exponential Primitives: sqrt, exp, log, xexp, xlog¶

`sqrt``exp``log``xexp``xlog`### 4.7.1 sqrt¶

`sqrt`The atomic unary sqrt has as domain all numeric values and returns a float representing the square root of its input. It returns null when the square root is not defined.

`sqrt````q
q)sqrt 2
_
q)sqrt 42.4
_
q)sqrt 1b
_
q)sqrt -2
_
```

### 4.7.2 exp¶

`exp`The atomic unary exp has as domain all numeric values and returns a float representing the base e raised to the power of its input.

`exp````q
q)exp 1
_
q)exp 4.2
_
q)exp -12i
_
```

Do not confuse the e used in the display of base-10 scientific notation with the mathematical base of exponentials and natural logarithms.

`e`q)1e10 / this is ten billion
1e+10

```q
q)1e10 / this is ten billion
1e+10
```

### 4.7.3 log¶

`log`The atomic unary log has as domain all numeric values and returns a float representing the natural logarithm of its input. It returns null when the logarithm is not defined.

`log````q
q)log 1
_
q)log 42.0
_
q)log .0001
_
q)log -1
_
```

### 4.7.4 xexp¶

`xexp`The atomic binary xexp has as domain all numeric values in both operands and returns a float representing the left operand raised to the power of the right operand. When the mathematical operation is undefined, the result is null.

`xexp````q
q)2 xexp 5
_
q)-2 xexp .5
_
```

A q naming convention

We point out here, since it is our first encounter, a q naming convention. A unary function â e.g., exp â sometimes has a binary version â xexp â with an x prepended to its name.

`exp``xexp``x`In classic Arthurian fashion, the rationalization is that the additional parameter is âxâ.

### 4.7.5 xlog¶

`xlog`The atomic binary xlog has as domain all numeric values in both operands and returns a float representing the logarithm of the right operand with respect to the base of the left operand. When the mathematical operation is undefined the result is null.

`xlog````q
q)2 xlog 32
-
```

```q
q)2 xlog -1
-
```

## 4.8 More Numeric Primitives¶

### 4.8.1 Integer Division div and Modulus mod¶

`div``mod`The atomic binary div is atomic in both operands, which are numeric values. The result is the integer quotient of the left operand (dividend) by the (positive) right operand (divisor), which is equal to the result of normal division rounded down to the next lower integer. The operation returns null for non-positive divisor.

`div````q
q)7 div 2
3
q)7 div 2.5
2
q)-7 div 2
-4
q)-7 div 2.5
_
q)7 div -2       / returns -4 in later versions
0N
q)3 4 5 div 2
_
q)7 div 2 3 4
_
q)3 4 5 div 2 3 4
_
```

The binary mod is atomic in both operands, which are numeric values. The result is the remainder of the integer quotient of the left operand (dividend) by the positive right operand (divisor). It is equal to

`mod`dividend â (dividend div divisor)

The result is null for non-positive divisor.

```q
q)7 mod 2
1
q)7 mod 2.5
_
q)-7 mod 2
1
q)-7 mod 2.5
_
q)7 mod -2
_
q)3 4 5 mod 2
_
q)7 mod 2 3 4
_
q)3 4 5 mod 2 3 4
_
```

Many languages use % for modulus, but it is division in q. This is a common mistake of qbies.

`%`### 4.8.2 Sign signum¶

`signum`The atomic unary signum has domain all numeric and temporal types and returns an int representing the sign of its input, where 1i represents positive, -1i represents negative and 0i represents a zero.

`signum``1i``-1i``0i````q
q)signum 42
1i
q)signum -42.0
_
q)signum 1b
_
q)signum 0
_
```

Temporal types are treated as their underlying offsets.

```q
q)signum 1999.12.31
_
q)signum 12:00:00.000000000
_
```

### 4.8.3 reciprocal¶

`reciprocal`The atomic unaryËreciprocal has as domain all numeric types and returns the float result of 1.0 divided by the input. It returns the appropriately signed infinity for the reciprocal of 0.

`reciprocal````q
q)reciprocal 0.02380952
42.00001
q)reciprocal 0.0
0w
q)reciprocal -0.0
-0w
```

### 4.8.4 floor and ceiling¶

`floor``ceiling`The atomic unary floor has as domain integer and floating-point types and returns a long representing the largest integer that is less than or equal to its argument.

`floor````q
q)floor 4.2
4
q)floor 4
_
q) floor -4.2
_
```

The floor operator can be used to truncate or round floating-point values to a specific number of digits to the right of the decimal.

`floor`q)x:4.242
q)0.01*floor 100*x
4.24

```q
q)x:4.242
q)0.01*floor 100*x
4.24
```

Analogous to floor, the atomic unary ceiling has as domain numeric types and returns the smallest long that is greater than or equal to its argument.

`floor``ceiling````q
q)ceiling 4.2
5
q)ceiling 4
_
q)ceiling -4.2
_
```

For reasons known only to the q gods, floor and ceiling do not apply to short types.

`floor``ceiling`q)floor 4h
'type

```q
q)floor 4h
'type
```

(Later versions of kdb+ include shorts in their domains. [Ed.] )

### 4.8.5 Absolute Value abs¶

`abs`The atomic unary abs has domain all integer and floating-point types. The result is the input when it is greater than or equal to zero and its negation otherwise. The result of abs has the same type as the argument except for binary types, which are type promoted to int.

`abs``abs````q
q)abs 42
_
q)abs -42
_
q)abs 1b
_
```

## 4.9 Operations on Temporal Values¶

Because all q time values are integral offsets from canonical base points, properties and operations on temporal types are simple. For example, one value of a temporal type comes before another value of the same type just when the same is true of their underlying integer values. When dealing with temporal values of different types, q implicitly promotes to the wider type and then proceeds as just described.

Moreover, basic temporal arithmetic is integer arithmetic. The difference of two (absolute) temporal values of the same type is the span given by the difference of their underlying integer counts. Conversely, given a temporal value, adding an integer to it (i.e., to its underlying integer count) yields a temporal value of the same type.

Casting reveals the underlying integer count of any temporal value.

```q
q)`int$1999.12.31
-1i
q)`int$2013.01m
_
q)`int$12:00:00.123
_
q)`long$12:00:00.123456789
_
```

There is no concept of time zone in q temporal values. Those of us who wrestled with Javaâs original time implementation are thankful.

### 4.9.1 Temporal Comparison¶

Comparison within a temporal type amounts to simple comparison of the underlying integral offsets. Comparison across temporal types recognizes that the underlying values express different units and realizes them in common units.

For example, midnight on the second day of the millennium should be equal to the second day, but a naÃ¯ve comparison of the underlying counts will not tell us.

```q
q)2000.01.02=2000.01.02D00:00:00.000000000
1b
q)`int$2000.01.02
1i
q)`long$2000.01.02D02:00:00.000000000
93600000000000
```

Values of different types should be compared in the same units, which effectively amounts to converting to the most granular units. The Cast operator has the logic for such conversions built-in.

```q
q)`timestamp$2001.01.02
2001.01.02D00:00:00.000000000
```

To compare temporal values of different types, q converts to the most granular type and then does a straight comparison of the underlying values.

```q
q)2000.01.01<2000.01.01D12:00:00.000000000
_
```

### 4.9.2 Temporal Arithmetic¶

In contrast to many traditional languages, expressions involving temporal types and numerical types that should make sense actually work as expected. For example, temporal values are their underlying offsets for equality and comparison testing against numeric values.

```q
q)2000.01.01=0
1b
q)12:00:00=12*60*60
_
q)1999.12.31<0
_
```

Adding an integral value to a temporal value works because it is just added to the underlying offset.

```q
q)2014.12.31+1
_
q)2015.01.01+til 31 / all days in January
_
q)12:00:00+1
```

Adding a temporal value to another causes it to be viewed as a span, as you would want.

```q
q)12:00:00+01:00:00
_
```

One important case is adding a timespan to a date to yield a timestamp. There is actually some calculation under the covers to make this work.

```q
q)2015.01.01+12:00:00.000000000
2015.01.01D12:00:00.000000000
```

The difference between two values of a temporal type that counts days is the int difference of their underlying day counts.

```q
q)2001.01.01-2000.01.01
366i
q)2015.06m-2015.01m
_
```

The difference between two values of a type with time is the difference of the underlying offsets, expressed as the same type â i.e., as a span.

```q
q)2015.01.01D00:00:00.000000000-2014.01.01D00:00:00.000000000
365D00:00:00.000000000
q)12:00:00-11:00:00
_
q)12:00-11:00
_
```

## 4.10 Operations on Infinities and Nulls¶

Here we summarize the various behaviors of nulls and infinities in one place.

The float infinities and nulls act in the mathematically correct fashion in numeric expressions and comparisons. Integer infinities act correctly in comparisons and act as their underlying (finite) values in other operations.

The bit patterns of the integral nulls and infinities are legitimate base-2 integral representations with the high-order bit being the sign.

`0Wh``0111111111111111b``-0Wh``1000000000000001b``0Wi``01111111111111111111111111111111b``-0Wi``10000000000000000000000000000001b``0W``0111111111111111111111111111111111111111111111111111111111111111b``-0W``1000000000000000000000000000000000000000000000000000000000000001b`The same type-promotion rules apply to a null as for a normal value of that type.

An infinity value equals or matches only itself. All nulls are equal (they represent missing data), but different type nulls do not match (type matters).

In contrast to some languages, such as C, separate instances of NaN are equal.

```q
q)(0%0)=0%0
_
```

The not operator returns 0b for all infinities and nulls since they all fail the test of equality with 0.

`not``0b````q
q)not 0W
_
q)not -0w
_
q)not 0N
_
```

The neg operator reverses the sign of infinities but does nothing to nulls since sign is meaningless for missing data.

`neg````q
q)neg 0W
-_
q)neg -0w
_
q)neg 0N
_
```

We saw previously that for any numeric type

null < negative infinity < normal value < positive infinity

Nulls of different type, while equal, are not otherwise comparable â i.e., any relational comparison results in 0b.

`0b`Infinities of different type are ordered by their width. For positive infinities

short < int < long < real < float

For negative infinities

-float < -real < -long < -int < -short

Some examples follow. Try to predict the result before pressing Return.

```q
q)42<0W
_
q)-0w<42.0
_
q)-0w<1901.01.01
_
q)-0w<0w
_
q)0W<0w
_
q)-0w<0W
_
q)-10000000<0N
_
q)0N<42i
_
q)0n<-0w
_
```

The null symbol is less than any other symbol

```q
q)`a<`
_
```

The behavior of | and & with infinities and nulls follows from the rules for equality, comparison and type promotion mentioned already.

`|``&````q
q)42|0W
_
q)-42&0N
_
q)0w|0n
_
q)-0w&0n
_
q)0n|0N
_
q)0Wi&0W
_
```

The last result obtains because int infinity is promoted to a long and its bit pattern corresponds to the maximal positive 32-bit integer.

## 4.11 Alias ::¶

`::`Because q is strict, expressions are normally evaluated as soon as encountered by the interpreter. In particular, assignment with an expression on the right requires the expression to be evaluated before the result is assigned.

An alias is a variable that is an expression â i.e., it is not the result of expression evaluation but the expression itself. Otherwise put, an alias provides a way to defer evaluation of an expression.

Evaluation of an alias is lazy, meaning that it occurs only when necessary. More precisely, evaluation is forced when the variable is referenced, at which point a determination is made whether the expression needs to be (re)evaluated.

- If it is the first reference or if any variable in its associated expression has changed since the last evaluation, evaluation proceeds with the current values of all the variables in the expression. The result of the most recent evaluation is then stored internally and also returned. The stored result is said to be memoized.
- If no variables in the expression have changed since the previous evaluation, the memoized value is returned.

The alias variable is said to depend on any variables in its expression.

### 4.11.1 Creating an Alias with Double Colon¶

Double colon :: used outside a function body defines the variable in the left operand as an alias of the expression in the right operand. When the alias is referenced, the underlying expression is (re)evaluated as described above. The following trivial example defines b as an alias for a, contrasted with c which is just assigned the value of a. Observe that the subsequently changed value of a is reflected in b but not in c.

`::``b``a``c``a``a``b``c````q
q)a:42
q)b::a
q)c:a
q)a:43
q)b
43
q)c
42
```

Here is a more interesting alias.

```q
q)w::(x*x)+y*y
q)x:3
q)y:4
q)w
25
q)y:5
q)w
34
```

The mysteriously named utility 0N! is the identity function fortified with the side effect of displaying its input on the console.

`0N!`It is a non-invasive way to inspect the inner workings of an in-flight evaluation.

```q
q)w::(0N!x*x)+y*y
q)x:3
q)y:4
q)w
9
25
q)w
25
q)y:6
q)w
9
45
```

Observe that the first time w is referenced, the expression is evaluated, as indicated by the display of the value of x*x. On the next reference, the expression is not re-evaluated since none of the variables it depends on have changed. After changing y, the last reference causes re-evaluation.

`w``x*x``y`### 4.11.2 Alias vs. Function¶

A function also represents deferred evaluation. In the previous example, we could define,

```q
q)fu:{(x*x)+y*y}
q)fu[3;4]
25
```

There are two key differences between an alias and the analogous function.

- To evaluate an expression wrapped in a function you explicitly provide the arguments and apply the function all in one step. With an alias you set the variables at any point in the program and the expression is evaluated when, and only when, the alias variable is referenced.
- The function does not memoize its result, so it recalculates on every application, even if the arguments do not change.

### 4.11.3 Dependencies¶

An alias variable depends on the entities in its associated expression. In our previous example w depends on x and y. A list of all dependencies is maintained in the system dictionary .z.b, which is also obtainable via the command \b.

`w``x``y``.z.b``\b````q
q)w::(x*x)+y*y
q).z.b
x| w
y| w
```

Each key in .z.b is associated to all the entities that depend on it.

`.z.b`It is permissible to create an alias with another alias in its expression. This results in a chain of dependencies. The entire chain is resolved lazily upon reference.

```q
q)u::w*w
q).z.b
x| w
y| w
w| u
q)x:3
q)y:4
q)u
_
```

Such a recursive definition leads to a hierarchy of dependencies, in which a variable depends not only on the variables in its own expression, but also any variables that its expression depends on, etc. You can easily build sophisticated dependency graphs this way.

You can easily build unmaintainable code this way.

A dependency chain that would create a loop is detected and results in an error. Continuing the example above,

```q
q)x::u
q)x
'loop
```

### 4.11.4 Views¶

Aliasing is commonly used to provide a database view by specifying a query as the expression.

```q
q)t:([]c1:`a`b`c`a;c2:20 15 10 20;c3:99.5 99.45 99.42 99.4)
q)v::select sym:c1,px:c3 from t where c1=`a
q)v
sym px
--------
a 99.5
a 99.4
q)update c3:42.0 from `t where c1=`a
_
q)v
_
```

The table dependencies of a view are reflected in .z.b.

`.z.b````q
q).z.b
_
```

