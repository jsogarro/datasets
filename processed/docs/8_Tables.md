# 8\. Tables¶

## 8.0 Overview¶

Tables are first-class entities in q, meaning they are data structures that live in memory just like lists or dictionaries. A q table is essentially a collection of named columns implemented as a dictionary. Consequently, q tables are column-oriented, in contrast to the row-oriented tables in relational databases. Moreover, since lists are ordered, so are columns, in contrast to SQL where the order of rows is undefined. The fact that q tables comprise ordered column lists makes kdb+ very efficient at storing, retrieving and manipulating sequential data. One important example is time-series data.

Kdb+ handles relational and time series data in the unified environment of q tables. There is no separate data-definition language, no separate stored-procedure language and no need to map internal representations to a separate form for persistence. Just q tables, expressions and functions.

Tables are built from dictionaries, so it behooves the cursory reader to review [Chapter 5](../5_Dictionaries/) before proceeding.

## 8.1 Table Definition¶

### 8.1.1 Review of Table as Column Dictionary¶

We summarize our findings on column dictionaries from [§5.4](../5_Dictionaries/#54-flipping-a-column-dictionary). We began with a rectangular collection of named column lists.
    
    
```q
q)dc:`name`iq!(`Dent`Beeblebrox`Prefect;98 42 126)
q)dc[`iq;]
98 42 126
q)dc[;2]
name| `Prefect
iq  | 126
q)dc[`iq; 2]
126
```
Transpose it with [`flip`](https://code.kx.com/q/ref/flip/) to get a table.
    
    
```q
q)t:flip `name`iq!(`Dent`Beeblebrox`Prefect;98 42 126)
q)t[;`iq]
98 42 126
q)t[2;]
name| `Prefect
iq  | 126
q)t[2;`iq]
126
```
We repeat our findings about a table defined this way.

  * It is a two-dimensional data structure that uses an integer index in the first slot and a symbol column name in the second slot.
  * Specifying only an integer in the first slot retrieves a section dictionary across that index – i.e., a record.
  * Specifying only a column name in the second slot retrieves that column.
  * Specifying both an integer and a column name retrieves the "field" at that row in that column.
  * A table is logically a list of section dictionaries.
  * The **only** effect of flipping the column dictionary is to reverse the order of its indices; no data is rearranged under the covers.



All tables have type `98h`.
    
    
```q
q)type t
98h
```
The proper way to extract a table column is by eliding the row index. For table columns (only), you can omit the leading semi-colon in the syntax.
    
    
```q
q)t[;`iq]
_
q)t[`iq]
_
q)t[`name`iq]
Dent Beeblebrox Prefect
98   42         126
```
Since it is possible to retrieve values from a dictionary using dot notation, this is also true for tables.
    
    
```q
q)t.name
_
```
Unfortunately dot notation doesn’t work inside functions so we recommend not using it at all.

### 8.1.2 Table Display¶

Observe that the row and column display of a table is indeed the transpose of the dictionary display. This reflects the transposed indices, even though the internal data layout is the same.
    
    
```q
q)dc
name| Dent Beeblebrox Prefect
iq  | 98   42         126
q)t
name       iq 
--------------
Dent       98 
Beeblebrox 42 
Prefect    126
```
### 8.1.3 Table-Definition Syntax¶

Constructing a table by flipping a column dictionary is useful when you need to build the table on the fly. There is an alternate syntax to define tables that makes things a bit more readable.
    
    
```q
([] *c1*:*L1*; ...; *cn*:*Ln*)
```
Here _c i_ is a symbol representing a column name and _L i_ is the corresponding list of column values. The _L i_ are lists of equal count, but can be atoms as long as at least one is a list. The brackets will contain key columns for keyed tables – explained in §8.3.3 – but are empty for tables. For readability, in this tutorial we shall often include optional whitespace after the closing square bracket and to the right of semicolon separators.

The colons in table-definition syntax are **not** assignment. They are part of the syntactic sugar and serve as markers separating column names from column values.

Do not omit the square brackets in table-definition syntax

The interpreter will not complain and you will end up with a list, rather than a table, and the colons will be assignment.

Here is how to define our favorite table using table-definition syntax, which is arguably simpler.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)t~flip `name`iq!(`Dent`Beeblebrox`Prefect;98 42 126)
1b
```
The value lists in table-definition syntax can originate from variables, which is useful for programmatic table definition. In this case the column names are the variable names.
    
    
```q
q)c1:`Dent`Beeblebrox`Prefect
q)c2:98 42 126
q)([] c1; c2)
_
```
This example shows that if you don't explicitly specify column names, q will create them on a best-effort basis. In this example, it used the variable names. It will also try to make duplicate column names unique by appending a numeric suffix.

Any valid q expression can appear to the right of the semi-colon in table-definition syntax; it is evaluated as part of the table construction. You must ensure that the resulting column lists all have the same length.
    
    
```q
q)([] c1:1+til 5; c2:5#42)
_
```
Provided you specify at least one column as a list, atoms will be extended to match.
    
    
```q
q)([] c1:`a`b`c; c2:42; c3:98.6)
c1 c2 c3  
----------
a  42 98.6
b  42 98.6
c  42 98.6
```
Using an atom in this way might appear to assign a default value. It does not.

You cannot define a single-row table using all atoms. You must enlist at least one of the atoms.
    
    
```q
q)([] c1:`a; c2:100)
'rank
q)([] enlist `a; c2:100)
_
```
If you create a table as the flip of a column dictionary, item-wise extension is performed when the column dictionary is flipped into a table.
    
    
```q
q)`c1`c2`c3!(`a`b`c;42;1.1)
c1| `a`b`c
c2| 42
c3| 1.1
q)flip `c1`c2`c3!(`a`b`c;42;1.1)
c1 c2 c3 
---------
a  42 1.1
b  42 1.1
c  42 1.1
```
### 8.1.4 Table Metadata¶

The column names of a table can be retrieved as a list of symbols with [`cols`](https://code.kx.com/q/ref/cols).
    
    
```q
q)cols t
_
```
The function [`meta`](https://code.kx.com/q/ref/meta/) applied to a table retrieves its metadata. The result is a keyed table (see §8.4.1) with one record for each column in the original table.

  * The key column `c` of the result contains the column names.
  * The column `t` contains a symbol denoting the type char of the column.
  * The column `f` contains the domains of any foreign key or link columns.
  * The column `a` contains any attributes associated with the column.


    
    
```q
q)meta t
c   | t f a
----| -----
name| s    
iq  | j
```
When `meta` displays an **upper** -case type char for a column, this indicates that column is a _compound_ list in which all fields are simple lists of the indicated type. Such tables arise, for example, when you group without aggregating in a query. Here is one created manually. Observe the upper case J in the `t` column for column `c2`.
    
    
```q
q)meta ([] c1:1 2 3; c2:(1 2; enlist 3; 4 5 6))
c | t f a
--| -----
c1| j    
c2| J   
```
When `meta` is applied to a partitioned table that has been mapped into memory, it examines only the most recent partition.

Thus it makes the implicit assumption that the partition slice schemas are consistent across the partitions. In the event that the partition slices are not consistent, `meta` blithely reports the schema found in the most recent partition. 

In order to determine the offending partition, you will have to run `meta` on each of the partition slices and determine the outliers. This is relevant because the `.Q` utilities will **not** prevent you from writing an inconsistent partition slice and you may discover the error later.

The function [`tables`](https://code.kx.com/q/ref/tables/) takes a symbolic namespace (see [§12.1](../12_Workspace_Organization/#121-namespaces)) and returns a sorted symbol list of the names of tables in that context. For example, we list all tables in the root context for a fresh q session.
    
    
```q
q)t2:([] c1:1 2 3; c2:(1 2; enlist 3; 4 5 6))
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)tables `.
`t`t2
```
Alternatively, the command [`\a`](https://code.kx.com/q/basics/syscmds/#a-tables) provides the same result. If no argument is provided, it returns the result for the current context.

### 8.1.5 Records¶

Since a table is logically a list of dictionary records, [`count`](https://code.kx.com/q/ref/count/) returns the number of records. In our example,
    
    
```q
q)count t
_
```
Now let's inspect the sequence of records.
    
    
```q
q)t
name       iq 
--------------
Dent       98 
Beeblebrox 42 
Prefect    126
q)t[0]
name| `Dent
iq  | 98
q)t[1]
name| `Beeblebrox
iq  | 42
```
Since a record dictionary slices across the table display horizontally, this motivates calling the value portion of the record dictionary a table _row_. The record associates column names with the values in a physical row. To retrieve the naked values of a row – i.e., without the column names – simply apply `value` to the record dictionary.
    
    
```q
q)value t[1]
`Beeblebrox
42
```
### 8.1.6 Flipped Column Dictionary vs. List of Records¶

Is a table a flipped column dictionary or a list of records? Logically it is both, but physically it is stored as a column dictionary. In fact, q dynamically recognizes a conforming list of dictionaries that could be formed into a table and reorganizes the data into columnar form automatically. It doesn’t ask for permission or seek forgiveness.

To verify this, we first create a list of non-conforming dictionaries that differ in  key names. As expected it has type 0.
    
    
```q
q)type (`name`iq!(`Dent;98); `nome`iq!(`Beeblebrox;42))
0h
```
Once we make the names agree so that the records conform, `type` tells us that q has indeed converted the list to a table.
    
    
```q
q)type (`name`iq!(`Dent;98); `name`iq!(`Beeblebrox;42))
98h
```
This is no mere illusion. The data have been reorganized into columns.

A useful special case of this phenomenon

Start with a simple dictionary and enlist it to create a singleton. What is this? 
    
    
```q
q)enlist `a`b!10 20
_
```
It is a table because it is a singleton list of records that conform.

You might think that you don’t want q to do this automatic reorganization, but you would be wrong. In general, column retrieval and manipulation will be significantly faster than operations on a list of dictionaries, especially if the columns are simple – i.e., the column values are stored contiguously.

A downside of tables being stored as columns is that row deletion is an expensive operation because all the column lists must be compressed to close the resulting gap. The best way to deal with this in large tables is not to do it. Instead of deleting a row, use a separate column that holds a flag indicating whether the row has been deleted and then exclude the "deleted" rows. Then compress this column to save space since it will be sparse.

## 8.2 Empty Tables and Schema¶

We saw in the previous section that a table can be defined and populated in one step using table syntax.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
```
Fully listing columns with literals is usually done only with smaller tables – e.g., lookup tables. Large tables are usually created programmatically from computed data or data read from files or received over the wire.

