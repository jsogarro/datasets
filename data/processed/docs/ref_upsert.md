# upsert

## Syntax

```
x upsert y    upsert[x;y]
```

## Parameters

- `x`: a table, table name as symbol atom, or splayed table directory handle
- `y`: zero or more records (as lists or table)

Records may be provided as:
- Lists matching the column types of `x`
- A table with columns that are members of `cols x` with corresponding types

## Behavior

"If `x` is the name of a table, it is updated in place. Otherwise the updated table is returned."

"If `x` is the name of a table as a symbol atom (or the name of a splayed table as a directory handle) that does not exist in the file system, it is written to file."

## Simple Table

For simple tables, new records are appended:

```q
q)t:([]name:`tom`dick`harry;age:28 29 30;sex:`M)

q)t upsert (`dick;49;`M)
name  age sex
-------------
tom   28  M
dick  29  M
harry 30  M
dick  49  M

q)t upsert((`dick;49;`M);(`jane;23;`F))
name  age sex
-------------
tom   28  M
dick  29  M
harry 30  M
dick  49  M
jane  23  F

q)`t upsert ([]age:49 23;name:`dick`jane)
`t
q)t
name  age sex
-------------
tom   28  M
dick  29  M
harry 30  M
dick  49
jane  23
```

## Keyed Table

"If the table is keyed, any new records that match on key are updated. Otherwise, new records are inserted."

```q
q)show a:([]s:`q`w`e;r:1 2 3;u:5 6 7)
s| r u
-| ---
q| 1 5
w| 2 6
e| 3 7

q)a upsert (`e;30;70)
s| r  u
-| -----
q| 1  5
w| 2  6
e| 30 70

q)a upsert ((`e;30;70);(`r;40;80))
s| r  u
-| -----
q| 1  5
w| 2  6
e| 30 70
r| 40 80

q)a upsert ([s:`e`r`q]r:30 4 10;u:70 8 50)
s| r  u                                       
-| -----
q| 10 50
w| 2  6
e| 30 70
r| 4  8

q)`a upsert ([s:`e`r`q]r:30 4 10;u:70 8 50)
`a
```

## Serialized Table

```q
q)`:data/tser set ([] c1:`a`b; c2:1.1 2.2)
`:data/tser
q)`:data/tser upsert (`c; 3.3)
`:data/tser

q)get `:data/tser
c1 c2
------
a  1.1
b  2.2
c  3.3
```

"Upserting to a serialized table reads the table into memory, updates it, and writes it back to file."

## Splayed Table

```q
q)`:data/tsplay/ set ([] c1:`sym?`a`b; c2:1.1 2.2)
`:data/tsplay/
q)`:data/tsplay upsert (`sym?`c; 3.3)
`:data/tsplay
q)select from `:data/tsplay
c1 c2
------
a  1.1
b  2.2
c  3.3
```

"Upserting to a splayed table appends new values to the column files."

"Upserting to a serialized or splayed table removes any attributes set."

## Related

- [`insert`](../insert/)
- [Join](../join/)
- [Joins](../../basics/joins/)
- [qSQL](../../basics/qsql/)
- [Tables](../../kb/faq/)
