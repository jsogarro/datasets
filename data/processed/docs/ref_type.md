# type – datatype of an object

## Syntax

```q
type x
type[x]
```

## Description

Returns the type of any object `x`. The type is represented as a short integer.

## Return Values

The function returns a short int indicating the object's type:

- **Zero** for a general list
- **Negative** for atoms of basic datatypes
- **Positive** for everything else

## Examples

```q
q)type 5                        / integer atom
-7h
q)type 2 3 5                    / integer vector
7h
q)type (2 3 5;"hello")          / general list
0h
q)type ()                       / general list
0h
q)type each (2;3 5;"hello")     / int atom; int vector; string
-7 7 10h
q)type (+)                      / function
102h
q)type (0|+)                    / composition
105h
```

## See Also

- [`key`](../key/#type-of-a-vector) – type of a vector
- [`.Q.ty`](../dotq/#ty-type) – type utility
- [Casting and encoding](../../basics/by-topic/#casting-and-encoding)
- [Datatypes](../../basics/datatypes/)