In these circumstances, it is useful to create an empty table initially and then populate it later by appending in place. You could do this with general empty lists.
    
    
```q
q)([] name:(); iq:())
```
All columns here are lists of general type, so data of any type can be appended in the first record. Should the first item being appended to an empty column be an atom (a common case), the column becomes a singleton list with that atom – i.e., a simple list of that type. As with all simple lists, type checking will thereafter be enforced for all inserts and updates to that column. Consequently all subsequent appends must match the initial one.

This situation is fine provided you are guaranteed that all data will match the initial record. Unfortunately, real world data can be highly unpredictable. Should the first record be bad, type checking will reject all subsequent good data.

It is good practice to specify the types of all columns in an empty table.

In the table definition, cast an empty list to the appropriate type.
    
    
```q
q)([] name:`symbol$(); iq:`int$())
_
```
A shorter, and arguably less obvious technique is the following.
    
    
```q
q)([] name:0#`; iq:0#0)
_
q)([] name:0#`; iq:0#0) ~ ([] name:`symbol$(); iq:`long$())
1b
```
Whichever form you use, it ensures that only data of the appropriate type can be appended.

## 8.3 Basic select and update¶

We shall cover q-sql in depth in [Chapter 9](../9_Queries_q-sql/), but we provide an introduction here in order to extract and display data in our examples. We use the following sample table in this section.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
```
### 8.3.1 Syntax of `select`¶

The basic select is a template that takes the form,
    
    
```q
select cols from table
```
where _table_ is either a table or a keyed table and _cols_ is a comma-separated list of columns. This expression results in a list of **all** records for the specified columns. This corresponds to the SQL statement,
    
    
```q
SELECT * FROM table
```
In q you do not write the wildcards when you want all columns in the table; simply omit the columns and you get them all.

This basic select syntax may seem comfortably familiar from SQL, but it should seem odd to the qbie who is just getting accustomed to parsing expressions right-to-left. Neither `select` nor `from` represent functions that can stand alone. There are many options for the basic select template whose elements appear between the `select` and `from` or after the _table_ element.

### 8.3.2 Displaying the Result¶

The result of `select` is always a table. You may think of it as a list of (conforming) records but it is actually constructed as a column dictionary.
    
    
```q
q)select from t
name       iq 
--------------
Dent       98 
Beeblebrox 42 
Prefect    126
```
### 8.3.3 Selecting Columns¶

To select specific columns, list them comma-separated in left-to-right order between `select` and `from`. You can optionally provide column names using the same colon format as table-definition syntax.
    
    
```q
q)select name from t
name
----------
Dent
Beeblebrox
Prefect
q)select c1:name, c2:iq from t
_
```
### 8.3.4 Basic update¶

The syntax of `update` is the same as `select`, but named columns represent replacement by the values to the right of the colon. In our example,
    
    
```q
q)update iq:iq%100 from t
_
```
## 8.4 Primary Keys and Keyed Tables¶

In SQL, one can declare one or more column(s) of a table as a primary key. This means that the values in the column(s) are unique over the domain of the rows, making it possible to identify and retrieve a row via its key value. These two features motivate how q implements a keyed table.

### 8.4.1 Keyed Table¶

We begin with a simple key – i.e., the key is a single column of simple type. The approach is to place the key column in a separate table parallel to a table containing the remaining columns to obtain a table of keys and a table of values. Since a table is logically a list of records, this is the same as a list of key records and list of value records. We establish a positional correspondence between these lists via a dictionary mapping.

A _keyed table_ is a dictionary mapping a table of key records to a table of value records. This represents a mapping from each row in a table of (presumably unique) keys to a corresponding row in a table of values – i.e., a positional correspondence of key rows to value rows. Using dictionary lookup on a key (record) retrieves the corresponding value record in the remaining columns. This is just what a primary key should do. Note that the key mapping assumes that the key records and value records are in corresponding order.

A keyed table is **not** a table – it is a dictionary and so has type `99h`.

Keys should be unique but (sadly) this is not enforced. As we have already noted, dictionary creation does not enforce key uniqueness. A value row associated with a duplicate key is not accessible via key lookup, but it can be retrieved via a `select` on the key column.

### 8.4.2 Simple Example¶

Let’s see how this works for our previous example. We begin with a flipped column dictionary to make things explicit.
    
    
```q
q)v:flip `name`iq!(`Dent`Beeblebrox`Prefect;98 42 126)
```
Now say we want to add a key column `eid` containing employee identifiers. We begin by placing the identifiers in a separate table. Recall from [§5.3.4](../5_Dictionaries/#534-column-dictionary-with-a-single-column) that we must enlist both the column name and the value list for single column table.
    
    
```q
q)k:flip (enlist `eid)!enlist 1001 1002 1003
```
Now establish the association between the two tables.
    
    
```q
q)kt:k!v
```
_Voilà!_ The console display of a keyed table is the combination of dictionary display and table display. It lists the key column(s) on the left, separated by a vertical bar from the value columns on the right.
    
    
```q
q)kt
eid | name       iq 
----| --------------
1001| Dent       98 
1002| Beeblebrox 42 
1003| Prefect    126
```
### 8.4.3 Keyed-Table Definition Syntax¶

Fundamentalists insist on constructing a keyed table as a dictionary of flipped dictionaries, but most folks prefer to use table-definition syntax.

Note

Due to space limitations on the printed page we cannot fit our keyed table examples on a single line, so we wrap the lines, with column alignment for readability. You cannot enter such a multiline statement into the q console, but you can place it in a script. If you are following this tutorial line by line, you can copy/paste the individual lines onto a single line in the q console.

Here is the fundamental form of our keyed table.
    
    
```q
q)kt:(flip (enlist `eid)!enlist 1001 1002 1003)!
    flip `name`iq!(`Dent`Beeblebrox`Prefect;98 42 126)
```
It is arguably simpler to use table-definition syntax. This is a generalization of (plain) table definition in which key column(s) are placed between the square brackets and the value columns are after the square brackets.
    
    
```q
q)kt:([eid:1001 1002 1003]
      name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
```
Placing key column(s) inside the square brackets is consistent with the notation for a (regular) table since a regular table has no keys.

To define an empty keyed table, use empty key and value columns.
    
    
```q
q)ktempty:([eid:()] name:(); iq:())
```
As with regular tables, empty columns should be typed with either of the following constructs to ensure data integrity.
    
    
```q
q)ktempty:([eid:`int$()] `symbol$name:(); iq:`int$())
q)ktempty:([eid:0#0] name:0#`; iq:0#0)
```
### 8.4.4 Accessing Records of a Keyed Table¶

Since a keyed table is a dictionary mapping, it provides access to records in the value table via key lookup. Remember that the records in the key table and value table are section dictionaries.
    
    
```q
q)kt[(enlist `eid)!enlist 1002]
name| `Beeblebrox
iq  | 42
```
Yikes! This is a cumbersome way to retrieve by key.

Fortunately, you can abbreviate the full dictionary specification of a key record to its key value. Our example reduces to,
    
    
```q
q)kt[1002]
name| `Beeblebrox
iq  | 42
```
Now we can look up the value for an individual column.
    
    
```q
q)kt[1002][`iq]
42
```
Or we can use the equivalent index at depth notation.
    
    
```q
q)kt[1002;`iq]
42
```
After the customary moment of q Zen, we realize that the net effect of "placing a key on a table" is to convert indexing of the rows from row number to key value.

### 8.4.5 Retrieving Multiple Records¶

We have seen how to look up a single record in a keyed table by key value.
    
    
```q
q)kt[1001]
name| `Dent
iq  | 98
q)kt 1001
_
```
You might think it is possible to retrieve multiple records from a keyed table via a simple list of keys. You would be wrong.
    
    
```q
q)kt[1001 1002]
'length
```
This works for a compound key – i.e., a multi-column key – just not for a single column key.

To look up multiple key values in a keyed table, we could use one of the following constructs to generate a list of enlisted keys.
    
    
```q
q)kt[(enlist 1001;enlist 1002)]
name       iq
-------------
Dent       98
Beeblebrox 42
q)kt[flip enlist 1001 1002]
_
```
This is still pretty cumbersome, so back to the drawing board. Since we are supposed to provide a list of key records for lookup, we can simply create an anonymous table whose records are precisely the form we need.
    
    
```q
q)kt ([] eid:1001 1002)
_
```
Now that’s slick!

With such a nifty way to create a list of key records in hand, we recall from [§5.2.2](../5_Dictionaries/#522-extracting-a-sub-dictionary) that a sub-dictionary can be extracted using a list of keys as the left operand of `#`. Applying this to our keyed table,
    
    
```q
q)([] eid:1001 1002)#kt
eid | name       iq
----| -------------
1001| Dent       98
1002| Beeblebrox 42
```
The Zen of keyed tables… no select statement required.

