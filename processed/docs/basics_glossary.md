# Glossary of Terms - kdb+ and q Documentation

## Overview

This glossary provides comprehensive definitions of technical terminology used in the kdb+ and q programming language documentation. It covers fundamental concepts ranging from basic data structures to advanced function characteristics.

## Key Definitions

**Aggregate Function**: A function that reduces its argument (typically a list) to an atom, such as `sum`.

**Atom**: "A single instance of a datatype, eg `42`, `"a"`, `1b`, `2012.09.15`." The type of an atom is always negative.

**Atomic Function**: "An atomic function is a uniform function such that for `r:f[x]` `r[i]~f x[i]` is true for all `i`."

**Dictionary**: "A dictionary is a mapping from a list of keys to a list of values." Keys should be unique, and values can be any data structure.

**Function**: "A mapping from input/s to result defined by an algorithm." Operators, keywords, compositions, projections, and lambdas are all functions.

**Lambda**: "Functions are defined in the lambda notation: an optional signature followed by a list of expressions, separated by semicolons, and all embraced by curly braces."

**List**: "An array, its items indexed by position."

**Matrix**: "A list in which all items are lists of the same count."

**Null**: "Null is the value of an unspecified item in a list formed with parentheses and semicolons." Its value is `` :: ``.

**Symbol**: "A symbol is an atom which holds a string of characters." Denoted with backtick prefix, like `` `abc ``.

**Table**: "A simple table is a list of named lists of equal count" or equivalently a list of dictionaries with identical keys.

**Vector**: "A uniform list of basic types that has a special shorthand notation."

## Function Characteristics

**Rank**: The number of arguments a function takes (nullary, unary, binary, ternary, quaternary).

**Domain**: "The domain of a function is all the possible values of its argument."

**Range**: "The range of a function is the complete set of all its possible results."

**Uniform Function**: A function where `count[x]~count f x` (e.g., `deltas`).

**Atomic Function**: Where application to each item equals applying to the whole structure.

## Data Organization

**Splayed Table**: A table stored with "its columns as separate files" to limit file sizes and speed searches.

**Keyed Table**: "A table of which one or more columns have been defined as its key."

**Enumeration**: "A representation of a list as indexes of the items in its nub or another list."

**Namespace**: "A container or context within which a name resolves to a unique value," designated by dot prefix.

## Operational Concepts

**Projection**: "A function passed fewer arguments than its rank projects those arguments and returns a projection."

**Iterator**: "An iterator is a higher-order operator. It takes a value as its argument and returns a derived function."

**Pass by Reference**: Passing "the name of an object (as a symbol atom) as an argument."

**Pass by Value**: Passing "an object (not its name) as an argument."

**View**: "A view is a calculation that is re-evaluated only if the values of the underlying dependencies have changed."

## Application & Evaluation

The glossary distinguishes between **infix** (operators between arguments like `2+3`), **prefix** (function to the left like `+ 2 3`), and **postfix** notation (iterators to the right like `+/`).

**Conformability** requires that lists, dictionaries, and tables either be atoms or have matching counts for operations.

**Depth** measures nesting levels: atoms have depth 0, simple lists depth 1, nested lists depth 2, etc.
