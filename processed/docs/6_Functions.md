# 6\. Functions¶

## 6.0 Overview¶

In this chapter, we cover functions in depth. Before diving in, you may wish to review the [Mathematics Refresher](../0_Overview/#02-mathematics-refresher) in Chapter 0 for the approach and terminology we adopt.

We describe various built-in functions along the way in this tutorial. Sometimes we shall use a built-in function with minimal explanation; simply look it up in [Appendix A](../A_Built-in_Functions/), which contains specifics and examples of nearly all the q built-in functions.

## 6.1 Function Specification¶

The notion of a function in q corresponds to a (mathematical) map that is specified by an algorithm. A _function_ is a sequence of expressions to be evaluated, having optional input parameters and a return value. _Application_ of a function is the process of evaluating the expressions in sequence, substituting actual arguments for any formal parameters. The result of the evaluation, should there be one, is the function's output value.

Because a q function can modify global variables, q is not a pure functional language. A mathematical function can never reach outside its own body and have side effects. You should minimize such behavior for code maintainability.

### 6.1.1 Function Definition¶

The syntax of function definition is a matching pair of braces `{` and `}` enclosing (optional) parameters followed by a sequence of expressions. In contrast to statically typed languages, the input parameters and the return value are not explicitly typed; in fact, they don't even need to be declared. Even the function name is optional.

Following is a full specification of a function that squares its input. In this case, the function simply returns the value of the (last) expression evaluated. We add optional whitespace after the parameter declaration for readability.
    
    
```q
q){[x] x*x}
```
Apply this function by following it with an argument enclosed in square brackets. This causes the supplied argument to be substituted for all occurrences of the parameter `x`, the expression to be evaluated and the result returned as the output value.
    
    
```q
q){[x] x*x}[3]
9
```
A function is a first-class value – i.e., it is data just like a long or float. In particular, a function can be assigned to a variable, whereupon it acquires a name. This variable can be used in place of the function.
    
    
```q
q)f:{[x] x*x}
q)f[3]
_
```
### 6.1.2 Function Notation and Terminology¶

The formal notation for function definition is,

{[p1;...;pn] e1; ...; em}

The optional p1, ... , pn are formal parameters. The expressions _e 1_, ... , _e m_ – which presumably involve the parameters – represent the steps of an algorithm that are evaluated in order. Note that while the expressions are sequenced left-to-right, an individual expression is always evaluated right-to-left.

For readability, in this text we shall normally insert optional whitespace after the square bracket that closes the parameter list, as well as after each semi-colon separator. Other styles may differ.

Semi-colons

The semi-colons in the body are separators, not terminators. Do not place a semi-colon after the last expression unless you deliberately want to suppress the return value.

In fact, `;` is an operator that evaluates its left operand and discards the result, then evaluates the right operand and returns that value. If you meditate on this, you will realize that this is precisely the behavior of the semi-colon separators in a function body.
    
    
```q
q)a:2*3;6*7
42
q)a
6
```
The number of formal input parameters (implicit or explicit) is the function _valence_. Most common are _unary_ (valence 1) and _binary_ (valence 2).

A _nullary_ function in q is a function that has no meaningful input. This is similar to a function taking `void` in C. In functional programming this is represented as a function defined on a type called "unit" having a single value, often written `()`. In q it is written with an empty parameter declaration,
    
    
```q
f:{[] … }
```
and is applied with empty argument – i.e., `f[]`. In this case, the function either returns a constant or has _side effects_ , meaning that it accesses resources outside its body.
    
    
```q
q){[] 42} / pure function returns constant 42
_
q){[] a*a} / impure function: references global a
_
```
The maximum valence permitted as of this writing (Sep 2015) is 8; specifying more than eight parameters will cause an error. You can circumvent this restriction by encapsulating multiple parameters in a list or dictionary.

The output value of a function written in functional programming style is the value of the last expression in its body. This is the recommended style of writing q since it results in linear flow that is easy to follow and debug.
    
    
```q
q){[x] x*x}
q){[x;y] a:x*x; b:y*y; r:a+b; r}
```
This convention can be supplanted by using a unary overload of `:` that immediately terminates function evaluation and returns the value of the expression to its right. This style is discouraged.
    
    
```q
q){[x] :x*x} / unnecessary use of :
q){[x] a:1; :x*x; b:3} / dead code
```
Keep it short

Q functions should be compact and modular: each function should perform a well-defined unit of work. Due to the power of q operators and built-in functions, many functions are one line. 

When a function exceeds 20 statements, you should look to refactor it.

Ambivalence

When Arthur Whitney, the creator of q, says that q is not ambivalent, he doesn’t mean that q is decisive. Rather, in contrast to k, q operators and built-in functions are _not_ overloaded on valence, meaning that the same function does not have different behavior in a unary and binary form. That said, q operators and built-in functions _are_ overloaded on just about anything else, including the type, sign or rank of arguments – sometimes multiply so.

### 6.1.3 Function Application¶

The notation for function application parallels that of the formal parameters in its definition. The function is followed by square brackets enclosing arguments separated by semi-colons. Application causes the expressions in the function body to be evaluated in sequence, substituting the value of each argument for the corresponding formal parameter.
    
    
```q
q){[x] x*x}[3]
9
q)f:{[x] x*x}
q)f[4]
_
q){[x;y] x+y}[3;4]
7
q)g:{[x;y] x+y}
q)g[3;4]
_
```
Function application in q is _strict_ , meaning that expressions in arguments are evaluated **before** substitution is performed.
    
    
```q
q)f:{[x] x*x}
q)f[0N!3+1]
4
16
```
The valence of a function is also called its _rank_. An application with too many arguments generates a `'rank` error. This doesn’t mean your code stinks, although it might.
    
    
```q
q) {[x] x*x}[3;4]
'rank
```
Apply a nullary function with no arguments or with an arbitrary argument, which is ignored.
    
    
```q
q)const42:{[] 42}
q)const42[]
_
q)const42[98.6]
_
```
### 6.1.4 Functions with No Return Value¶

Every q function has a return value but sometimes the value carries no meaningful information – i.e., it is used purely for side effects. For example, it may send off a message asynchronously without waiting for an acknowledgement of receipt or it may update a global table. To achieve the equivalent of a `void` return in C, make the expression after the last semi-colon in the function body empty.
    
    
```q
q)fvoid:{[x] `a set x;}
q)fvoid 42
q)
```
In fact, the actual return value is the nil item `::` that is a stand-in for `void` in this situation. Its display is suppressed in the q console but it can be revealed.
    
    
```q
q).Q.s1 fvoid 42
"::"
```
Semi-colon terminators

A common error made by qbies is to use semi-colons in a function body as expression terminators rather than as separators. This effectively makes the function return nil since the last expression is empty.

### 6.1.5 We Don't Need No Stinkin’ Brackets¶

There is an alternative notation for application of unary functions (only) that dispenses with brackets. Simply separate the function from its argument with whitespace – customarily a single blank.
    
    
```q
q){[x] 2*x} 42
84
q)f:{[x] x*x}
q)f 5
_
```
This convention for function application is called _prefix syntax_ and is the norm in functional programming. While it may be jarring for newcomers, the decrease in superfluous punctuation – i.e., program noise – is worth the effort.

### 6.1.6 Application of a Name¶

When a function has been assigned to a global variable, it can be applied by name – i.e., use the symbol name of the variable in place of the variable itself.
    
    
```q
q)f:{x*x}
q)f[5]
_
q)`f[5]
25
q)`f 5
_
q).my.name.space.f:{2*x}
q)`.my.name.space.f[5]
_
```
This works because a global variable is implemented as a symbolic-name/function pair in a dictionary. Referring to the variable is actually lookup-by-name. Consequently application by name does **not** work for local variables or q system variables.