### 8.4.6 Reverse Lookup¶

Because a keyed table is a dictionary, it is possible to perform reverse lookup from value records to key records. Let’s show an example having a single value column.
    
    
```q
q)kts:([eid:1001 1002 1003] name:`Dent`Beeblebrox`Prefect)
q)kts
_
```
As in the case of key lookup, we can use an anonymous table with a list of value records.
    
    
```q
q)kts?([] name:`Prefect`Dent)
_
```
### 8.4.7 Components of a Keyed Table¶

Since a keyed table is a dictionary mapping the table of keys to the table of values, the functions [`key`](https://code.kx.com/q/ref/key/) and [`value`](https://code.kx.com/q/ref/value/) extract the constituents.
    
    
```q
q)key kt
_
q)value kt
_
```
The function `keys` returns a list of symbolic key column name(s).
    
    
```q
q)keys kt
_
```
Observe that [`cols`](https://code.kx.com/q/ref/cols/) retrieves all column names of the keyed table – i.e., from both the key and value tables.
    
    
```q
q)cols kt
_
```
### 8.4.8 Tables vs. Keyed Tables¶

It is possible to convert dynamically between a regular table having a column of potential key values and the corresponding keyed table using binary primitive [`xkey`](https://code.kx.com/q/ref/xkey/). The right operand is the source table/keyed table and the left operand is a symbol (or list of symbols) with the column name(s) to be used as the key.
    
    
```q
q)t:([] eid:1001 1002 1003; name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)`eid xkey t
eid | name       iq 
----| --------------
1001| Dent       98 
1002| Beeblebrox 42 
1003| Prefect    126
```
Conversely, to convert a keyed table to a regular table, use `xkey` with an empty general list as the left operand.
    
    
```q
q)kt:([eid:1001 1002 1003] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)kt
_
q)() xkey kt
eid  name       iq 
-------------------
1001 Dent       98 
1002 Beeblebrox 42 
1003 Prefect    126
```
You can also use an overload of `!` to [key/unkey](https://code.kx.com/q/ref/enkey/) tables. The left operand is a non-negative integer that specifies the number of left-most columns to include in the key, where 0 indicates none – i.e., no keys. With t and kt as above,
    
    
```q
q)1!t
_
q)0!kt
_
```
While these forms are terse, the first makes your code less obvious since the new key column(s) are only implicit.

The table/keyed table conversions above (in both forms) work on copies and do not affect the original table. Use call-by-name to modify the original.
    
    
```q
q)`eid xkey `t
`t
q)t
_
q)() xkey `kt
`kt
q)kt
_
```
If `xkey` is applied with a column that does not contain unique values, the result is not a error but rather a keyed table that does not have a true primary key.
    
    
```q
q)t:([] eid:1001 1002 1003 1001; name:`Dent`Beeblebrox`Prefect`Dup)
q)ktdup:`eid xkey t
q)ktdup
eid | name
----| ----------
1001| Dent
1002| Beeblebrox
1003| Prefect
1001| Dup
```
Duplicate key values are not accessible via key lookup.
    
    
```q
q)ktdup 1001
name| Dent
```
They are accessible via select.
    
    
```q
q)select from ktdup where eid=1001
eid | name
----| ----
1001| Dent
1001| Dup
```
### 8.4.9 Compound Primary Key¶

