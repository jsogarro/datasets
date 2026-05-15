# 0\. Overview¶

## 0.0 The Evolution of Q¶

Arthur Whitney developed the q programming language and its database kdb+. Released by Kx Systems, Inc. in 2003, the primary design objectives of q are expressiveness, speed and efficiency. In these, it is beyond compare. The design trade-off is a terseness that can be disconcerting to programmers coming from verbose traditional database programming environments – e.g., C++, Java, C# or Python – and a relational DBMS. Whereas the q programming gods revel in programs resembling an ASCII core dump, this manual is for the rest of us.

Q evolved from [APL](https://en.wikipedia.org/wiki/APL_\(programming_language\)) (A Programming Language), which was first invented as a mathematical notation by [Kenneth Iverson](https://en.wikipedia.org/wiki/Kenneth_E._Iverson) at Harvard University in the 1950s. APL was introduced in the 1960s by IBM as a vector programming language, meaning that it processes a list of numbers in a single operation. It was successful in finance and other industries that required heavy number crunching.

The mitochondrial DNA of q traces from APL to A to [A+](http://www.aplusdev.org/) and to k. All were well suited to performing complex calculations quickly on vectors. What's new in q/kdb+ is that it processes large volumes of time-series data very efficiently in the relational paradigm. Its syntax allows "select" expressions that are similar to SQL 92, and its collection of built-in functions provides a complete and powerful stored procedure language.

There is also some [Lisp](https://en.wikipedia.org/wiki/Lisp_\(programming_language\)) in q's genes: the fundamental data construct of q is a list. Although the notation and terminology are different, symbols are drawn from their counterparts in Scheme.

The APL lineage of q also shows the influence of functional programming. In his 1977 Turing Award lecture that introduced purely functional programming, [Backus](https://en.wikipedia.org/wiki/John_Backus) acknowledged inspiration from APL. While q is not purely functional, it is strongly functional in that even its basic data structures, list and dictionary, are viewed as mathematical mappings.

## 0.1 Philosophy¶

A proficient q developer thinks differently than in conventional programming environments such as C++, Java, C# or Python, henceforth referred to as "traditional programming." In order to get you into the correct mindset, we summarize some of the potential discontinuities for the q newbie – henceforth known as qbie.

Recall some of the data-related issues in traditional database programming:

  * In-memory representation – e.g., collections of objects – must be mapped to a different representation – i.e., tables – for persistence. It takes considerable effort to get the object-relational correspondence correct.
  * Objects must be mapped to another representation for transport, usually some binary or XML form that flattens reference chains.
  * Data manipulation – e.g., selection, grouping and aggregation – on large data sets is best done in stored procedures on the database server. Demanding numeric calculations are best done apart from the database on an application server.
  * Data transformation for GUI display is best done in a separate layer – e.g., to HTML5 and JavaScript in a browser.



Much of traditional programming design is spent getting the various representations correct, requiring many lines of code to marshal resources and synchronize the different representations. These are remarkably simple in q/kdb+.

**Interpreted** Q is interpreted, not compiled. During execution, data and functions live in an in-memory workspace. Iterations of the development cycle tend to be quick because all run-time information needed to test and debug is immediately available in the workspace. Q programs are stored and executed as simple text files called scripts. The interpreter's `eval` and `parse` routines are exposed so that you can dynamically generate code in a controlled manner.

**Types** Q is a dynamically typed language, in which type checking is mostly unobtrusive. Each variable has the type of its currently assigned value and type promotion is automatic for most numeric operations. Types are checked on operations to homogenous lists.

**Evaluation Order** While q is entered left-to-right, expressions are evaluated right-to-left or, as the q gods prefer, left of right – meaning that a function is applied to the argument on its right. There is no operator precedence and function application can be written without brackets. Punctuation noise is significantly reduced.

**Null and Infinity Values** In classical SQL, the value `NULL` represents missing data for a field of any type and takes no storage space. In q, null values are typed and take the same space as non-nulls. Numeric types also have infinity values. Infinite and null values can participate in arithmetic and other operations with (mostly) predictable results.

**Integrated I/O** I/O is done through function handles that act as windows to the outside world. Once such a handle is initialized, passing a value to the handle is a write.

**Table Oriented** Give up objects, ye who enter here. In contrast to traditional languages, you'll find no classes, objects, inheritance and virtual methods in q. Instead, q has tables as first class entities. The lack of objects is not as severe as might first appear. Objects are essentially glorified records (i.e., entities with named fields), which are modeled by q dictionaries. A table can be viewed as a list of record dictionaries.

**Ordered Lists** Because classical SQL is the algebra of sets – which are unordered with no duplicates – row order and column order are not defined, making time series processing cumbersome and slow. In q, data structures are based on ordered lists, so time series maintain the order in which they are created. Moreover, simple lists occupy contiguous storage, so processing big data is fast. Very fast.

**Column Oriented** SQL tables are organized as rows distributed across storage and operations apply to fields within a row. Q tables are column lists in contiguous storage and operations apply on entire columns.

**In-Memory Database** One can think of kdb+ as an in-memory database with persistent backing. Since data manipulation is performed with q, there is no separate stored procedure language. In fact, kdb+ comprises serialized q column lists written to the file system and then mapped into memory.

## 0.2 Mathematics Refresher¶

In order to understand q, it is important to have a clear grasp of the basic concepts and terminology of mathematical functions. In fact, the basic constructs of q can all be interpreted as function mappings. The following refresher is intended to help those who are rusty with mathematics.

### 0.2.1 Variables¶

In mathematics, a variable represents an unknown whose potential values can be drawn from a specified domain. The power of the concept is that a property or formula that can be demonstrated to hold for a variable is then known to hold for every particular value of its domain.

A variable has a symbolic representation – i.e., a name. The names of variables occur in syntactically well-formed expressions. In such a context, a variable is given meaning by substitution – i.e., all occurrences of the variable are replaced by a specific value. Once substitution is performed, the variable no longer is an unknown; it has the substituted value from that point onward. In the colloquial, a variable is substituted once in its lifetime.

Imperative programming languages, including q, abuse the concept of a variable. They use the term "variable" to mean a name associated with storage that can hold a value of a specific domain. This should be called an "assignable". The act of associating a name and a value, called _assignment_ , is achieved by placing the value into the storage referenced by the name. Assignment is **not** substitution: the assignment can subsequently be changed and the variable will then have **that** value. Shared assignable variables are the source of great evil in concurrent and distributed computing.

### 0.2.2 Functions¶

In mathematics, a function associates a unique output value to each input value. The collection of all input values is the _domain_ of the function and the collection from which the output values are chosen is the _codomain_ (or _range_). A function is also called a _map_ (or _mapping_) from its domain to its codomain.

The output value that a function f associates to an input value x is read "f of x" or "f at x" or "f on x" More verbosely, we apply (or evaluate) f at an argument to obtain the corresponding result. In mathematics and many programming languages, the output value of a function is represented with the function name to the left of its argument(s), which are enclosed in matched parentheses or brackets. Some functions can take multiple inputs, which are separated by commas or semicolons.

There are two basic ways to define a function: an algorithm or a graph. An algorithm is a sequence of operations on the input value(s) to arrive at the corresponding output value. For example, we define the squaring function, over the domain and codomain of real numbers, to assign as output value the input value times itself. Alternatively, you can define a function by explicitly listing its input-output associations. The collection of associated input-output pairs is the graph of the function.

As you will no doubt recall from many bucolic hours in high-school math class, a function defined by algorithm can be converted to a graph by feeding in input values, cranking out the associated outputs, and collecting the pairs into a table. In general, there is no explicit formula to calculate the values for an arbitrary input-output graph. If it is possible to define a function via a formula, this is usually the preferred way to specify it since it is compact, but there is no guarantee that the formula will be easy or quick to compute.

Here are the two forms for the squaring function over the domain of natural numbers 0 through 3, as you might recall them from school.

_f(x) = x 2_
    
    
```q
I   O
------
0   0
1   1
2   4
3   9
```
When graphing a function, we normally think of the I/O table as a list of _(x,y)_ pairs in the Cartesian plane.
    
    
```q
0, 0
1, 1
2, 4
```
Alternately, we can view this as a pair of columns having positional correspondence.
    
    
```q
0 |-> 0
1 |–> 1
2 |–> 4
```
This perspective will prove useful.

The number of inputs to a function is called its _valence_. Lower valences have their own terminology. A function of valence 1 (i.e., defined by a formula that has one parameter) is said to be _unary_. For example, `neg(x)` takes a number and returns the result of reversing the sign. A function of valence 2 (i.e., two parameters) is said to be _binary_. For example, `sum(x, y)` takes two numbers and adds them to get the result. A function with no parameters is _Nullary_. For example, a constant function with no parameters that returns 42.

Given functions _f_ and _g_ for which the codomain of _g_ is the domain of _f_ , the _composite_ , denoted _f·g_ , is the function obtained by chaining the output of _g_ into _f_. That is, the composite assigns to an input _x_ the output value _f(g(x))_. Observe the usual mathematical convention in which the order of the functions in the composite matches their order in the nested evaluation.

Pictorially, we can see that the composite chains the output of _g_ into the input of _f_ ,
    
    
```q
  g        f
x |–> g(x) |–> f(g(x))
```
The domain of the composite is the domain of _g_ and its codomain is the codomain of _f_.

Convention

In the body of this document, we shall use the term _map_ , or _mapping_ , as shorthand for "mathematical function" and will mean a q function when we write "function" without a modifier.

### 0.2.3 Natural Numbers¶

The natural numbers are the basis of computing. It is instructive to derive the natural numbers from the primitive notion of counting.

  * Start with no items.
  * For any number of counted items, we can count one more, called the _successor_ of that number



We all did this on our fingers as children.

Mathematicians prefer symbols.

  * Start with no items, written 0.
  * For any number of counted items _n_ we can count one more, called the _successor_ of the number and written _s(n)_



Thus the natural numbers are built by iteration of counting "one more."
    
    
```q
0 s(n) s(s(n)) s(s(s(n))) …
```
Writing all those successors can be cumbersome so we give them symbolic names.

0 1 2 3 …

How do we guarantee that these are the **only** things in our collection? Without such assurance, a forger could add spurious items such as infinity or -1 and pass off the forged collection as the natural numbers.

### 0.2.4 Mathematical Induction and Recursion¶

The _principle of mathematical induction_ says that the natural numbers comprise the minimal collection satisfying the following rules.

  * The collection contains 0
  * Whenever _n_ is in the collection so is _s(n)_



For those who care, the formal expression of minimality is that given **any** collection satisfying these two properties, there is a unique map from our collection to that collection. The minimality requirement provides the guarantee that any collection satisfying these rules is essentially the same as our naturals – no more, no less.

Perhaps you are more familiar with the following equivalent way of stating the principle of mathematical induction: to prove that a property holds for all natural numbers, show that the collection of numbers for which it holds satisfies the above two rules.

Let’s take stock. As defined above, a natural number has one of two forms:

  * It is 0
  * Or it is s(n) where n is a natural number previously defined.



Thus to define anything on all natural numbers, we must:

  * Define it on 0
  * For any natural number n on which it is defined, define it on s(n)



This is _inductive_ definition. To define a function _f_ on the natural numbers, we follow this prescription.

  * Define the result of _f_ at 0
  * Assuming _f_ is defined on a natural number n, define it on s(n)



An example of such a function is right in front of us: the successor s.

  * s on 0 is s(0)
  * s on n is s(n)



Now we look at a special case of this general inductive style of function definition with an added requirement (in bold).

  * Define the value of _f_ at 0
  * Assuming _f_ is defined on a natural number n, define it on s(n) **in terms of its value on n**.



A function defined in this manner is _recursive_. At first glance it seems that recursive definition might seriously limit the functions that we can define, but that is not the case.

### 0.2.5 Addition¶

What is addition? The key insight is that adding is merely counting "one mores" – i.e., counting applications of the successor _s_. Expressed inductively,

  * Adding 0 applies _s_ no times – i.e., returns the original number
  * Provided we know how to add _n_ , adding _s(n)_ is one more application _of s_



Let’s make addition so described into a recursive function. For an arbitrary natural number _m,_ we define a recursive function _add_to_m_ that counts applications of the successor to _m_.

  * _add_to_m_ on 0 is _m_ – i.e., apply _s_ no times
  * _add_to_m_ on _s_(_n_) is _s_(_add_to_m_(_n_)) – i.e., apply _s_ once more



Once you experience the Zen, you will see that we can **define** the customary notation for addition using _add_to_m_ _._ **
    
    
```q
   def
m+n = add_to_m(n)
```
It is an instructive exercise to prove the usual properties of addition. For example, show commutativity – which is not immediately obvious.
    
    
```q
    ?
n+m = m+n
```
The proof uses mathematical induction (of course).

### 0.2.6 Recursion Restated¶

As a special case of addition, we observe that _n_ \+ 1 is _s_(_n_). Now we can restate the notion of recursive function definition in the more familiar form.

  * Define _f_ for the base case 0
  * Express _f_(_n_ +1) in terms of _f_(_n_)



It is an interesting exercise left to the reader to prove that this is equivalent to the seemingly more powerful form:

  * Define _f_ for the base case 0
  * Express _f_(_n_ +1) in terms of _f_(0), …, _f_(_n_)



This is the form of recursive definition we shall assume in following chapters. We note that the second portion of this prescription – i.e., the inductive step – gives recursive functions their identifying characteristic. For a non-base input value, the function output involves applications of itself on "lesser" values. Consequently, evaluation involves unwrapping a nested series of such applications until the base case is reached.

### 0.2.7 Multiplication¶

Having defined addition recursively as counting how many times to apply the successor, we can do the same thing by counting how many times to apply addition. This is multiplication. In words,

  * Multiplying by 0 performs no additions
  * Provided we know how to multiply by _n_ , multiplying by _s(n)_ is one more addition of _n_



We translate this into a recursive function _times_m_.

  * _times_m_ on 0 is the identity for addition-i.e., 0
  * _times_m_ on _n_ +1 is _n+times_m(n)_



Incidentally, the fact that multiplication counts how many times to add is why we call multiplication "times" in English and is how the "times" tables we all memorized in elementary school are derived. Again we **define** the customary notation for multiplication using _times_m_.
    
    
```q
    def
m*n = times_m(n)
```
### 0.2.8 Exponentiation¶

Having defined multiplication recursively as counting how many times to apply addition, we repeat the same technique to define exponentiation. Namely, exponentiation counts how many times to apply multiplication. In words,

  * Raising to the power 0 multiplies no times
  * If we know how to raise to the power _n_ , then raising to the power _s(n)_ is one more multiplication



We define the recursive function _m_power_of_ as follows.

  * _m_power_of_ on 0 is the identity for multiplication – i.e., 1
  * _m_power_of_ on _n_ +1 is _m*m_power_of(n)_



Again we **define** the customary notation in terms of the recursive _m_power_of._
    
    
```q
   def
m^n = m_power_of(n)
```
In summary, we have defined basic arithmetic using only recursion, 0 and "one more." The economy of induction is impressive.

### 0.2.9 Lists¶

Lists can also be defined inductively, starting with the empty list and the notion of adding one item to the list.

  * The empty list, written (), is the list with no items
  * Given a list L of items of some domain together with a value x of that domain, we obtain a new list by appending the value x to the end of the list, written L,x



The list constructed by successively appending x1, x2, …, xn to the empty list is written

(x1; x2; …; xn)

Since lists are inductively defined, operations on lists can be defined recursively. For example, the _count_ (i.e., length) of a list is defined as follows.

  * The count of the empty list () is 0
  * Given a list L of count n and an atom x, the count of L,x is 1+count L, which is 1+n



As an exercise you can extend the notion of appending a single value to appending one list to another and then prove – inductively of course – that for two lists L and M,

count(L,M) = count(L) + count(M)

### 0.2.10 Rationals and Reals¶

In everyday usage of mathematics, we implicitly identify things that aren't actually the same. Most common is identifying a natural number – e.g. 42 – as a rational number or as a real number. But a natural number isn’t actually a real number; it just plays one on TV.

Indeed, we have seen that natural numbers are obtained by counting "one more" starting from 0. Ignoring signs for the moment, a rational number is an equivalence class of formal quotients _a_ /_b_ of naturals, In plain speak, _a_ /_b_ and _c_ /_d_ are considered the same just when _ad_ = _bc_. So 2/1 and 4/2 and 6/3, etc. are all considered to be the same. The naturals have a faithful embedding into the rationals in which _n_ is mapped to _n_ /1. This allows us to identify _n_ with (the class of rationals equivalent to) _n_ /1, but they are clearly different mathematical entities.

Similarly, a real number is (in the formulation most of us use) an infinite decimal – i.e., an infinite sequence of natural numbers representing place values in powers of 10. Long division converts a rational to its repeating decimal expansion, so there is an embedding of rationals into the reals, but _a_ /_b_ is manifestly **not** an infinite decimal.

The identification of naturals and rationals as reals is normally done without thinking. In fact, some argue vehemently that a natural number **is** a real number. It is not but it can be identified with one.

Another interesting fact of programming life is that normal data types can only represent an insignificant set of real numbers. Typically reals are represented by single or double precision floating point values. Since these are limited to at most 16 decimal digits, they are clearly rationals, and a pretty limited set of rationals at that. Even "unlimited" decimal values are still rational, as any computer has finite storage. The way to represent a real properly is with a lazy sequence that presents digits "on demand."

The moral of the story is that we can normally go about our programming day without worrying about these distinctions. But there are situations where they matter.

Back to top 