Do not confuse application using the function name with call-by-name in which parameters are passed by name.

### 6.1.7 Implicit Parameters¶

If, like mathematicians, you are satisfied with generic parameters _x_ , _y_ and _z_ , you can omit the formal parameters and their brackets. Three implicit positional parameters `x`, `y` and `z` are automatically defined and available in a function’s expressions when no parameters are explicitly declared. Thus, the following two specifications result in equivalent input-output relations.
    
    
```q
{[x] x*x}
{x*x}
```
As do,
    
    
```q
{[x;y] x+y}
{x+y}
```
When using implicit parameters, `x` is always substituted by the first actual argument, `y` the second and `z` the third.

The following function `f` only evaluates partially unless it is called with three parameters.
    
    
```q
q)g:{x+z}  / likely meant x+y; requires 3 args in call
q)g[1;2]   / still waiting for 3rd arg – i.e., a projection
{x+z}[1;2]
q)g[1;2;3] / 2nd arg is required but ignored
4
```
Only use the names `x`, `y` and `z` for the first three parameters of a function, either explicit or implicit.

Any other use leads down the path to perdition.
    
    
```q
q)evil:{[y;z;x] ...} / don't do this!
```
### 6.1.8 Anonymous Functions and Lambda Expressions¶

Many traditional programming languages include the function name as part of its definition. A function without a name is then called an _anonymous_ function. In functional programming, functions are all anonymous and are called _lambda expressions_ or _lambdas_ for short. All q functions are anonymous until assigned to a variable. Incidentally, the term “lambda” actually arose from a typesetting error in Church’s early work on what became known as the _lambda calculus_.

An anonymous function can be useful as an in-line macro where it would otherwise be awkward to substitute. For example,
    
    
```q
f{[...] ...; {...}[...]; ...}
```
It is arguably more readable to factor this out.

Another use for anonymous functions is a rudimentary form of dynamic dispatch, in which functions are placed in a collection (e.g., list, dictionary, table) and then selected dynamically (i.e., at runtime) rather than by an intrinsic name.
    
    
```q
q)powers:({1}; {x}; {x*x}; {x*x*x})
…
q)selected:2
q)powers[selected]
{x*x}
```
### 6.1.9 The Identity Function `::`¶

When `::` is used with function application syntax, it is the identity function – i.e., it returns its input as output.

The naked identity function cannot be used with prefix syntax; it must be enclosed in parentheses.
    
    
```q
q)::[42]
_
q)::[`a`b`c]
_
q):: 42 / error
'
q)(::) 42
_
```
### 6.1.10 Functions Are Data¶

The q data entities we have met until now are atoms (of various types), lists and dictionaries. For those new to functional programming, it may come as a surprise that functions are also data. A function can be passed around just like a long or float – e.g., as an item in a list.
    
    
```q
q)(1; 98.6; {x*x})
_
q)f:{x*x}
q)(f; neg)
_
q)(f; neg)[0]
q)(f; neg)[1; 5]
_
```
A function can also be used as the input or output value of another function. For example, here is a function that expects a function and a value and applies the function to the value.
    
    
```q
q)apply:{x y}
q)sq:{x*x}
q)apply[sq; 5]
_
```
A function that operates on other functions is called a _higher-order function_. Higher-order functions are a powerful concept from functional programming. The derivative and indefinite integral are higher-order functions in elementary calculus.

## 6.2 Call-by-Name¶

Ordinary function application in q uses call-by-value, meaning that arguments are reduced to values at the start of application. In the process, copies are made so that any original values are undisturbed. When the size of the input data is large, this copy may be prohibitive. In this case, there is an alternative method of application, _call-by-name_ , in which names of (global) variables are passed instead of their values.

There is no special syntax for call-by-name in q. Instead, the function implementation simply expects a symbol containing the name of a variable and handles it appropriately. One example of a function that uses call-by-name is the built-in function [`get`](https://code.kx.com/q/ref/get/), which returns the value of the global variable whose name is passed.
    
    
```q
q)a:42
q)get `a
42
```
Another example is binary [`set`](https://code.kx.com/q/ref/get/#set), which does the actual work of global assignment. It expects the name of a global variable and the value to be assigned.
    
    
```q
q)`a set 43
`a
q)a
43
```
The result of any built-in call-by-name function application is the symbol that was passed as input. This enables call-by-name functions to be chained – i.e., composed.

Do not confuse the return value of call-by-name with an error message. The former begins with a back-tick whereas the latter begins with tick.

## 6.3 Local and Global Variables¶

### 6.3.1 Defining Local and Global Variables¶

A variable that is assigned with `:` within a function body is called a _local_ variable. For example, a is a local variable in the following function.
    
    
```q
q)f:{a:42; a+x}
```
As of this writing (Sep 2015), the maximum number of local variables permitted in a function is 24.

Some notes on local variables:

  * A local variable exists only for the duration of an application. There is no notion of a static local variable that maintains its value across applications.
  * A local variable is not visible outside its immediate scope of definition.
  * A local variable cannot be used as an argument to a q function that uses call-by-name.
  * A local variable is not visible within the body of a local function defined in the same scope. Otherwise put,

**q does _not_ have lexical scoping**.




Here is a simple example demonstrating the lack of lexical scoping and its consequence. The function `f` defines a local variable `a` and a local function `helper`. In functional programming languages, the variable `a` is accessible throughout its scope of definition, which is the body of `f`. In particular, the variable `a` would be accessible within the body of `helper`.
    
    
```q
q)f:{[p1] a:42; helper:{[p2] a*p2}; helper p1}
```
In q it is not. Nasty surprise!
    
    
```q
q)f 5
{[p2] a*p2}
'a
```
Instead, you must declare an additional parameter in the local functions and pass the local variable as an argument. Projection provides a relatively clean way to do this.
    
    
```q
q)f:{[p1] a:42; helper:{[a; p2] a*p2}[a;]; helper p1}
q)f 5
210
```
A variable assigned outside any function definition is called a _global_ variable. Global variables are visible inside any function body.
    
    
```q
q)b:7
q)f:{b*x}
q)f[6]
_
```
Maximum number of global variables

As of this writing (Sep 2015), the maximum number of global variables that can be referenced in a function is 32. 

If this is a problem, redesign your code. For example collect values in a list or dictionary and pass that; the dictionary essentially provides pass-by-name.

### 6.3.2 Assigning Global Variables within a Function¶

Although functional programming purists will cringe, you can assign a global within a function body. Some q programmers mistakenly use double colon `::`, which works provided there is no local variable with the same name.
    
    
```q
q)b:6
q)f:{b::7; x*b}
q)f[6]
42
q)b
7
```
This usage of `::` is analogous to other forms of _amend-in-place_ , such as `+:` or `,:`, but using the assignment operator itself `:`. This accounts for its somewhat unintuitive properties – it is not truly a global assignment operator.

If there is a local variable having the same name as the global you intend to assign, double-colon assigns the **local** , not the global.
    
    
```q
q)b:6
q)f:{b:42; b::x; b}
q)f[98]
98
q)b
6
```
Global assignment

We recommend against `::` for global assignment. Instead, use `set` which truly does global assignment and doesn’t get hung up on a local-global name collision.

As we saw earlier, `set` uses call-by-name. 
    
    
```q
q)a:42
q)f:{a:98.6; `a set x}
q)f 43
`a
q)a
43
```
## 6.4 Projection¶

Projection means specifying only some of the parameters in function application, resulting in a function of the remaining parameters.

### 6.4.1 Function Projection¶

In some circumstances, a portion of the arguments to a multivalent function may be fixed while the remaining arguments vary. A common example is where a date is the last parameter and you apply the function for many dates keeping all other inputs fixed. In this situation, you can specify the fixed arguments and leave the others for later. This is called _projecting_ the function onto the remaining arguments. Notationally, projection is a partial application in which some arguments are supplied and the others are omitted – i.e., nothing is in those slots.