The q implementation of a keyed table as a dictionary mapping between a pair of tables carries over unchanged to compound keys. Recall that a compound key in SQL is a collection of multiple columns that together provide a unique value for each row. A compound key in q is simply a table association `k!v` in which `k` has multiple columns. Presumably each record in the key table has a unique combination of field values, but this is **not** checked.

Here is our galactic travelers table redone to replace the employee ID with a compound key comprising last and first names.
    
    
```q
q)ktc:([lname:`Dent`Beeblebrox`Prefect; fname:`Arthur`Zaphod`Ford]; iq:98 42 126)
```
Observe that the console displays a compound keyed table with the key columns on the left of the vertical bar and the value columns to the right.
    
    
```q
q)ktc
lname      fname | iq 
-----------------| ---
Dent       Arthur| 98 
Beeblebrox Zaphod| 42 
Prefect    Ford  | 126
```
Here is lookup by a compound key record,
    
    
```q
q)ktc[`lname`fname!`Beeblebrox`Zaphod]
_
```
As with a simple key, we can abbreviate a full key record to the key value for key lookup.
    
    
```q
q)ktc[`Dent`Arthur]
_
```
The empty keyed table can be typed using table-definition syntax with either of the following,
    
    
```q
q)ktc:([lname:`symbol$();fname:`symbol$()] iq:`int$())
q)ktc:([lname:0#`;fname:0#`] iq:0#0)
```
For the fundamentalists, here is the same compound keyed table built from its constituent column dictionaries.
    
    
```q
q)ktc:(flip `lname`fname!(`Dent`Beeblebrox`Prefect;`Arthur`Zaphod`Ford))!
    flip (enlist `iq)!enlist 98 42 126
```
Most will agree that the table-definition syntax is simpler.

### 8.4.10 Retrieving Records with a Compound Primary Key¶

Unlike a simple key, we can lookup multiple value records with a list of compound keys.
    
    
```q
q)ktc (`Dent`Arthur;`Prefect`Ford)
_
```
Of course the nifty construct with an anonymous table works with compound keys too.
    
    
```q
q)ktc ([] lname:`Dent`Prefect; fname:`Arthur`Ford)
_
```
As does the use of [`#`](https://code.kx.com/q/ref/take/ "Take") to retrieve a sub keyed table from a list of keys.
    
    
```q
q)K:([] lname:`Dent`Prefect; fname:`Arthur`Ford)
q)K#ktc
lname   fname | iq 
--------------| ---
Dent    Arthur| 98 
Prefect Ford  | 126
```
### 8.4.11 Extracting Column Data¶

In this section we use the following example tables. The first has a simple key of long and the second a compound key of symbols.
    
    
```q
q)kts:([k:101 102 103] v1:`a`b`c; v2:1.1 2.2 3.3)
q)kts
_
q)ktc:([k1:`a`b`c;k2:`x`y`z] v1:`a`b`c; v2:1.1 2.2 3.3)
q)ktc
_
```
In the previous section we saw how to retrieve **all** the value columns using an anonymous table of keys.
    
    
```q
q)kts[([] k:101 103)]
_
q)ktc[([] k1:`a`c;k2:`x`z)]
_
```
Often we need to extract the naked column data from some value columns. No problem for the fundamentalist. Since the result of the lookup is a sub-table of the value table, we can index into it to get the column list(s).
    
    
```q
q)kts[([] k:101 103)][`v1]
`a`c
q)ktc[([] k1:`a`c;k2:`x`z)][`v1`v2]
_
```
And we can simplify using indexing at depth.
    
    
```q
q)kts[([] k:101 103); `v1]
_
q)ktc[([] k1:`a`c;k2:`x`z); `v1`v2]
_
```
## 8.5 Foreign Keys and Virtual Columns¶

A foreign key in SQL is a column in one table whose values are members of a primary key column in another table. Foreign keys are the mechanism for establishing relations between tables.

An important feature of a foreign key is that the RDBMS enforces referential integrity, meaning that values in the foreign key column are **required** to be in the related primary key column. Before you can insert a row having a foreign key field value that is not there, you must first ensure there is a row with that primary key in the related table.

### 8.5.1 Definition of Foreign Key¶

A foreign key in q should provide a relation with referential integrity (in one direction, at least). Can we implement this with a construct we already know in q? The setting is a collection of record items in a column list that are drawn from the (supposedly) unique record items in another column list. Sound familiar? It is precisely the situation of an enumeration!

A _foreign key_ is one or more table columns whose values are defined as an enumeration over the key column(s) of a keyed table. As in the case of symbol enumeration ``sym$`, the enumeration restricts foreign key values to be in the list of primary key values.

### 8.5.2 Example of Simple Foreign Key¶

We return to the galactic travelers keyed table.
    
    
```q
q)kt:([eid:1001 1002 1003] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
```
Suppose we have a table with the detail records of the results of repeated IQ tests of the travelers.
    
    
```q
([] eid:1003 1001 1002 1001 1002 1001; sc:126 36 92 39 98 42)
```
This table has no restriction on `eid` other than it be a long. To ensure that only `eid` values for actual travelers can be entered, we make the `eid` column in this table a foreign key related to the `eid` column in `kt`. This is done by enumerating over the name of the keyed table – i.e., ``kt$`.

When q sees the name of a keyed table in an enumeration domain it knows to use the list of key records.

Here is the enumeration of the column in isolation.
    
    
```q
q)`kt$1002 1001 1001 1003 1002 1003
```
As in the case of symbol enumeration, q looks up the index of each foreign key value in the list of key records and, under the covers, replaces the field value with that index. Also as with symbols, the enumeration is displayed in reconstituted form instead of as the underlying indices. To see the underlying indices, cast to an integer.
    
    
```q
q)`long$`kt$1002 1001 1001 1003 1002 1003
1 0 0 2 1 2
```
As always, the enumeration can be substituted for the original in normal operations.
    
    
```q
q)1003=`kt$1002 1001 1003 1002 1003
000101b
```
And now, the moment of truth. Does the enumeration provide referential integrity?
    
    
```q
q)`kt$1004
'cast
```
It does. Attempting to enumerate a value that is not in the primary key column causes an error.

We put this together with table-definition syntax to define a details table with a foreign key over `kt`.
    
    
```q
q)tdetails:([] eid:`kt$1003 1001 1002 1001 1002 1001; sc:126 36 92 39 98 42)
```
Observe that a foreign key is denoted by the name of the target keyed table in the `f` column in the output of `meta`.
    
    
```q
q)meta tdetails
c  | t f  a
---| ------
eid| j kt  
sc | j    
```
The built-in function [`fkeys`](https://code.kx.com/q/ref/fkeys/) applied to a table (or keyed table) returns a dictionary in which each foreign key column name is mapped to its primary key table name.
    
    
```q
q)fkeys tdetails
eid| kt
```
### 8.5.3 Resolving a Foreign Key¶

When you wish to resolve a foreign key – i.e., get the actual values instead of enumerated values – apply `value` to the enumerated column.
    
    
```q
q)meta update value eid from tdetails
c  | t f a
---| -----
eid| j    
sc | j    
```
Observe that there is no longer an entry in the `f` column.

### 8.5.4 Foreign Keys and Relations¶

In SQL, a join is used to splice back together data that has been normalized via relations. The splice is done along a foreign key that establishes a relation to the primary key. In the result, columns from both tables are available using dot notation.

The same effect is achieved in q using foreign keys but you don't need to perform the join explicitly. The notation is similar, but the operation is different enough to warrant close attention.

Let `tf` be a table having a foreign key column `f` enumerated over a keyed table `kt`. All columns in `kt` are available via dot notation in a `select` expression whose _from_ domain is `tf`. To access a column `c` in `kt`, use the notation `f.c` in the `select` expression. This column takes the name `c` by default in the result.

In our galactic travelers example, we can access columns in `kt` via a `select` on `tdetails`. Here is a query that retrieves a "foreign" column in addition to one from `tdetails`.
    
    
```q
q)select eid.name, sc from tdetails
name       sc 
--------------
Prefect    126
Dent       36 
Beeblebrox 92 
Dent       39 
Beeblebrox 98 
Dent       42 
```
There is an implicit left join between `tdetails` and `kt` here.

Tip

The implicit join with dot notation is powerful and convenient when your tables are in normal form and there are multiple foreign key relations. For example, a query could retrieve 
    
    
```q
select name.street.city.zip.country from residents where …
```
in a single select with no explicit joins.

## 8.6 Working with Tables and Keyed Tables¶

In this section, we use the galactic travelers tables.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)kt:([eid:1001 1002 1003] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
```
### 8.6.1 Appending Records¶

The fundamental way to append a record to a table is to view the table as a list of records and join with `,:`. Note that the fields in the record do not need to be in column order.
    
    
```q
q)t,:`name`iq!(`W; 26)
q)t,:`iq`name!(200; `Albert)
q)t
_
```
You can also append naked row values with `,:` but the fields **must** be in column order.
    
    
```q
q)t,:(`H; 142)
_
q)t,:(97;`J)
'type
q)t
_
```
### 8.6.2 First and Last Records¶

Because a table is logically a list of records, the functions [`first`](https://code.kx.com/q/ref/first/) and [`last`](https://code.kx.com/q/ref/last/#last) retrieve the initial and final records, respectively.
    
    
```q
q)first t
name| `Dent
iq  | 98
q)last t
_
```
A keyed table is a dictionary so functions apply to the value table.
    
    
```q
q)first kt
name| `Dent
iq  | 98
q)last kt
_
```
Because tables and keyed tables are ordered, these functions can be used for aggregation in queries without any need of the SQL clause `ORDER BY` – provided your table was created in order.

You can retrieve the first or last _n_ records of a table or keyed table using the Take operator `#`. Why does this work? Tables are lists and keyed tables are dictionaries and `#` works on both. Since `#` always returns a list and the extracted records conform, q recognizes the result of `#` as a table or keyed table with the same schema as the input.
    
    
```q
q)2#t
_
q)-3#kt
_
```
Also see [§9.3.2.4](../9_Queries_q-sql/#9324-select) for another way to achieve this result using `select[n]`.

### 8.6.3 Find¶

The [Find](https://code.kx.com/q/ref/find/) operator `?` used with a table returns the index of a record – i.e., its row number.
    
    
```q
q)t?`name`iq!(`Dent;98)
0
```
As usual, the record can be abbreviated to a row list provided the fields have the right type and order.
    
    
```q
q)t?(`Dent;98)
_
```
Since Find is atomic in the right operand, you can determine multiple row indices.
    
    
```q
q)t?((`Dent;98);(`Prefect;126))
_
```
Since a keyed table is a dictionary, Find `?` performs a reverse lookup of a value record/row and returns the first associated key record.
    
    
```q
q)kt?`name`iq!(`Dent;98)
eid| 1001
q)kt?(`Dent;98)
_
```
As with key lookup, a single column must have enlisted values, or you can use the anonymous table construct.
    
    
```q
q)t1:([] eid:1001 1002 1003)
q)t1?enlist each 1001 1002
0 1
q)t1?([] eid:1001 1002)
_
```
### 8.6.4 Union with `,`¶

The [Join](https://code.kx.com/q/ref/join/) operator `,` is defined for tables and keyed tables since they both comprise lists of records. It is essentially the same as `UNION` in SQL.

You can use `,` to append a record to (a copy of) a table, but no type checking will be performed.
    
    
```q
q)t,`name`iq!(`Slaartibartfast; `123)
name            iq  
--------------------
Dent            98  
Beeblebrox      42  
Prefect         126 
Slaartibartfast `123
```
Using a row with `,` will not yield a table. Instead you get a general list.
    
    
```q
q)t,(`Slaartibartfast; 110)
`name`iq!(`Dent;98)
`name`iq!(`Beeblebrox;42)
`name`iq!(`Prefect;126)
`Slaartibartfast
110
```
Tables having **exactly** the same [`meta`](https://code.kx.com/q/ref/meta/) result can be joined to form a table. Since a table is a list of records, the result is obtained by appending the records of the right operand to those of the left.
    
    
```q
q)t,([] name:1#`W; iq:1#26)
_
q)t,t
_
```
Two tables with the same columns in different order can not be joined because the order of columns is significant in q.
    
    
```q
q)t,([] iq:1#42; name:`W)
'mismatch
```
Later versions of kdb+ do not signal a mismatch. _[Ed.]_

Two keyed tables with the same `meta` result can be joined with `,`. Because a keyed table is a dictionary whose keys and values are record lists, the operation has upsert semantics. Keys in the right operand that are not in the left operand are treated as append (i.e., insert), whereas the right operand acts as an update on common key values. In other words, the right operand is upserted into the left.
    
    
```q
q)kt,([eid:1003 1004] name:`Prefect`W; iq:150 26)
eid | name       iq 
----| --------------
1001| Dent       98 
1002| Beeblebrox 42 
1003| Prefect    150
1004| W          26 
```
### 8.6.5 Coalesce `^`¶

[Coalesce](https://code.kx.com/q/ref/coalesce/) `^` can be used to merge two keyed tables having the same columns. Its behavior derives from its behavior on dictionaries. For a common key value and a common column, the value of the column in the right keyed table prevails over that of column in the left keyed table, except where the right column is null, in which case the left column value survives. On non-common keys the individual values carry thru.

The behavior of `^` is the same as `,` when there are no nulls in a column in the right table.
    
    
```q
q)([k:`a`b`c] v:10 0N 30)^([k:`a`b`c] v:100 200 0N)
k| v
-| ---
a| 100
b| 200
c| 30
q)([k:`a`b`c`x] v:10 0N 30 40)^([k:`a`b`c`y]; v:100 200 0N 0N)
_
```
The performance of `^` is slower than that of `,` since fields of the right operand must be checked for null.

### 8.6.6 Column Join¶

Two tables with the same number of records can be joined sideways with Join Each (`,'`) to create a _column join_ in which the columns are aligned in parallel
    
    
```q
q)([] c1:`a`b`c),'([] c2:100 200 300)
c1 c2 
------
a  100
b  200
c  300
```
When the column lists of the tables are not disjoint, the operation on the common columns has upsert semantics because each record is a dictionary.
    
    
```q
q)([] c1:`a`b`c; c2:1 2 3),'([] c2:100 200 300)
c1 c2 
------
a  100
b  200
c  300
```
A sideways join on keyed tables requires that the key records conform, meaning that the key columns must have identical meta. The columns from the right operand are aligned along common keys and appended elsewhere.
    
    
```q
q)([k:1 2 3] v1:10 20 30),'([k:3 4 5] v2:1000 2000 3000)
k| v1 v2  
-| -------
1| 10     
2| 20     
3| 30 1000
4|    2000
5|    3000
```
## 8.7 Complex Column Data¶

### 8.7.1 Simple Example¶

There is no restriction on the column lists of a table other than they are rectangular. In practice, simple lists are preferable because they are faster, more storage-efficient and are easier to process. There are situations in which it may be convenient to use nested column lists, although far less often than most q programmers seem to think. In general, it is more efficient to flatten columns and then use joins or grouping to create structure on the way out. For those who simply must have nested column structure, we provide examples here.

Suppose we want to keep track of a pair of daily observations, say a low temperature and a high temperature in Hawaii. The normal form stores the low and high values in separate columns with flat lists for rows and columns.
    
    
```q
q)tf:([] d:2015.01.01 2015.01.02; l:67.9 72.8; h:82.1 88.4)
q)tf 0
_
q)tf `l
_
q)tf `h
_
```
Alternatively, we can store pairs in a single column, resulting in nested lists for records and columns.
    
    
```q
q)tp:([] d:2015.01.01 2015.01.02; lh:(67.9 82.10; 72.8 88.4))
q)tp 0
d | 2015.01.01
lh| 67.9 82.1
q)tp `lh
67.9 82.1
72.8 88.4
```
This example can easily be generalized to the situation of n-tuples. For example, we could store the daily values of a yield curve. Further, the fields can have different length or even different type, although the latter is strongly discouraged as it will not be possible to persist them in kdb+.

### 8.7.2 Operations on Compound Column Data¶

The case of a nested column in which all items are simple lists of the same type is handled specially in kdb+. We call this a _compound_ column. There is no requirement that the simple lists all have the same length.

As an example, say we want to analyze the weekly gross revenues for movies and we don't care about the titles (we don't have room to display them here). Since there will be a different number of movies in release each week, the number of observations in each field will vary. An oversimplified time series that fits within the margins looks like the following in which the gross revenues are in millions.
    
    
```q
q)tm:([] wk:2015.01.01 2015.01.08; rv:(38.92 67.34; 16.99 5.14 128.23 31.69))
q)tm
wk         rv                     
----------------------------------
2015.01.01 38.92 67.34            
2015.01.08 16.99 5.14 128.23 31.69
```
Storing complex values in a single column in a table enables sophisticated operations to be performed in a single expression, provided you remember:

Nested columns mean iterators. Lots of iterators.

Using our movie data, we can produce the sorted, the average and high gross for each week in one expression.
    
    
```q
q)select wk, srt:desc each rv, avgr:avg each rv, hi:max each rv from tm
wk         srt                     avgr    hi    
-------------------------------------------------
2015.01.01 67.34 38.92             53.13   67.34 
2015.01.08 128.23 31.69 16.99 5.14 45.5125 128.23
```
While sorts and aggregates such as `MAX` and `AVG` are standard SQL, think of how you’d produce the sorted sub list and the aggregates together. In your favorite traditional programming environment, you'll soon discover that you need a sordid list of rows and a loop to unravel it.

Now let’s compute the drops between the ranked revenue numbers within each week. No loop required in q.
    
    
```q
q)select wk, drp:neg 1_'deltas each desc each rv from tm
wk         drp             
---------------------------
2015.01.01 ,28.42          
2015.01.08 96.54 14.7 11.85
```
### 8.7.3 Compound Foreign Key¶

A nested column is how to make a foreign key on a compound primary key. We recast the galactic travelers to make a keyed table with common key of last and first name.
    
    
```q
q)ktc:([lname: `Dent`Beeblebrox`Prefect; fname:`Arthur`Zaphod`Ford]; iq:98 42 126)
```
We create a details table with a foreign key over `ktc` by placing the names in the foreign key column.
    
    
```q
q)tdetails:([] name:`ktc$(`Beeblebrox`Zaphod;`Prefect`Ford;`Beeblebrox`Zaphod); 
    sc:36 126 42)
```
The columns of `ktc` are available as virtual columns from `tdetails`.
    
    
```q
q)select name.lname, name.iq, sc from tdetails
lname      iq  sc 
------------------
Beeblebrox 42  36 
Prefect    126 126
Beeblebrox 42  42 
```
When defining the schema of a table with a compound key column, specify the foreign key as a cast of an empty list.
    
    
```q
q)([] name:`ktc$();sc:`long$())
_
```
When the foreign key comprises multiple types you also cast the general list with the foreign table in its schema.
    
    
```q
q)ktc:([k1:1 2 3; k2:2001.01.01 2001.01.02 2001.01.03] v: 1. 2. 3.)
q)tfc:([] fk:`ktc$(); s:`symbol$())
q)tfc,:(`ktc$(1; 2001.01.01); `a)
q)tfc,:(`ktc$(1; 2015.01.01); `a)
'cast
```
The failure of the cast in the second append is correct; it is enforced referential integrity.

## 8.8 Attributes¶

Attributes are metadata that you attach to lists of special forms. They are also used on a dictionary domain or a table column to speed retrieval for some operations. The q interpreter can make certain optimizations based on the structure of the list implied by the attribute.

Attributes (other than ``g#`) are descriptive rather than prescriptive. By this we mean that by applying an attribute you are asserting that the list has a special form, which q will check. It does **not** instruct q to (re)make the list into the special form; that is your job. A list operation that respects the form specified by the attribute leaves the attribute intact (other than ``p#`), while an operation that breaks the form results in the attribute being removed in the result.

The syntax for applying an attribute is (yet) another [overload of `#`](https://code.kx.com/q/ref/set-attribute/ "Set Attribute"), whose left operand is a symbol specifying the attribute and whose right operand is the target list. 

The attribute is applied to the target list in place, not on a copy.

KX says not to expect significant benefit from an attribute for fewer than a million items. This is why attributes are not automatically applied in mundane situations. You should test your use case to see whether applying an attribute provides performance benefit. Do not just apply attributes blindly, as they consume resources.

### 8.8.1 Sorted ``s#`¶

Applying the sorted attribute ``s#` to a simple list indicates that the items of the list are sorted in ascending order; there is no way to indicate a descending sort. When a list has the sorted attribute, linear search is replaced with binary search, which makes certain operations faster – for example, [Find](https://code.kx.com/q/ref/find/) `?`, [Equal](https://code.kx.com/q/ref/equal/) `=`, [`in`](https://code.kx.com/q/ref/in/) and [`within`](https://code.kx.com/q/ref/within/).

When an attribute is successfully applied to a list, it becomes part of the list and is displayed on the q console. Observe that q checks to see that the list is actually sorted when the attribute is applied.
    
    
```q
q)`s#1 2 4 8
`s#1 2 4 8
q)`s#2 1 3 4
's-fail
```
The sort function [`asc`](https://code.kx.com/q/ref/asc/) automatically applies the sorted attribute to its result but [`til`](https://code.kx.com/q/ref/til/) does not.
    
    
```q
q)asc 2 1 8 4
`s#1 2 4 8
q)til 5
0 1 2 3 4
```
When a list with an attribute is amended with `,:` the result is checked to see that the attribute is preserved; if not, it is removed.
    
    
```q
q)L:`s#1 2 3 4 5
q)L,:6
q)L
_
q)L,:0
q)L
_
```
One place to apply the sorted attribute is on the date or time column of a simple time series.
    
    
```q
q)t:([] ti:`s#00:00:00 00:00:01 00:00:03; v:98 98 100.)
q)meta t
c | t f a
--| -----
ti| v   s
v | f    
```
Applying the sorted attribute to a table unintuitively applies the parted attribute (see next section) to the first column.
    
    
```q
q)meta `s#([] ti:00:00:00 00:00:01 00:00:03; v:98 98 100.)
c | t f a
--| -----
ti| v   p
v | f    
```
Applying the sorted attribute to a dictionary applies the attribute to the key list. Lookup is faster because binary search is used. A side effect of the way binary search is implemented is that the mapping given by the dictionary is now a step function, meaning that values between successive key values are "filled in."
    
    
```q
q)d:`s#10 20 30 40 50!`a`b`c`d`e
q)key d
`s#10 20 30 40 50
q)d 10
`a
q)d 12
`a
q)d 15
_
q)d 20
`b
```
Since a keyed table is a dictionary of tables, applying the sorted attribute to a keyed table applies to the table of keys that, in turn, applies to the initial key column.
    
    
```q
q)meta `s#([k:1 2 3 4] v:`d`c`b`a)
c| t f a
-| -----
k| j   s
v| s    
```
### 8.8.2 Unique ``u#`¶

