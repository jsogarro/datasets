# if Control Construct

## Syntax

```q
if[test;e1;e2;e3;…;en]
```

## Description

The `if` control construct evaluates a series of expressions conditionally based on a test condition.

**Parameters:**
- `test`: An expression evaluating to an atom of integral type
- `e1`, `e2`, … `en`: Expressions to evaluate

**Behavior:**
Unless `test` evaluates to zero, expressions `e1` through `en` are evaluated in order. The construct always returns the generic null value.

## Example

```q
q)a:100
q)r:""
q)if[a>10;a:20;r:"true"]
q)a
20
q)r
"true"
```

## Important Notes

`if` is not a function but a control construct—it cannot be iterated or projected. It is typically preferred over `Cond` when a test guards side effects like amending globals.

Common use cases include validating function arguments:

```q
foo:{[x;y]
  if[type[x]<0; :x];            / no-op for atom x
  if[count[y]<>3; '"length"];   / invalid y
  ..
  }
```

## Name Scope

"The brackets of the expression list do not create lexical scope. Name scope within the brackets is the same as outside them." Setting local variables via `if` can produce unintended consequences.

## Related Constructs

- `Cond`, `do`, `while`, Vector Conditional
- See also: Controlling evaluation