For example, a homegrown binary function that adds its arguments can be projected onto the second parameter by providing just the first argument.
    
    
```q
q)add:{x+y}
q)add[42;]
{x+y}[42;]
```
You can see from the console that q views the projection as a partial application that awaits its remaining argument(s). The projected function is a legitimate unary function, which adds its input to the constant 42.
    
    
```q
q)add[42;][3]
45
```
Since functions are first class, we can assign a projection to a variable and then apply it, none the wiser of its origin.
    
    
```q
q)g:add[42;]
q)g[3]
_
```
Projection is related to _currying_ in functional programming (named after Haskell Curry who did important early work on the theory of combinators). Both provide realizations of the mathematical observation that a function of two parameters is equivalent to a function of the first parameter that returns a function of the second parameter. This extends to multiple parameters as well. In q you simply project out successive parameters.
    
    
```q
q)add3:{x+y+z}
q)add3[2][3][4]
9
```
Here we can view the result of `add3` applied to 2 as a function that when applied to 3 returns a function that when applied to 4 yields `add[2;3;4]`.

Style recommendation

We recommend that you do _not_ omit trailing semi-colons in projections, as this can obscure the intent of code. For example, if you came upon the following while reading q code, how would you know that `mystery` is actually a binary function unless you saw the comment line?
    
    
```q
q)mystery:f[2] / surprise! f is actually f:{x+y+z}
```
The brackets denoting a function projection are required, but the additional brackets in a unary projection’s application can be omitted using prefix syntax as with any function.
    
    
```q
q){x+y}[42;][3]
_
q){x+y}[42;] 3
_
```
Choice of notation is a matter of coding style. Some coders have good style; many do not.

The function body in a projection is captured at the time it is first encountered by the interpreter.

Thus if the underlying function is subsequently changed, the projection is not.
    
    
```q
q)f:{x-y}
q)g:f[42;]
q)g
{x-y}[42;]
q)g[6]
_
q)f:{x+y}
q)g
_
q)g[6]
_
```
### 6.4.2 Operator Projection¶

When used in infix form, a q operator can be projected by fixing its left operand (only). This requires parentheses. For example, the projection of `*` onto its right operand is,
    
    
```q
q)(7*) 6
42
```
Due to the overloading of parentheses in q, an operator cannot be projected by fixing its right argument, since this would lead to notational ambiguity. For example, `-42` can only be the atom -42 and not a projection.

Nonetheless, every operator is a function, so you can project it in prefix form.
    
    
```q
q)-[;42] 98
_
```
The whitespace is actually not necessary in prefix syntax here since the brackets make things unambiguous.
    
    
```q
q)(7*)6
_
q)-[;42]98
_
```
Such notation is an acquired taste not shared by the author.

### 6.4.3 Multiple Projections¶

Functions having valence greater than two can be projected in multiple ways. For example, a function of valence three can be projected onto the second parameter by specifying the first and third. The result is a unary function of the second parameter only.
    
    
```q
q){x+y+z}[1;;3]
{x+y+z}[1;;3]
q){x+y+z}[1;;3] 2
6
```
We could arrive at the same result in two steps.

  * _Step 1_ : Project onto the second and third parameters by specifying just the first argument. This yields a binary function.


    
    
```q
q){x+y+z}[1;;]
{x+y+z}[1;;]
q){x+y+z}[1;;][2;3]
_
```
  * _Step two_ : Project the resulting binary function onto its first parameter by providing only the second.


    
    
```q
q){x+y+z}[1;;][;3]
{x+y+z}[1;;][;3]
q){x+y+z}[1;;][;3] 2
_
```
This is equivalent to projecting in the reverse order.
    
    
```q
q){x+y+z}[;;3][1;]
{x+y+z}[;;3][1;]
q){x+y+z}[;;3][1;] 2
_
```
## 6.5 Everything Is a Map¶

Here we explore the deep relationship between q data structures and functions. While this section can be skipped on first reading, that would be like not eating your vegetables when you were a kid.

### 6.5.1 Similarity of Notation¶

You have no doubt noticed that the notation for list indexing, dictionary lookup and unary function application are identical. Namely,
    
    
```q
q)L:0 1 4 9 16 25 36
q)f:{x*x}
q)L[2]
_
q)L 2
_
q)f[2]
_
q)f 2
_
```
This is no accident. In [Chapter 3](../3_Lists/) we saw that a list is a map defined by positional retrieval via item indexing. In [Chapter 5](../5_Dictionaries/) we saw that a dictionary is a map defined by key-value lookup. A function is a map defined by a sequence of expressions representing an algorithm for computing an output value from the input. The common notation is simply that of a map. It may take a little time to get accustomed to the Zen of q.

### 6.5.2 List of Indices, Keys and Arguments¶

Just as we can use a list of indices to retrieve items from a list, so can dictionary lookup retrieve a list of values corresponding to a list of keys. And a list of arguments to an atomic function yields a list of outputs. This justifies the common notation.
    
    
```q
q)L:10 20 30 40 50
q)L[2 5]
_
q)I:2 5
q)L I
_
q)d:`a`b`c!10 20 30
q)ks:`a`c
q)d ks
_
q)f[2 5]
_
q)f I
_
```
In fact, these are all examples of composition of maps.
    
    
```q
  I     L
--------------
0 |-> 2 |-> 30
1 |-> 5 |-> 0N

  ks     d
---------------
0 |-> `a |-> 10
1 |-> `c |-> 30

  I     f