Applying the unique attribute ``u#` to a list indicates that the items of the list are distinct. Knowing that the elements of a list are unique makes [`distinct`](https://code.kx.com/q/ref/distinct/) the identity function and shortens some operations – i.e., if an operation has found one of what you’re looking for, it’s done. 

Note

Applying ``u#` essentially causes q to create a hash table, which uses storage and adds overhead. It is best to apply the attribute after the list has been created, if possible. If you have a column that will always have unique values and your large table does not change often, you can get a significant performance speedup with ``u#`.

Observe that uniqueness is checked. Don’t take the error message personally.
    
    
```q
q)`u#2 1 4 8
`u#2 1 4 8
q)`u#2 1 4 8 2
'u-fail
```
An amend that does not preserve uniqueness causes the attribute to be lost.
    
    
```q
q)L:`u#2 1 4 8
q)L,:3
q)L
_
q)L,:2
q)L
2 1 4 8 3 2
```
The unique attribute can be applied to the domain of a dictionary, a column of a table, or the key column of a keyed table. It cannot be applied to a dictionary, a table or a keyed table directly.

### 8.8.3 Parted ``p#`¶

The parted attribute ``p#` indicates that all common occurrences of any value in a list are adjacent. Viewing the list as a map, the parted attribute says that its graph is a step function with distinct steps. The parted attribute can be applied to a simple list of any type whose underlying value is integral – e.g., the integer types, dates, times, timestamps, timespans. You can also apply parted on a list of enumerated symbols since the underlying index values are integral.

