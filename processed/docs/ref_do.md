# do Control Word Reference

## Syntax

```q
do[count;e1;e2;e3;…;en]
```

## Parameters

- **count**: A non-negative integer specifying how many times to execute the expressions
- **e1, e2, … en**: Expressions evaluated sequentially, repeated `count` times

## Description

The `do` construct is a control structure that evaluates a series of expressions repeatedly. According to the documentation, "the expressions `e1` to `en` are evaluated, in order, `count` times." The result is always the generic null value.

## Example

Computing a continued fraction for π over 7 iterations:

```q
q)r:()
q)t:2*asin 1
q)do[7;r,:q:floor t;t:reciprocal t-q]
q)r
3 7 15 1 292 1 1
```

## Important Notes

- `do` is a **control construct, not a function**, so it cannot be iterated or projected
- Brackets in the expression list do not create lexical scope; name scope remains identical to the outer context
- The construct always returns generic null, regardless of expressions evaluated

## See Also

- [Accumulators – Do](../accumulators/#do)
- [`if`](../if/)
- [`while`](../while/)
- [Controlling evaluation](../../basics/control/)
- _Q for Mortals_ §10.1.6 `do`