--------------
0 |-> 2 |-> 4
1 |-> 5 |-> 25
```
### 6.5.3 Indexing at Depth¶

Next we examine the correspondence between indexing at depth and multivalent function evaluation. Notationally, a nested list is a list of lists, but it can also be viewed functionally as multivariate position retrieval. The display of a nested list tells us how: go down then over.
    
    
```q
q)a:(1 2 3; 100 200)
q)a
1 2 3
100 200
q)a[1;0]
100
```
The first index goes down and the second index goes over. A more deeply nested list is a positional map whose valence is one greater than the depth of nesting.

Similarly, a dictionary with complex values can also be viewed as a multivariate map.
    
    
```q
q)d:`a`b!(1 2 3; 100 200)
q)d[`a;1]
2
```
In this case, the first parameter locates a key and the second goes into the value list.

### 6.5.4 Projection and Index Elision¶

You have no doubt noticed that the notations for [function projection](https://code.kx.com/q/basics/application/#projection) and elided indices in a list are identical. For example, an array is positional retrieval in which the first index is "down" and the second is "over." Thus eliding the first index indeed results in a list in which the remaining index is simply ordinary positional retrieval.
    
    
```q
q)m:(10 20 30; 100 200 300)
q)m
10  20  30
100 200 300
q)m[1;2]
_
q)L:m[;2]
q)L
_
q)L[1]
_
```
### 6.5.5 Out of Bounds Index¶

Viewing a list as a map on an integer domain also motivates the behavior of item indexing in the case of an out of bounds index. In traditional languages, this would either result in some sort of uncaught error – the infamous indexing off the end of an array – or an exception.

By viewing a list as a function defined on a sub-domain of integers, it is reasonable to extend the domain of the function to all integers by assigning a null output value outside the proper domain. Here null means "missing value." As we have seen previously,
    
    
```q
q)10 20 30 40[100]
0N
q)`a`b`c[-1]
`
q)(1.1; 1; `1)[3]
0n
```
The behavior for dictionaries is completely analogous.
    
    
```q
q)d:`a`b`c!10 20 30
q)d[`x]
0N
```
## 6.6 Atomic Functions¶

The characterization of q as a vector language arises from the fact that most of the built-in operations are atomic. We examine the notion of [atomic functions](https://code.kx.com/q/basics/atomic/) in more detail. The application of an atomic function is characterized by the fact that it recurses into an argument’s structure until it gets to atoms and then applies there. It does this without explicit loops or other control flow constructs.

### 6.6.1 Unary Atomic Functions and "map"¶

Atomic behavior is easiest to see with a unary function.
    
    
```q
q)neg 10
-10
q)neg 10 20 30
-10 -20 -30
q)neg (10 20 30; 40 50)
-10 -20 -30
-40 -50
q)neg `a`b`c!10 20 30
a| -10
b| -20
c| -30
q)neg `a`b`c!(10 20; 30 40 50; 60)
a| -10 -20
b| -30 -40 -50
c| -60
```
A consequence of this behavior is that the result of a unary atomic function always has the same shape as its argument – i.e., the output conforms to the input.

A special case of this is that applying an atomic function to a list is the same as applying it to each item in the list, which is sometimes described as "atomic functions extend automatically to lists." Similar functionality is achieved with "foreach" in traditional languages and the higher-order function "map" in functional languages.

### 6.6.2 Binary Atomic Functions and zip¶

Atomic behavior is more interesting for binary functions. A binary function can be atomic in either or both arguments. The first case can be thought of as a unary atomic function if you fix the non-atomic argument – i.e. project the function. For example, [Find](https://code.kx.com/q/ref/find/) (`?`) is atomic in only its second argument since it consumes the entire first argument in its search.
    
    
```q
q)10 20 30?10
0
q)10 20 30?10 20 30 40 50
0 1 2 3 3
q)(enlist 10)?10
0
q)10 20?10
0
q)10 20 30 40 50?10
0
```
The arithmetic, comparison and relation operators are all atomic in both operands. This leads to four cases for application:

  * atom with atom
  * atom with list
  * list with atom
  * list with list



In the last case, the two list operands must have the same length.
    
    
```q
q)1+10
11
q)1+10 20 30
11 21 31
q)1 2 3+10
11 12 13
q)1 2 3+10 20 30
11 22 33
```
Similar behavior is achieved with "zip" in traditional and functional programming.

### 6.6.3 Creating Atomic Functions¶

Brief meditation reveals that the composition of atomic functions is atomic. Hence, the guaranteed way to build your own atomic functions is to compose built-in atomic functions. First unary.
    
    
```q
q)f:{(x*x)+(2*x)-1}
q)f 0
-1
q)f til 10
-1 2 7 14 23 34 47 62 79 98
```
Now binary.
    
    
```q
q)pyth:{sqrt (x*x)+y*y}
q)pyth[1; 1]
1.414214
q)pyth[1; 1 2 3]
1.414214 2.236068 3.162278
q)pyth[1 2 3; 1 2 3]
1.414214 2.828427 4.242641
```
You might ask how to get this behavior with a function that is not atomic. Timely question.

## 6.7 Iterators¶

[Iterators](https://code.kx.com/q/ref/iterators/) (previously _adverbs_) are higher-order functions that modify the behavior of functions for application on lists. 

Proficiency in the use of iterators is one skill that separates q pretenders from q contenders.

Not every function is atomic, but we may still want to apply it to individual items of a data structure without loopy code. Iterators to the rescue.

### 6.7.1 Unary each¶

We have seen that an atomic function automatically applies to the atoms in a nested list.
    
    
```q
q)neg (1 2 3; 4 5)
-1 -2 -3
-4 -5
```
Clearly a function that aggregates a list into an atom cannot be atomic since it collapses the list structure. In particular, aggregate functions operate only at the top level of a nested list.
    
    
```q
q)count 10 20 30
_
q)count (10 20 30; 40 50)
_
```
Suppose instead that we want to count each of the two items in our nested list. This is precisely what the [`each`](https://code.kx.com/q/ref/each/) iterator does. It modifies a unary function so that it applies to each item in a list rather than to the list as a whole. When used infix, `each` follows the function after (required) whitespace.
    
    
```q
q)count each (10 20 30; 40 50)
3 2
```
The nature of `each` as a higher-order function is more clearly revealed in its equivalent prefix form. This may look odd at first, but it shows that `each` takes a function and modifies its behavior by applying to each item in a list.
    
    
```q
q)each[count] (10 20 30; 40 50)
3 2
```
Each

The higher-order function `each` is similar to "foreach" in imperative languages and is called "map" in functional programming languages. It is one half of the famous map-reduce paradigm.

We could say informally that `each` makes a non-atomic function act atomically… at least at one level. For deeply nested lists, you need to iterate `each` to apply a function at deeper levels.
    
    
```q
q)(count each) each ((1 2 3; 3 4); (100 200; 300 400 500))
3 2
2 3
q)each[each[count]] ((1 2 3; 3 4); (100 200; 300 400 500))
_
```
The effect of iterated `each` is perhaps most clearly observed when applied to a matrix, where successive higher-order applications cause the original function to be applied on another axis.
    
    
```q
q)count (1 2 3; 10 20 30)
_
q)count each (1 2 3; 10 20 30)
_
q)(count each) each (1 2 3; 10 20 30)
```
You can use `each` with any unary function, although it is redundant on atomic functions.
    
    
```q
q)reverse "live"
"evil"
q)reverse ("life"; "the"; "universe"; "and"; "everything")
_
q)reverse each ("life"; "the"; "universe"; "and"; "everything")
_
q)neg each (1 2 3; 4 5)
_
```
It often arises that you want to convert a vector of length n to an n×1 matrix. You can do that with `enlist each` but `flip enlist` executes faster for large lists.
    
    
```q
q)enlist each 1001 1002 1004 1003
1001
1002
1004
1003
q)flip enlist 1001 1002 1004 1003
_
```
### 6.7.2 Each `'`¶