Note

The parted attribute causes q to create an underlying structure that keeps track of the steps. Because the attribute is not preserved under most operations you should apply it only after the list is fully created. One exception is that when two lists have ``p#` and their values are disjoint, the attribute will be preserved when the lists are joined with `,`.

Here is a simple example of a parted list. Notice that the items are neither sorted nor unique.
    
    
```q
q)`p#2 2 2 1 1 4 4 4 4 3 3
`p#2 2 2 1 1 4 4 4 4 3 3
```
Notice that the rather nasty error does not say `'p-fail`; rather it contains a `'u-fail` error.
    
    
```q
q)`p#2 2 2 1 1 4 4 4 4 3 3 2
k){$[3=x;(`#y;`u#y i;(i:&~=':y),#y);(y;`u#!r;+\0,#:'x;,/x:. r:=y)]}
'u-fail
#
`u
2 1 4 3 2
q.o))
```
With the exception in the note above, the parted attribute is **not** preserved under any operation on the list, even if the operation preserves the property.
    
    
```q
q)L:`p#1 1 2 3 3
q)L
_
q)L,:3
q)L
1 1 2 3 3 3
```
Historical time-series databases for ticker symbols are usually sorted by time within symbol with the parted attribute applied to the (enumerated) symbol column. This makes queries by ticker fast and guarantees that results for a given symbol are returned in time order.

### 8.8.4 Grouped ``g#`¶

The grouped attribute ``g#` differs from other attributes in that it can be applied to any list. It causes q to create and maintain an index – essentially a hash table. Grouped can be applied to a list when no other assumptions about its structure can be made.

For example,
    
    
```q
q)`g#1 2 3 2 3 4 3 4 5 2 3 4 5 4 3 5 6
_
q)L:`g#100?100
q)L
_
q)L,:1 1 1 1
q)L
_
```
The grouped attribute is maintained as operations are performed on the list, which can cause significant processing overhead in addition to the storage required. Best to apply it after the entire list has been created, if possible.

Applying the grouped attribute to a table column roughly corresponds to placing an index on a column in an RDBMS. As of this writing (Sep 2015), in q3.2 the maximum number of grouped attributes that can be placed on a single table is unlimited.

### 8.8.5 Remove Attribute ``#`¶

The operations ``#` removes any attribute that may currently be applied. For example,
    
    
```q
q)L:`s#til 10
q)L
_
q)`#L
_
```
Back to top 
