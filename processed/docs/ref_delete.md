# `delete`¶

_Delete rows or columns from a table, entries from a dictionary, or objects from a namespace_
    
    
```q
delete    from x
delete    from x where pw
delete ps from x
```
`delete` is a [qSQL query template](../../basics/qsql/) and varies from regular q syntax

For the Delete operator `!`, see

[Functional SQL](../../basics/funsql/#delete)

## Table rows¶
    
    
```q
delete    from x
delete    from x where pw
```
Where

  * `x` is a table
  * `pw` is a condition



deletes from `x` rows matching `pw`, or all rows if `where pw` not specified.
    
    
```q
q)show table: ([] a: `a`b`c; n: 1 2 3)
a n
---
a 1
b 2
c 3
q)show delete from table where a = `c
a n
---
a 1
b 2
```
Attributes may or may not be dropped: reapply or remove as needed

## Table columns¶
    
    
```q
delete    from x
delete ps from x
```
Where

  * `x` is a table
  * `ps` a list of column names



deletes from `x` columns `ps` or all columns if `ps` not specified.
    
    
```q
q)show delete n from table
a
-
a
b
c
```
## Dictionary entries¶
    
    
```q
delete    from x
delete ps from x
```
Where

  * `x` is a dictionary
  * `ps` a list of keys to it



deletes from `x` entries for `ps`.
    
    
```q
q)show d:`a`b`c!til 3
a| 0
b| 1
c| 2
q)delete b from `d
`d
q)d
a| 0
c| 2
```
Cond is not supported inside q-SQL expressions

Enclose in a lambda or use [Vector Conditional](../vector-conditional/) instead.

[qSQL](../../basics/qsql/#cond)

## Namespace objects¶
    
    
```q
delete    from x
delete ps from x
```
Where

  * `x` is a namespace
  * `ps` a symbol atom or vector of name/s defined in it



deletes the named objects from the namespace.
    
    
```q
q)a:1
q)\v
,`a
q)delete a from `.
`.
q)\v
`symbol$()
```
[qSQL](../../basics/qsql/)

Back to top 