An atomic operator automatically applies to corresponding pairs of items when applied to lists of the same length.
    
    
```q
q)1 2 3+10 20 30
11 22 33
```
The binary Join operator `,` is not atomic since it consumes (copies of) both of its operands in their entirety when it concatenates.
    
    
```q
q)"abc","de"
"abcde"
```
Suppose we want to join list items pair-wise. For example,
    
    
```q
q)("abc"; "uv"),'("de"; "xyz")
"abcde"
"uvxyz"
```
The iterator [Each `'`](https://code.kx.com/q/ref/maps/#each) modifies a binary function (operator, keyword) to apply pairwise to corresponding list items. Conventionally no whitespace should be between `'` and the function it modifies but this does not generate an error. In some programming languages, Each is called "zip".
    
    
```q
q)("abc"; "uv"),'("de"; "xyz")
"abcde"
"uvxyz"
```
There is no effect when the arguments are atoms.
    
    
```q
q)3,'4
3 4
```
A function modified by Each has the usual properties of atomic binary functions. For example, you get a length error if arguments don't line up.
    
    
```q
q)("abc"; "uv"),'("de"; "xyz"; "uhoh")
'length
```
The modified function also extends an atom to match a list. You will see this idiom often. For example,
    
    
```q
q)1,'10 20 30
1 10
1 20
1 30
q)1 2 3,'10
_
q)2#'("abcde"; "fgh"; "ijklm")
"ab"
"fg"
"ij"
```
If you are so inclined, you can apply a function modified by Each using brackets. This is not common but can simplify things in the functional form of queries (see [§9.12](../9_Queries_q-sql/#912-functional-forms)).
    
    
```q
q),'[("abc"; "uv"); ("de"; "xyz")]
_
```
The idiom `,'` works fine for making a list of pairs from two simple lists but does not always work as expected with general lists. For example it loses the enlisting of the singleton in the following.
    
    
```q
q)L1:(enlist `a; `b)
q)L2:1 2
q)L1,'L2
`a 1
`b 2
```
A reliable way to make a list of pairs from a pair of lists is:
    
    
```q
q)flip (L1; L2)
,`a 1
`b 2
```
Advanced

A nice example of `,'` arises when both operands are tables. Since a table is a list of records, it is possible to apply `,'` to tables with the same number of rows. The record dictionaries in each row are upserted, resulting in a sideways join of the tables, as in the following example.
    
    
```q
q)t1:([] c1:1 2 3)
q)t2:([] c2:`a`b`c)
q)t1,'t2
c1 c2
-----
1  a
2  b
3  c
```
### 6.7.3 Each Left `\:`¶

An atomic operator extends an atom in the right operand to match a list on the left.
    
    
```q
q)1 2 3+10
_
```
The [Each Left](https://code.kx.com/q/ref/maps/#each-left-and-each-right) iterator `\:` modifies a binary function so that it applies the second argument with each item of the first argument – for example, to append a string to each string in a list. We insert unnecessary whitespace around the modified function in the following example to make it more readable.
    
    
```q
q)("abc"; "de"; enlist "f") ,\: ">"
"abc>"
"de>"
"f>"
```
Traditionally there is no space between the iterator and the function it modifies but this causes no error.

### 6.7.4 Each Right `/:`¶

An atomic operator extends an atom in the left operand to match a list on the right.
    
    
```q
q)10+1 2 3
_
```
The [Each Right](https://code.kx.com/q/ref/maps/#each-left-and-each-right) iterator `/:` modifies a binary function so that it applies the first argument to each item of the second argument. For example, to prepend a string to each string in a list. Again we surround the modified function with unnecessary whitespace for readability.
    
    
```q
q)"</" ,/: ("abc"; "de"; enlist "f")
_
```
Composing the last two examples makes data XML-ey (say it aloud) without loops.
    
    
```q
q)"</",/:("abc"; "de"; enlist "f"),\:">"
_
```
### 6.7.5 Cross Product¶

A cross (Cartesian) product of two lists pairs each item on the left with each item on the right. If we compose Join with Each Right and Each Left, we are almost there, except that there is an extra level of nesting, which we eliminate with the built-in [`raze`](https://code.kx.com/q/ref/raze/).
    
    
```q
q)1 2 3,/:\:10 20
1 10 1 20
2 10 2 20
3 10 3 20
q)raze 1 2 3,/:\:10 20
1 10
1 20
2 10
2 20
3 10
3 20
```
The built-in operator [`cross`](https://code.kx.com/q/ref/cross/) computes this cross product.
    
    
```q
q)1 2 3 cross 10 20
_
```
Observe that we could alternatively compose Each Left with Each Right and would arrive at the transpose of the previous result.
    
    
```q
q)raze 1 2 3,\:/:10 20
1 10
2 10
3 10
1 20
2 20
3 20
```
### 6.7.6 Over (`/`) for Accumulation¶

The [Over](https://code.kx.com/q/ref/accumulators/) iterator `/` is a higher-order function that provides the principal mechanism for recursion in q. In its simplest form it modifies a binary function to accumulate results over a list.

Suppose we want to add the first 10 natural numbers starting at 1. In imperative programming this requires control flow: initialize a result variable, establish a loop counter and then loop while adding into the result until the counter reaches the end of the list. In functional programming, simply declare that you want to accumulate starting with an initial value of the accumulator. No counters, no tests, no loop. You say "what" to do rather than "how" to do it.

Let’s see this in q. For readability, we have included optional whitespace.
    
    
```q
q)0 +/ 1 2 3 4 5 6 7 8 9 10
55
```
In this expression, we apply a compound operator formed from `+` modified by Over `/`.

There can be no whitespace between the function and `/`, due to `/` also being used to begin a comment.

The initial accumulator value is the left operand and the list to accumulate over is the right operand. Under the covers, each item of the list is progressively added into the accumulator, moving across the list until the end is reached. The final accumulated value is returned. In our example, 1 is added into the initial accumulator value of 0 to get 1; then 2 is added to this value to get 3; and so on until the end of the list. Just like old-fashioned imperative code, except you never see it. Here is an instrumented form of accumulating sum. It uses the `0N!` operator to display the accumulator `x` paired with the next input `y` before returning their sum.
    
    
```q
q)addi:{0N!(x;y); x+y}
q)0 addi/ 1 2 3 4 5
0 1
1 2
3 3
6 4
10 5
15
```
Observe that as a higher-order function `/` takes a binary function and returns a related binary function. The operands of the modified function are no longer symmetric – i.e., the left operand is the initial accumulator value and the right operand is the list to be accumulated.

In functional programming languages Over is called _fold_ or _reduce_. Along with `each` this completes the famous map-reduce paradigm.

Often the generality of specifying the initial value of the accumulator is unnecessary. In our summation example, we could initialize the accumulator with the first list item and then accumulate the remaining items. The syntax for this form encloses the modified function in parentheses and omits the initial accumulator.
    
    
```q
q)(+/) 1 2 3 4 5 6 7 8 9 10
55
```
Another way to think of this is by representing the list in general form.

(1; 2; 3; 4; 5; 6; 7; 8; 9; 10)

Then `+/` effectively replaces the semi-colons with `+`, associated to the left.

A few notes about the alternate form of Over.

  * The parentheses are mandatory
  * The modified function in this form is unary
  * This construct is actually k and not q. Don’t tell anyone you are writing k.



Both the unary and binary forms can be written in prefix.
    
    
```q
q)+/[0; 1 2 3 4 5 6 7 8 9 10]
_
q)+/[1 2 3 4 5 6 7 8 9 10]
_
```
The basic Over pattern is powerful. Use various binary functions to obtain common aggregate operations.
    
    
```q
q)(*/) 1 2 3 4 5 6 7 8 9 10 / product
3628800
q)(|/) 7 8 4 3 10 2 1 9 5 6 / maximum
10
q)(&/) 7 8 4 3 10 2 1 9 5 6 / minimum
1
```
Common aggregates are given their own names in q.

aggregate | name  
---|---  
`+/` | `sum`  
`*/` | `prd`  
`|/` | `max`  
`&/` | `min`  
  
Anyone with information about the location of the missing ‘o’ in `prd` please notify KX.

The edge cases when the above aggregates are applied to empty lists are completely determined by the appropriate invariants. For example, for `|/` the following must hold for the join of any numeric lists `L1` and `L2`.
    
    
```q
max[L1,L2] = max[L1]|max[L2]
```
After a moment of meditation we realize that choosing either `L1` or `L2` to be an empty list of numeric type forces,
    
    
```q
q)(|/) `long$()
-0W
q)(|/) `float$()
-0w
```
Advanced: morphism

For those who know about such things, the above equation says that `|/` is a morphism from the free monoid of lists with concatenation `,` to the monoid of naturals with `|`. The edge case says the morphism preserves the identity. 

Other edge cases are analogous.
    
    
```q
q)(&/) `long\(()
0W
q)(+/) \`long\)()
_
q)(*/) `long$()
_
q)(,/) ()
_
```
Applying `,/` across a list concatenates the items, effectively eliminating the top level of nesting. If there is only one level of nesting, this flattens a list of lists to a plain list. The function `,/` goes by the descriptive name `raze`.
    
    
```q
q)(,/)((1 2 3; 4 5); (100 200; 300 400 500))
1 2 3
4 5
100 200
300 400 500
q)raze ((1 2 3; 4 5); (100 200; 300 400 500))
_
```
For those who know about monads, lists comprise a monad. In Haskell-speak, `each` serves as `fmap`, `enlist` as `return` and `,/` as `join`.

You can use your own function with either form of Over. The modified function can even be used infix, even though your original function cannot.
    
    
```q
q)f:{2*x+y}
q)100 f/ 1 2 3 4 5 6 7 8 9 10
_
q)(f/) 1 2 3 4 5 6 7 8 9 10
_
```
### 6.7.7 More Over: Iteration¶

The next pattern of recursion we investigate is another declarative equivalent of loopy code. In this version of `/`, the left operand is a natural number indicating how many times to perform the iteration and the right operand is the initial value.

Let’s compute the nth Fibonacci number – see [§1.12](../1_Q_Shock_and_Awe/#112-example-fibonacci-numbers) for a more detailed derivation of `fib`. Again the modified function can be used infix or prefix.
    
    
```q
q)fib:{x,sum -2#x}
q)10 fib/ 1 1
1 1 2 3 5 8 13 21 34 55 89 144
q)fib/[10; 1 1]
_
```
Yet another version of `/` runs the recursion until convergence, or until a loop is detected. Our example uses a simple form of Newton-Raphson – see [§1.13](../1_Q_Shock_and_Awe/#113-example-newtons-method-for-nth-roots) for a detailed exposition for nth roots. In this section we illustrate a variation of the square root in which we use a numerical approximation for the derivative. The approximation is the slope of the secant through points an epsilon to the left and right of _x_. This works provided we make epsilon sufficiently small because the derivative is defined as the slope of the tangent, which is the limit of the slope of the secant as epsilon approaches 0. Recall that in order to compute the square root of 2, we find the zero of `{-2+x*x}`.
    
    
```q
q)f:{-2+x*x}
q)secant:{[f;x;e] (f[x+e]-f x-e)%2*e}
q){x-f[x]%secant[f; x; 1e-6]}/[1.5]
1.414214
```
How does q determine convergence? At each step the result is compared to that of the previous step. If the two agree within equality tolerance (10-14 at the time of this writing – Sep 2015) the algorithm is considered to have converged and the recursion is complete; otherwise it continues.

Mathematically, it is possible to generate an infinite loop with Newton-Raphson. To wit, we select the function _x_ 3 – 2 _x_ \+ 2 and choose the initial value 0, which causes the algorithm to go into a cycle.
    
    
```q
q)newtcycle:{[xn] xn-((xn*xn*xn)+(-2*xn)+2)%-2+3*xn*xn}
q)newtcycle/[0.0]
1f
```
How did q detect the cycle and stop the iteration? At each step it compares the result with the initial value. If they match, a cycle has occurred and the recursion is halted; otherwise it continues. Note that we mean "match" literally as in the `~` operator.

Open loops

If the values are equal but of different type, no cycle is detected and the loop will continue. In our example, providing a non-float initial value will cause an infinite loop. You will have to kill your console session if you enter the following.
    
    
```q
q)newtcycle/[0] / oops
```
The final overload of `/` is equivalent to a “while” loop in imperative programming. It provides a declarative way to specify a test to end the iteration. Thinking functionally, we provide a predicate function that is applied to the result at each step. The iteration continues as long as the predicate result is `1b` and stops otherwise. For example, we stop the computation of the Fibonacci sequence once it exceeds 1000 as follows.
    
    
```q
q)fib:{x,sum -2#x}
q)fib/[{1000>last x}; 1 1]
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597
```
### 6.7.8 Scan `\`¶

The Scan iterator `\` is a higher-order function that behaves just like `/` except that it returns all the intermediate accumulations instead of just the final one. Whereas Over produces an aggregate function that collapses a list, Scan produces a function whose output has the same number of items as its input – i.e., a _uniform_ function. Informally, Scan is a “running” version of Over.

We omit optional whitespace in the following so that you can get accustomed to reading "real q."
    
    
```q
q)0+\1 2 3 4 5 6 7 8 9 10
1 3 6 10 15 21 28 36 45 55
q)(*\)1 2 3 4 5 6 7 8 9 10
_
q)(|\)7 8 4 3 10 2 1 9 5 6
_
q)(&\)7 8 4 3 10 2 1 9 5 6
_
q)100 f\1 2 3 4 5 6 7 8 9 10
_
q)(f\)1 2 3 4 5 6 7 8 9 10
_
```
All the variations of Over are also available with Scan. Switching to Scan can be useful to see intermediate results of an accumulation. For example, we can see how fast Newton-Raphson converges to the square root by setting the float display to unlimited precision and running Scan.
    
    
```q
q)newtsqrt[1.0]
q)\P 0
q)newtsqrt[1.0]
1 1.5 1.4166666666666667 1.4142156862745099 1.4142135623746899
1.4142135623730951
```
### 6.7.9 Each Prior (`':`)¶

