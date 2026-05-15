# while Control Construct

## Syntax

```q
while[test;e1;e2;e3;…;en]
```

## Description

The `while` construct repeatedly evaluates expressions as long as a condition remains true.

**Parameters:**
- `test`: An expression that evaluates to an integral atom
- `e1`, `e2`, … `en`: Expressions to evaluate in sequence

**Behavior:** Unless `test` evaluates to zero, expressions `e1` through `en` execute in order. This cycle—evaluate test, then the expressions—repeats until `test` becomes zero. The result of `while` is always the generic null.

## Example

```q
q)r:1 1
q)x:10
q)while[x-:1;r,:sum -2#r]
q)r
1 1 2 3 5 8 13 21 34 55 89
```

This generates a Fibonacci sequence by repeatedly summing the last two elements while decrementing a counter.

## Important Notes

- `while` is a control construct, not a function; it cannot be iterated or projected
- The bracket notation does not create lexical scope—name scope within brackets matches the surrounding context

## See Also

- [`do`](../do/)
- [`if`](../if/)
- [Accumulators – While](../accumulators/#while)
- [Controlling evaluation](../../basics/control/)
