# Controlling Evaluation in kdb+ and q

## Overview

Evaluation in q is controlled through several mechanisms: iterators for repetitive operations, conditional structures for branching logic, explicit returns from functions, error handling, control words, and process termination.

## Iterators

Iterators are "the primary means of iterating in q."

### Maps
Maps (Each, Each Left, Each Right, Each Parallel, and Each Prior) apply values across items of lists and dictionaries.

### Accumulators
Scan and Over iterators apply values progressively—first to arguments, then to each evaluation's result. For unary values, they support three forms: Converge, Do, and While.

### Case Mapping
Rather than switch statements, q handles case logic through indexing:

```q
q)show v:10?`v1`v2`v3
`v1`v1`v3`v2`v3`v2`v3`v3`v2`v1
q)`r1`r2`r3 `v1`v2`v3?v
`r1`r1`r3`r2`r3`r2`r3`r3`r2`r1
q)(`v1`v2!`r1`r2) v
`r1`r1``r2``r2```r2`r1
q)`r1`r2`default `v1`v2?v
`r1`r1`default`r2`default`r2`default`default`r2`r1
```

Functions can be mapped as values:

```q
q)((`abc,;string;::) `v1`v2?v)@'v
`abc`v1
`abc`v1
`v3
"v2"
`v3
"v2"
`v3
`v3
"v2"
`abc`v1
```

## Control Structures

### Conditional Evaluation

The Cond operator `$[test;et;ef;…]` evaluates `ef` when `test` is zero; otherwise `et`. Extended forms implement if/then/elseif chains.

### Control Words

Three control words exist:
- **`do`**: Execute expressions a specified number of times
- **`if`**: Execute expressions when a condition is true
- **`while`**: Execute expressions while a condition holds

These return generic null and are "little used in practice for iteration" compared to iterators.

Common pitfalls include:
```q
a:if[1b;42]43               / use Cond instead
a:0b;if[a;0N!42]a:1b        / unintended sequence
```

## Explicit Return

The notation `:x` terminates a lambda and returns value `x`.

## Error Handling

Signal exits the current lambda and signals an error. Trap and Trap At catch errors during evaluation.

## Process Termination

The `exit` keyword terminates kdb+ with a specified return code.