The iterator [Each Prior](https://code.kx.com/q/ref/maps/#each-prior) `':` provides a declarative way to perform a binary operation on each item of a list with its predecessor.

During traversal of the list, the current item is the left operand and the previous item is the right operand.

Since the initial item of the list does not have a predecessor, we must provide one in the left operand of the modified operator. One choice is the initial item of the source list. For example, to calculate the deltas of a list of prices.
    
    
```q
q)100 -': 100 99 101 102 101
0 -1 2 1 -1
```
As with the other iterators, there is a unary form of Each Prior (technically, it is k). It may come as a surprise that the unary form does **not** choose the initial item of the input list as the initial predecessor. Rather, it returns the initial item of the input on `-` and `%`.
    
    
```q
q)(-':)100 99 101 102 101
100 -1 2 1 -1
q)deltas 100 99 101 102 101
100 -1 2 1 -1
q)(%':)100 99 101 102 101
100 0.98999999999999999 1.0202020202020201 1.0099009900990099 0.99019607843137258
q)ratios 100 99 101 102 101
_
```
The motivation is that the initial predecessor is implicitly chosen as the identity for the binary function (assuming such exists). This has the advantage of preserving desirable invariants.
    
    
```q
q)sums deltas 100 99 101 102 101
100 99 101 102 101
q)deltas sums 100 99 101 102 101
_
```
On the other hand, if you are looking for anomalies in absolute differences – e.g., price changes – you will get a nasty surprise.
    
    
```q
q)deltas 100 99 101 102 101
100 -1 2 1 -1
```
It is easy enough to make our own version for this case.
    
    
```q
q)deltas0:{first[x] -': x}
q)deltas0 100 99 101 102 101
0 -1 2 1 -1
```
A powerful idiom with Each Prior uses Match `~` to test if successive items are identical. It is actually more useful to know where they do not match; the latter is built in as [`differ`](https://code.kx.com/q/ref/differ).
    
    
```q
q)(~':) 1 1 1 2 2 3 4 5 5 5 6 6
011010001101b
q)not (~':) 1 1 1 2 2 3 4 5 5 5 6 6
100101110010b
q)differ 1 1 1 2 2 3 4 5 5 5 6 6
_
```
Observe that the `1b` values in the result of `differ` are the start points for runs of identical items, so `where` yields their indices. Then use `cut` to split them out.
    
    
```q
q)L:1 1 1 2 2 3 4 5 5 5 6 6
q)where differ L
0 3 5 6 7 10
q)(where differ L) cut L
1 1 1
2 2
,3
,4
5 5 5
6 6
```
Now let’s do some q.
    
    
```q
q)runs:(where differ L) cut L / store runs
q)ct:count each runs / store count of each run
q)runs where ct=max ct / find the runs of maximum length
1 1 1
5 5 5
```
While some q coders don’t like the style, we point out that the above can be written in a single line of q using intra-line assignments.
    
    
```q
q)runs where ct=max ct:count each runs:(where differ L) cut L
_
```
Using comparison and the correct initial value, we can also use the above technique to find runs of increasing or decreasing values.
    
    
```q
q)L:9 8 7 11 10 12 13
q)(where -0W>':L) cut L
9 8 7
11 10
,12
,13
q)(where 0W<':L) cut L
,9
,8
7 11
10 12 13
```
This is the _raison d’être_ for integer infinities.

## 6.8 General Application¶

We have seen that the syntactic forms of list indexing, key lookup and function application can use either square brackets or prefix syntax.
    
    
```q
q)L:(1 2;3 4 5; enlist 6)
q)L[0]
_
q)L 1
_
q)L[0 2]
_
q)L[1][2]
_
q)L[1; 2]
_
q)d:`a`b`c!(1 2; 3 4 5; enlist 6)
q)d[`a]
1 2
q)d `a
_
q)d[`a`c]
_
q)d[`a][1]
_
q)d[`a; 1]
_
q)f:{x*x}
q)f[0]
_
q)f 1
_
q)f[0 2]
_
q)g:{x+y}
q)g[1][2]
_
q)g[1;2]
)
```
It turns out that both forms are syntactic sugar for how q actually treats application. You won't be surprised that application is actually a higher-order function that takes a function and value(s) and evaluates the function at the value(s). Thorough understanding of the general application is another test that separates the q pretenders from the contenders. The simpler forms of general application are usually written infix and are traditionally considered to be operators.

### 6.8.1 Operator `@`¶

[Apply At](https://code.kx.com/q/ref/apply/#apply-at)

Basic application comprises retrieving a list item by index, looking up a dictionary value by key, or evaluating a unary function. These are all instances of application of a univalent mathematical mapping. These are to be distinguished from indexing at depth, which is discussed in the next section.

The higher-order function `@` is the true form of basic application in q. It applies a unary mapping to an argument. As with all built-in functions, it can be written infix or prefix.
    
    
```q
q)10 20 30 40@1
20
q)L:10 20 30 40
q)L@1
_
q)@[L; 1]
_
q)count@L
_
q)@[count; L]
_
q){x*x}@L
_
q)d:`a`b`c!10 20 30
q)d@`a
_
q)@[d;`b]
_
```
Applying a nullary function with `@` is best done using the nil argument `::` but any argument suffices since it is ignored.
    
    
```q
q)f:{6*7}
q)f[]
_
q)@[f; ::]
_
q)f@(::)
_
q)f@43
42
```
For contrast with vector application in the next section, we collect previously discussed facts about ordinary application with a non-atom argument.

  * For lists, dictionaries and atomic functions, `@` application yields an output that conforms to the shape of the input.
  * An aggregate function collapses the top level of nesting from the input.
  * A uniform function has output the same length as an input list.



Apply At and tables

The function `@` also applies to tables and keyed tables since they are lists and dictionaries, respectively. Since a table is a list of records, using `@` retrieves records; for dictionaries and keyed tables it looks up keys:
    
    
```q
q)t:([]c1:1 2 3; c2:`a`b`c)
q)t@1
c1| 2
c2| `b
q)d:`a`b`c!10 20 30
q)d@`b
_
q)kt:([k:`a`b`c] v:1.1 2.2 3.3)
q)kt@`c
v| 3.3
```
### 6.8.2 Operator Apply `.`¶

[Apply](https://code.kx.com/q/ref/apply/)

Indexing a list at depth, retrieving a nested value from a dictionary and evaluating a function with multiple parameters are all instances of applying a multi-variate mapping. The higher-order function `.` is the true form of multi-variate application in q. It applies a multi-variate mapping to multiple arguments and can be written infix or prefix.

The right argument of `.` must be a list.

Separate operator `.` from its operands by whitespace if they are names or literal constants, as proximity to dots used in name spacing and in decimal numbers can lead to confusion.

Here we show various forms of indexing at depth rewritten with `.`.
    
    
```q
q)L:(10 20 30; 40 50)
q)L[1][0]
40
q)L[1; 0]
_
q)L . 1 0
_
q)d:`a`b`c!(10 20 30; 40 50; enlist 60)
q)d[`b][0]
40
q)d[`b; 0]
_
q)d . (`b; 0)
_
q)g:{x+y}
q)g[1; 2]
_
q)g . 1 2
_
```
Observe that operator Apply effectively allows us to apply a multi-variate function to a list of arguments instead of multiple individual arguments. Mathematically, this means that we are thinking of the function as defined on a vector instead of its individual components.

Dot application provides function application syntax that is conveniently independent of the valence of the function. This is useful for situations in which a function and its arguments are generated dynamically – e.g., dynamic dispatch – and the valence cannot be known in advance.
    
    
```q
q)g:{x+y}
q)f:g
q)f . 1 2
_
q)h:{x+y+z}
q)f:h
q)f . 1 2 3
_
```
You can apply a unary function with `.`. The insight is that a function of a scalar is inter-convertible with a function of a singleton vector. This is realized in q if you enlist the argument and use `.` for application.
    
    
```q
q)f:{x*x}
q)f@5
_
q)f . enlist 5
_
q)f . enlist 1 2 3
1 4 9
```
We conclude that `.` can apply a function of any valence and that `@` is actually unnecessary. Most q programmers do use `@` where applicable (pun intended).

To denote an elided index with `.` application, use the nil item.
    
    
```q
q)m:(1 2 3;4 5 6)
q)m[0;]
_
q)m . (0; ::)
_
q)m . (::; 1)
_
```
Evaluating a nullary function with `.` should properly use the enlisted nil item, although any enlisted value will suffice since it is ignored.
    
    
```q
q)f:{6*7}
q)f . enlist (::)
_
q)f . enlist 42
_
```
Advanced

Let’s climb the philosophical mountain. All data structures in q are composed from lists and dictionaries. Because q views both lists and dictionaries as mathematical mappings, retrieval from each is function application. Retrieval from un-nested lists and dictionaries is unary application with `@`. Retrieval from nested entities is multivariate application with `.`. In light of the previous observation that a unary function can be viewed as a function on a singleton vector, the function `.` provides a mechanism for retrieval from an arbitrary data structure. Gazing out from the summit, we see that all application is actually `.`.

Following are some examples of complex data structures and indexing at depth. See if you can predict them before entering at the console. (They make good q test questions)
    
    
```q
q)L:10 20 30
q)L . enlist 1
_
q)m:(10 20 30; 100 200 300)
q)m . 0 1
_
q)ds:(`a`b`c!10 20 30; `x`y!100 200)
q)ds . (0; `b)
_
q)mix:(10 20 30; `a`b`c!(1; 2; (300 400)))
q)mix . (1; `c; 1)
_
q)dc:`c1`c2!(1 2 3; `a`b`c)
q)dc . (`c2; 1)
_
q)t:([]c1:1 2 3;c2:`a`b`c)
q)t . (1; `c2)
_
```
And for keyed tables (see [§8.4](../8_Tables/#84-primary-keys-and-keyed-tables)).
    
    
```q
q)kt:([k:`a`b`c] v:1.1 2.2 3.3)
q)kt . `b`v
_
```
Indexing at depth

Note that indexing at depth fails for accessing items inside a nested column.
    
    
```q
q)t:([] c1:`a`b`c; c2:(1 2; 100 200 400; enlist 1000))
q)t . (1; `c2)
100 200 400
q)t . (1; `c2; 1)
'rank
```
(Until V3.5, that is. _Ed._) 
    
    
```q
q)t . (1; `c2; 1)
200
```
### 6.8.3 General Form of Function Application¶

[Amend, Amend At](https://code.kx.com/q/ref/amend/)

The uses of `@` and `.` seen heretofore are actually special cases of more general higher-order functions of greater valence. Since these must be written prefix, we begin by writing some simple examples of function application from the previous section into prefix form. We use lists for simplicity but other data structures are also applicable.
    
    
```q
q)L:10 20 30 40 50
q)@[L; 1]
20
q)@[L; 0 2]
10 30
q)m:(10 20 30; 100 200 300)
q).[m; 0 2]
30
```
Each application above can be viewed as **retrieval** from the data structure along a sub-domain of its definition. From this perspective, the general forms to come will apply an **arbitrary operation** along a sub-domain. Otherwise said, they modify a substructure **within** the original structure. This is a very powerful technique that is not present in traditional programming languages.

### 6.8.4 General Apply (`@`) with Unary Functions¶

We restate the `@` retrieval examples from the previous section.
    
    
```q
q)L:10 20 30 40 50
q)@[L; 1]
20
q)@[L; 0 2]
10 30
```
Now instead of simply retrieving along the sub-domain, we supply a function to be **applied** along the sub-domain.
    
    
```q
q)@[L; 1; neg]
10 -20 30 40 50
q)@[L; 0 2; neg]
-10 20 -30 40 50
```
Effectively we reach inside (a copy of) the list, apply `neg` along the specified sub-domain, and return the modified list. Contrast this with normal application on the sub-domain, which returns only the modified items.
    
    
```q
q)neg L@0 1
-10 -20
```
The significance of enhanced application is that it returns the entire modified list, which allows the result to be chained (i.e., composed) into further operations. If you don't think this is a big deal, think again.

The syntax for general application of a unary atomic function on a list is,
    
    
```q
@[L;I;f]
```
where `L` is the list and `I` is a collection of indices into `L`. Viewing `L` as a mapping, `I` is a top-level sub-domain of `L`. In fact, this form generalizes to any data structure viewed as a mapping. For example, given a dictionary and a list of keys,
    
    
```q
q)d:`a`b`c!10 20 30
q)ks:`a`c
q)@[d; ks; neg]
a| -10
b| 20
c| -30
```
Compound data structures

For compound data structures such as nested lists, tables and keyed tables, application of `@` occurs along a sub-domain at the top level. We’ll see how to reach down into data structure in the next section.
    
    
```q
q)m:(10 20 30; 100 200 300; 1000 2000 3000)
q)@[m; 0 2; neg]
-10 -20 -30
100 200 300
-1000 -2000 -3000
```
All the examples we have shown thus far work against copies of the input data structure. It is also possible to modify the original data structure in place. Although q does not have references, general application provides pass-by-name with the name being a symbol. We demonstrate with a simple example. In the first application, the original `L` is unchanged; in the second it is modified.
    
    
```q
q)L:10 20 30 40
q)@[L; 0; neg]
-10 20 30 40
q)L
10 20 30 40
q)@[`L; 0 ; neg]
`L
q)L
-10 20 30 40
```
Observe that the result of an application of pass-by-name is the passed symbolic name. This allows chaining of function applications that modify in place.

### 6.8.5 General Apply (`@`) with Binary Functions¶

In the previous section we saw how to apply a **unary** function within the top level of a data structure. In the same way, we can apply a binary atomic function using `@` by providing the function together with an operand to apply along the sub-domain. Clearly the shape of the supplied operand must conform to the specified sub-domain. The exception is applying with an atom, in which case the latter is automatically extended to match the sub-domain.
    
    
```q
q)L:10 20 30 40
q)@[L; 0 1; +; 100 200]
110 220 30 40
q)@[L; 0 1; +; 100]
_
q)d:`a`b`c!10 20 30
q)@[d; `a`b; +; 100 200]
a| 110
b| 220
c| 30
q)@[d; `a`b; +; 100]
_
```
The general form of functional `@` for a binary atomic function is,
    
    
```q
@[L; I; g; v]
```
The notation is suggestive of lists, but `L` can be any data structure, viewed as a mapping. Here `I` is a top-level sub-domain of `L`; `g` is a binary function; and `v` is an atom or list conforming to `I`.

We contrast application along a sub-domain with normal application on the extracted substructure.
    
    
```q
q)L:10 20 30 40
q)@[L; 0 1; +; 100 200]
110 220 30 40
q)100 200+L@0 1
110 220
```
As in the unary case, application of `@` along a sub-domain of compound data structures (e.g., nested lists, tables and keyed tables) occurs at the top Level.
    
    
```q
q)m:(10 20 30; 100 200 300; 1000 2000 3000)
q)@[m; 0 2; +; 1 2]
11 21 31
100 200 300
1002 2002 3002
```
A useful case of binary application uses the assignment operator `:` to modify values along the sub-domain.
    
    
```q
q)L:10 20 30 40
q)@[L; 0 2; :; 42 43]
42 20 43 40
```
As in the unary case, to apply to the original in place, instead of to a copy, use pass-by-name.
    
    
```q
q)L:10 20 30 40
q)@[`L; 0 2; :; 42 43]
`L
q)L
42 20 43 40
```
### 6.8.6 General Apply (`.`) for Unary Functions¶

We have seen that general `@` applies a function on a vector along the top level of a data structure. Now consider indexing at depth.

Although the construct is valid at any depth, there is a case to be made for not going beyond depth two in q (i.e., data normalization with tables).

We begin by restating simple examples of indexing at depth in prefix notation, using moderately nested lists and dictionaries for simplicity.
    
    
```q
q)m:(10 20 30; 100 200 300)
q).[m; 0 1]
20
q)d:`a`b`c!(10 20 30; 40 50; enlist 60)
q).[d; (`a; 1)]
20
```
In contrast with `@`, the vector argument of `.` reaches down into the data structure and picks out a single point in the domain. Here we target that point with a unary function.
    
    
```q
q).[m; 0 1; neg]
10 -20 30
100 200 300
q).[d; (`a; 1); neg]
a| 10 -20 30
b| 40 50
c| ,60
```
As before, these applications work against a copy of the data. To modify the original in place, `.` supports pass-by-name.
    
    
```q
q).[m; 0 1; neg]
q)m
_
q).[`m; 0 1; neg]
`m
q)m
_
q).[d; (`a; 1); neg]
_
q)d
_
q).[`d; (`a; 1); neg]
`d
q)d
_
```
We previously saw that to elide an index in `.` retrieval, we place the nil item `::` in that slot. The same holds for general application, where it means apply the function along **all** indices at that level.
    
    
```q
q).[m; (0; ::); neg]
-10 -20 -30
100 200 300
q)d:`a`b`c!(100 200 300; 400 500; enlist 600)
q).[d; (`a; ::); neg]
a| -100 -200 -300
b| 400 500
c| ,600
q).[d; (::; 0); neg]
a| -100 200 300
b| -400 500
c| ,-600
```
The general form of `.` for unary functions is,
    
    
```q
.[L; I; f]
```
Here `L` is a data structure, `I` is an in-depth sub-domain of `L` and `f` is a unary atomic function.

### 6.8.7 General Apply (`.`) for Binary Functions¶

By now you can predict how general application `.` with a binary function works. The form of functional `.` for a binary function is,
    
    
```q
.[L; I; g; v]
```
Here `L` is a data structure, `I` is an in-depth sub-domain of `L`, `g` is a binary function and `v` is an atom (for `g` atomic) or a vector that conforms to `I`. To apply in place use pass-by-name.
    
    
```q
q)m:(10 20 30; 100 200 300)
q).[m; 0 1; +; 1]
10 21 30
100 200 300
q).[m; (::; 1); +; 1 2]
10 21 30
_
q)m
_
q).[`m; (::; 1); +; 1]
_
q)m
_
q).[`m; (::; 1); :; 42]
`m
q)m
_
q)d:`a`b`c!(100 200 300; 400 500; enlist 600)
q).[d; (`a; 1); +; 1]
_
q).[d; (`a; ::); +; 1]
_
q).[d; (::; 0); +; 1]
_
q)d
_
q).[`d; (::; 0); :; 42]
_
q)d
_
```
Back to top 
