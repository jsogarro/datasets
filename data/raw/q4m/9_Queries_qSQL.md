# 9\. Queries: q-sql¶

## 9.0 Overview¶

We call the collection of functions for manipulating tables _q-sql_ , since many of them resemble their SQL counterparts in form or function. The usual suspects such as _insert_ , _select_ , _update_ , are present, as well as functionality that is not available in traditional SQL. But appearances can be deceiving: there are some significant differences in the syntax and behavior.

The first important difference is that a q table has ordered rows and columns. This is particularly useful when dealing with the situation where records arrive in time order. Appending them to a table ensures that they enter – and stay – in order. Subsequent select operations always retrieve the records in order without any need for sorting.

A second difference is that a q table is stored physically as a collection of column lists. This means that operations on column data are vector operations. Moreover, for simple column lists, atomic, aggregate and uniform functions applied to columns are especially simple and fast since they reduce to direct memory addressing.

A third difference is that q-sql provides upsert semantics. Recall that upsert semantics on a dictionary mean that when a key-value pair is applied with [`,`](https://code.kx.com/q/ref/join/ "Join") and the key is present, the value is updated; otherwise the pair is inserted. In the context of tables and keyed tables, which are both dictionaries, this has far-reaching consequences for many common operations, including joins. Upsert semantics permeate q-sql.

In this chapter, we cover the important features of q-sql, beginning with simple examples for each. Eventually more complex examples are introduced.

Some examples are based on the `sp.q` script included in the q installation files. Tables in the script are,
    
    
```q
q)meta s
c     | t f a
------| -----
s     | s
name  | s
status| j
city  | s
q)meta p
c     | t f a
------| -----
p     | s
name  | s
color | s
weight| j
city | s
q)meta sp
c  | t f a
---| -----
s  | s s
p  | s p
qty| j
```
You should [load](../13_Commands_and_System_Variables/#13111-load-l) and display these tables in your console session now.

## 9.1 Inserting Records¶

There are multiple ways to insert – i.e., append – records in q.

The `upsert` function is superior to `insert` and is to be preferred. We include `insert` for nostalgia only.

### 9.1.0 Append Using Assign¶

Since a table is (logically) a list of records, it is possible to append records in place using `,:`. Type checking on field values is performed.
    
    
```q
q)t:([] name:`symbol$(); iq:`int$())
q)t,:`name`iq!(`Beeblebrox; 42)
q)t,:`name`iq!(`Dent; 98.0)
'type
```
_Assign_ can also be used with a row of naked field values provided the fields align exactly with the target columns.
    
    
```q
q)t,:(`Prefect; 126)
```
Applying Assign to a table repeatedly with the same argument results in duplicate records.
    
    
```q
q)t,:(`Prefect; 126)
q)t,:(`Prefect; 126)
q)t
_
```
You can use Assign to append to a keyed table using the full record form, but you will quickly see why no one does this.
    
    
```q
q)kt:([eid:`long$()] name:`symbol$(); iq:`long$())
q)kt,:(enlist (enlist `eid)!enlist 1001)!enlist `name`iq!(`Beeblebrox; 42)
```
It is much easier to use naked field values, provided they align exactly with both the key and value columns.
    
    
```q
q)kt,:(1002; `Dent; 98)
q)kt
_
```
Assign has upsert semantics on keyed tables, so repeated operation on the same key will retain only the last values.
    
    
```q
q)kt,:(1002; `Dent; 101)
q)kt
_
```
### 9.1.1 Basic insert¶

Those who prefer the familiarity of SQL can append records to an existing global table using the binary function [`insert`](https://code.kx.com/q/ref/insert/) whose left operand is a symbol containing the name of a **global** table (_target_) and whose right argument is a record, or list of records, conforming to the target. The result is a list of integers representing the row number(s) of the appended record(s).

Important

Since `insert` essentially reduces to amend in place, fields are type checked.

  * If the target column is simple, the type must match exactly.
  * If the target column is an untyped empty list, the result will take the type of the field in the initial `insert`.



For a regular – i.e., non-keyed – table, the effect of `insert` is to append a new record with the specified field values. Let’s use our simple example.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:42 98 126)
```
Here is how to insert a single record or the equivalent row list.
    
    
```q
q)`t insert (`name`iq)!(`Slartibartfast; 134)
,3
q)`t insert (`Marvin; 150)
,4
q)t
_
```
Repeatedly applying `insert` to a table with the same argument results in duplicate records.
    
    
```q
q)`t insert (`Marvin; 150)
_
q)`t insert (`Marvin; 150)
_
q)
_
```
Since a list of conforming records is a table, inserting a list of records is the same as inserting a table. Observe that use of `3#` to truncate the table.
    
    
```q
q)t:3#t
q)`t insert (`name`iq!(`Slartibartfast; 134); (`name`iq!(`Marvin; 200)))
_
q)t
_
q)t:3#t
q)`t insert ([] name:`Slartibartfast`Marvin; iq:134 200)
_
```
You can also use `insert` in prefix form, possibly with the table name projected. For example, the previous insert can be written as,
    
    
```q
q)insert[`t; (`Slartibartfast; 134)]
_
q)insert[`t;] (`Slartibartfast; 134)
_
```
### 9.1.2 Bulk Columnar Insert¶

We have seen that it is possible to insert a naked list of row values instead of the full record dictionary. We have also seen that it is possible to bulk insert a list of conforming records, which is just a table with the same schema as the target.

It is also possible to bulk insert naked field values but there is a twist. To bulk insert naked field values, you provide a list of **columns** not a list of rows.
    
    
```q
q)t:([] name:`Dent`Beeblebrox; iq:98 42)
q)`t insert ((`Prefect; 126); (`Marvin; 200))
'type
q)`t insert (`Prefect`Marvin; 126 200)
8 9
```
After a brief q Zen meditation, you will realize that this is consistent with the previous bulk insert of a table, since a table is a collection of columns. From this perspective, we should view the insertion of a single naked row more correctly as a trivial list of column atoms.

### 9.1.3 Insert into Empty Tables¶

Inserting into a table that has been initialized with empty lists of general type causes the result table to take the type of the first record inserted. In particular, an atomic field in the inserted record results in a simple column with its type.
    
    
```q
q)t:([] name:(); iq:())
q)`t insert (`Dent;98)
,0
q)meta t
c   | t f a
----| -----
name| s
iq  | j
```
This is fine as long as all the types in the initial record are correct. Should any field have an unintended type, subsequent records of the correct type will all be rejected.
    
    
```q
q)t:([] name:(); iq:())
q)`t insert (`Dent;98.0)
,0
q)`t insert (`Beeblebrox; 42)
'type
```
Recommendation

It is good practice to type all columns in an empty table. This will ensure that incorrect types are rejected and correct ones accepted.
    
    
```q
 q)t:([] name:`symbol$(); iq:`int$())
 q)`t insert (`Dent;98.0)
 'type
 q)`t insert (`Beeblebrox; 42)
 ,0
```
It is also possible to insert a **list** of conforming records (i.e., a table) into a table that does not exist. This is the same as assigning the table to a variable of the specified name.
    
    
```q
q)tnew
'tnew
q)`tnew insert enlist `c1`c2!(`a; 10)
_
q)tnew
_
```
### 9.1.4 Insert and Foreign Keys¶

When inserting data into a table that has foreign key(s), the values destined for the foreign key column(s) are checked to ensure that they appear in the primary key column(s) pointed to by the foreign key(s). This is referential integrity (well, half of it).

Returning to our previous foreign-key example.
    
    
```q
q)kt:([eid:1001 1002 1003] name:`Dent`Beeblebrox`Prefect; iq:98 42 126)
q)tdetails:([] eid:`kt$1003 1002 1001 1002 1001; sc:126 36 92 39 98)
```
The first insert in the following succeeds but the second fails when trying to enumerate the foreign-key value 1042 that does not appear in `kt`.
    
    
```q
q)`tdetails insert (1002;42)
,5
q)`tdetails insert (1042;150)
'cast
```
Recall that enumeration is a form of cast.

### 9.1.5 Insert into Keyed Tables¶

You can use `insert` to append data to a keyed table, but this probably does not have the desired semantics. Specifically, you can insert into a keyed table only if the key value is **not** already in the table. For this and other reasons, `upsert` should normally be preferred over `insert`.

Since a keyed table is a dictionary, to use `insert` we should properly provide a dictionary entry comprising a key record and a value record.
    
    
```q
q)kt:([eid:1001 1002] name:`Dent`Beeblebrox; iq:98 42)
q)`kt insert (enlist ((enlist `eid)!enlist 1003))!enlist `name`iq!(`W; 21)
,2
```
Yikes! Nobody does this. Instead, you provide a list of raw field values with the proviso that they align exactly across the key and value columns.
    
    
```q
q)`kt insert (1005; `Marvin; 200)
,3
q)`kt insert (1004;`Slartibartfast;158)
,4
```
Repeating the last insert now fails because the key value 1004 already exists.
    
    
```q
q)`kt insert (1004; `Marvin; 200)
'insert
```
The records in the keyed table are stored in insert order rather than key order.
    
    
```q
 q)kt
 _
```
## 9.2 Upsert¶

The [`upsert`](https://code.kx.com/q/ref/upsert/) template is like `insert`, only better. Except for the last sub-section on keyed tables, all the examples in the previous section all work the same for `upsert`.

### 9.2.1 Upsert Replacing Insert¶

Here we repeat some examples from the previous section to demonstrate that `upsert` can (and should) be used in place of `insert` for appending rows to a table.
    
    
```q
q)t:([] name:`Dent`Beeblebrox`Prefect; iq:42 98 126)
q)`t upsert (`name`iq)!(`Slartibartfast; 134)
`t
q)`t upsert (`Marvin; 150)
_
q)`t upsert ([] name:`Slartibartfast`Marvin; iq:134 200)
_
q)t:3#t
q)upsert[`t; (`Slartibartfast; 134)]
_
```
Tip

To bulk upsert naked field values, use rows instead of columns. 
    
    
```q
 q)t upsert ((`Prefect; 126); (`Marvin; 200))
 _
```
### 9.2.2 Upsert by Name¶

A limitation of `insert` is that it uses pass-by-name, so it can only operate against global tables. In contrast, `upsert` supports both pass-by-name and pass-by-value. Thus it can be used with anonymous or local tables.
    
    
```q
q)([] c1:`a`b; c2:10 20) upsert (`c; 30)
c1 c2
-----
a  10
b  20
c  30
q)f:{t:([] c1:`a`b; c2:10 20); t upsert x}
q)f (`c; 30)
c1 c2
-----
a  10
b  20
c  30
```
### 9.2.3 Upsert on Keyed Tables¶

We have seen that `insert` has undesirable semantics on keyed tables – i.e., it rejects “duplicate” keys. What we really want is, well, upsert semantics.
    
    
```q
q)`kt upsert (1001; `Beeblebrox; 42)
_
q)`kt upsert (1001; `Beeblebrox; 43)
_
q)kt
eid | name iq
----| -------------
1001| Beeblebrox 43
```
This is the second reason to use `upsert` instead of `insert`.

### 9.2.4 Upsert on Persisted Tables¶

You can use `upsert` to append records to serialized and [splayed tables](https://code.kx.com/q/kb/splayed-tables/). Simply pass the handle of the file or splayed directory as the name of the table. This is the final strike against `insert`, since it cannot do this.

We serialize a table and then append a row to it.
    
    
```q
q)`:/q4m/tser set ([] c1:`a`b; c2:1.1 2.2)
q)`:/q4m/tser upsert (`c; 3.3)
`:/q4m/tser
q)get `:/q4m/tser
_
```
Upserting to a serialized table reads the entire table into memory, updates it and writes out the result.

Next we splay a table and then append a row to it. Observe that we ensure that all symbols are enumerated, as required for splayed tables.
    
    
```q
q)`:/q4m/tsplay/ set ([] c1:`sym?`a`b; c2:1.1 2.2)
`:/q4m/tsplay/
q)`:/q4m/tsplay upsert (`sym?`c; 3.3)
`:/q4m/tsplay
q)select from `:/q4m/tsplay
_
```
Upserting to a splayed table does **not** read the persisted image into memory; rather, it appends to the ends of the column files. This allows incremental creation of large splayed (or partitioned) tables by upserting chunks that comfortably fit into memory. See [§11.3](../11_IO/#113-splayed-tables) for more details on splayed tables.

Upserting to a table in either form will destroy any attributes on table columns. You will have to reapply them.

## 9.3 The `select` Template¶

In this section we investigate the general form of [`select`](https://code.kx.com/q/ref/select/), which like all q-sql templates, has required and optional elements. The template elements contain phrases that are expressions (presumably) involving column values of a specified table. The template is converted by the interpreter into a functional form and is applied against the table to produce a result table. While the syntax and behavior of `select` resemble the analogous SQL statement, the underlying mechanics are quite different.

The result of `select` is always a table.

We examine each of the constituents of `select` in detail. We introduce the concepts with illustrative examples using trivial tables so that the basic mechanics are not obscured by large or complex data.

### 9.3.1 Syntax¶

The `select` template has the following form, where elements enclosed in matching angle brackets `<...>` are optional.

`select` <_p s_> <`by` _p b_> `from` _t exp_ <`where` _p w_>

The `select` and `from` keywords are required; omission or mistyping either results in an error. The table expression _t exp_, which is any q expression whose value is a table or keyed table, is also required. The remaining elements _p s_, _p b_ and _p w_ are optional. They are called the Select, the By and the Where phrases, respectively.

Tip

If `where` is present and texp is itself a select expression, the inner expression should be enclosed in parentheses to avoid confusion in the binding of the Where phrase.

Each phrase in the select template is a comma-separated list of subphrases. A _subphrase_ is an arbitrary q expression (presumably) involving columns of _t exp_ or columns of another table accessed via foreign key. The evaluation of subphrases within a phrase is sequenced left-to-right by the commas, but each subphrase expression is evaluated right-to-left, like any q expression.

Commas

The commas separating the subphrases are separators, so it is not necessary to enclose a subphrase in parentheses unless the expression contains the Join operator `,`. Any expression containing the operator `,` within any template phrase must be enclosed in parentheses or it will be interpreted as the separator. Forgetting this is both easy and painful.

The order of evaluation of the `select` template is:

  1. From expression _t exp_

  2. Where phrase _p w_

  3. By phrase _p b_

  4. Select phrase _p s_




### 9.3.2 The Select Phrase¶

We begin our examples with the Select phrase because it is the easiest. We shall use the following table for our examples.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
```
The Select phrase specifies the columns in the result table, one per subphrase. If the Select phrase _p s_ is absent, all columns are returned. There is no need for the `*` wildcard of SQL.
    
    
```q
q)select from t
_
q)t~select from t
1b
```
To specify result columns, list them separated by commas. Whitespace after commas is optional but some think it improves readability, especially for complicated queries.
    
    
```q
q)select c1, c3 from t
_
```
### 9.3.2.1 Result Column Names¶

To specify names for the result columns, place the name followed by colon before the subphrase.
    
    
```q
q)select c1, res:2*c2 from t
c1 res
------
a  20
b  40
c  60
```
**Notes**

  * The colon used to specify a name is **not** assignment; it is simply part of the syntax of the template.
  * Just as with the use of colon in a variable assignment, the column name is part of the syntactic sugar. It is not a symbol and **cannot** be parameterized. Use functional form if you need this – see §9.12.
  * Unlike in SQL, columns in the Select phrase do not actually exist until the final result table is returned. Thus a computed column **cannot** be used in other column expressions.



If you do not provide a name for a computed column, q determines one.

  * Normally the name is taken from the left-most term in the column expression.
  * When q cannot determine a name it uses `x`.
  * If q’s chosen name duplicates a previously determined column name, it will suffix it with `1`, `2`, etc. to make it unique. Recall that columns are sequenced from left to right.


    
    
```q
q)select c1, c1, 2*c2, c2+c3, string c3 from t
c1 c11 x  c2   c3
--------------------
a  a   20 11.1 "1.1"
b  b   40 22.2 "2.2"
c  c   60 33.3 "3.3"
```
### 9.3.2.2 The Virtual Column `i`¶

A virtual column `i` represents the offset of each record in the table – i.e., `i` is the row number. It is implicitly available in the Select phrase.
    
    
```q
q)select i, c1 from t
x c1
----
0 a
1 b
2 c
```
Observe that `select` does not carry the name `i` into the result.

Tip

You can name the result column `ix` to avoid confusion with the always-present virtual column `i`.
    
    
```q
 q)select ix:i, c1 from t
 _
```
### 9.3.2.3 `select distinct`¶

The special form `select distinct` returns only unique records in the result – i.e., it eliminates duplicates.
    
    
```q
q)select distinct from ([] c1:`a`b`a; c2:10 20 10)
c1 c2
-----
a  10
b  20
```
### 9.3.2.4 `select[]`¶

You can return the first or last _n_ records in a select by using function parameter syntax after `select`. A positive integer parameter returns the first records, a negative parameter the last.
    
    
```q
q)select[2] from s where city<>`athens
s | name status city
--| -------------------
s1| smith 20 london
s2| jones 10 paris

q)select[-1] from s where city<>`athens
s | name status city
--| -------------------
s4| clark 20 london
```
We could achieve the same result using [`#`](https://code.kx.com/q/ref/take/ "Take") after the `select` (i.e., to its left).
    
    
```q
q)2#select from s where city<>`athens
_
q)-1#select from s where city<>`athens
_
```
The difference is that the `#` construct requires computing the entire result set and then keeping only the desired rows, whereas `select[n]` only extracts the desired number of rows. The latter will be faster and consume less memory for large tables.

This syntax is extended to `select[n m]` where `n` is the starting row number and `m` is the number of rows.
    
    
```q
q)select[1 2] from s where city<>`athens
_
```
One final extension of the syntax specifies a sorting criterion inside the brackets. For ascending sort, place `<` before a column name and for descending sort use `>`.
    
    
```q
q)select[>name] from s where city<>`athens
_
q)select[<city] from s where city<>`athens
_
```
You can combine the two syntax extensions by separating them with a semicolon.
    
    
```q
q)select[2; >name] from s where city<>`athens
_
```
The items inside the brackets must be in this order.

### 9.3.2.5 Select on Nested Columns¶

You can use `select` on tables with nested columns but things become more complicated. The rule of thumb is that you will need iterators… lots of iterators.

Let’s take a simple example.
    
    
```q
q)show tnest:([] c1:`a`b`c; c2:(10 20 30; enlist 40; 50 60))
c1 c2
-----------
a  10 20 30
b  ,40
c  50 60
```
What can we do with this table? We **cannot** apply an aggregate or uniform operation straight to `c2` since the fields do not conform.
    
    
```q
q)select avg c2 from tnest
'length
```
We **can** apply an aggregate or uniform function to each field of `c2` in `tnest`.
    
    
```q
q)select avg each c2 from tnest
_
```
Similarly we can use [Each](https://code.kx.com/q/ref/maps/#each) to compute a weighted average using two columns.
    
    
```q
q)update c3:(1.1 2.2 3.3; enlist 4.4; 5.5 6.6) from `tnest
`tnest
q)select wtavg:c2 wavg' c3 from tnest
_
```
### 9.3.3 Filtering with `where`¶

The Where phrase controls which records of the input table are actually used in the query. The effect of the Where phrase is to include only the records that meet its criteria.

### 9.3.3.1 Basic where¶

The action generalizes the built-in [`where`](https://code.kx.com/q/ref/where) function on lists (See [§3.12.3](../3_Lists/#3123-where)). Recall that a table is logically a list of records.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
q)t where t[`c2]>15
c1 c2 c3
---------
b  20 2.2
c  30 3.3
q)select from t where c2>15
_
```
In fact, you can provide a boolean list to `where`.
    
    
```q
q)select from t where 011b
_
```
### 9.3.3.2 The Virtual Column `i` in `where`¶

The virtual column `i` is useful for paginating a table. Use `within`, which returns a boolean indicating whether the left operand is in the closed interval specified in the right operand, to determine the bounds of the page.
    
    
```q
q)tbig:100#t
q)select from tbig where i within 50 99
_
```
To use this construct with non-literal values as the endpoints, you **must** use general-list notation, since simple-list notation cannot be used with variables.
    
    
```q
q)s:50
q)e:99
q)select from tbig where i within (s;e)
_
```
### 9.3.3.3 Multiple Where Subphrases¶

Each Where subphrase is a predicate expression that produces a boolean result vector corresponding to records passing or failing a criterion. The **logical** effect of multiple subphrases is to join them with “and”. Note that the parentheses are necessary in the second query.
    
    
```q
q)r1:select from t where c2>15,c3<3.0
q)r2:select from t where (c2>15)&c3<3.0
q)r1~r2
1b
```
However, since the Where subphrases are sequenced from left-to-right, their order affects the actual processing. As each subphrase is applied, only the records it passes are tested in the next subphrase. The net effect is a progressively narrowed sublist of rows to consider. There is often an optimal order that significantly narrows in the first one or few subphrases, which in turn reduces the amount of processing.

Place the most limiting `where` subphrase first, followed by others in decreasing strictness.

Consider the following table comprising a million observations of two variables `a` and `b` taken every millisecond starting at midnight. On the author’s laptop, the version of the query that narrows the time interval first executes in under a millisecond whereas the one specifying the variable name first takes 15 milliseconds.
    
    
```q
q)t:([] c1:00:00:00.000+til 1000000;c2:1000000?`a`b;c3:1000000?100.)
q)\t select from t where c1 within 00:00:01.000 00:00:01.999, c2=`a
0
q)\t select from t where c2=`a, c1 within 00:00:01.000 00:00:01.999
15
```
### 9.3.3.4 Nested Columns in Where¶

Nested columns generally require iterators, and the Where phrase is no different. A common mistake made by qbies is, when trying to find a specific string in a column of strings, they forget that strings are not first-class in q.
    
    
```q
q)t:([] f:1.1 2.2 3.3; s:("abc";enlist "d";"ef"))
q)select from t where s="ef"
'length
=
q)select from t where s~"ef"
f s
---
q)select from t where s~\:"ef"
f s
--------
3.3 "ef"
```
The first query is in error because it tests atomic equality between a simple list and a nested list. The second query does not achieve the desired result because it asks if the entire column matches the specified string. The final query works because it tests the specified string for match against each string in the column.

Tip

For matching strings, the [`like`](https://code.kx.com/q/ref/like/) operator is more efficient. See [§A.45](../A_Built-in_Functions/#a45-like) for a detailed description but we only need the special case that asks if two strings match exactly – i.e., no wildcards. The fact that `like` is pseudo-atomic in the left operand makes it suited for comparing against a column. It is actually faster than the above expression with `~\:` for large lists.
    
    
```q
q)select from t where s like "ef"
_
```
### 9.3.3.5 `fby` in Where¶

A common use case in a query is to filter on groups. For example, in the following example using the table `p` from the distribution samples, we wish to include only the records having the maximum weight in their respective cities. We could start with a correlated subquery.
    
    
```q
q)select max weight by city from p
city  | weight
------| ------
london| 19
paris | 17
rome  | 17
```
But you run into the issue of what to do with the other fields in the subquery. Do without aggregation, which loses the information of where the maximum occurs?
    
    
```q
q)select name, color, max weight by city from p
city  | name           color        weight
------| ----------------------------------
london| `nut`screw`cog `red`red`red 19    
paris | `bolt`cam      `green`blue  17    
rome  | ,`screw        ,`blue       17
```
Or do you apply other aggregates that will pick values that are uncorrelated with the original?
    
    
```q
q)select first name, first color, max weight by city from p
city  | name  color weight
------| ------------------
london| nut   red   19    
paris | bolt  green 17    
rome  | screw blue  17
```
In SQL you would use `HAVING`, but q is having none of that. Instead use [`fby`](https://code.kx.com/q/ref/fby) in the Where phrase.  Since it returns the value of the aggregate across each group, you simply compare the target column to the `fby` result to get a boolean vector with `1b` at precisely the records whose fields match the aggregate on the group.

Used in a Where phrase, `fby` takes the form

(_f agg_;_expr col_) `fby` _c_

The left operand is a two-item list comprising an aggregate function _f agg_ and a column expression _expr col_ on which the function will be applied. The right operand _c_ is the column to be grouped.

In our example,
    
    
```q
q)select from p where weight=(max;weight) fby city
p | name  color weight city  
--| -------------------------
p2| bolt  green 17     paris 
p3| screw blue  17     rome  
p6| cog   red   19     london
```
Now we include another Where phrase for the desired result.
    
    
```q
q)select from p where weight=(max;weight) fby city,color=`blue
p | name  color weight city
--| -----------------------
p3| screw blue  17     rome
```
To group on multiple columns, encapsulate them in an anonymous table in the right operand of `fby`.
    
    
```q
q)t:([]sym:`IBM`IBM`MSFT`IBM`MSFT;
    ex:`N`O`N`N`N;
    time:12:10:00 12:30:00 12:45:00 12:50:00 13:30:00;
    price:82.1 81.95 23.45 82.05 23.40)
q)select from t where price=(max;price) fby ([]sym;ex)
sym  ex time     price
----------------------
IBM  N  12:10:00 82.1 
IBM  O  12:30:00 81.95
MSFT N  12:45:00 23.45
```
### 9.3.4 Grouping and Aggregation¶

In contrast to SQL, where grouping and aggregation are performed together, in q-sql they are independent. In this section we use the tables defined in the `sp.q` script included in the distribution.
    
    
```q
q)p
p | name  color weight city  
--| -------------------------
p1| nut   red   12     london
p2| bolt  green 17     paris 
p3| screw blue  17     rome  
p4| screw red   14     london
p5| cam   blue  12     paris 
p6| cog   red   19     london
q)s
s | name  status city  
--| -------------------
s1| smith 20     london
s2| jones 10     paris 
s3| blake 30     paris 
s4| clark 20     london
s5| adams 30     athens
q)sp
s  p  qty
---------
s1 p1 300
s1 p2 200
s1 p3 400
s1 p4 200
s4 p5 100
s1 p6 100
s2 p1 300
s2 p2 400
s3 p2 200
s4 p2 200
s4 p4 300
s1 p5 400
```
### 9.3.4.1 Aggregation without Grouping¶

When an aggregate function is applied against a column of simple type in the Select phrase, the result is an atom. If all columns in the Select phrase are computed with aggregation and there is no grouping, the result will be a table with a single row – e.g., a summary or rollup. While q has many built-in aggregates, you can also define and use your own.

Here we calculate the total and mean order quantity using the built-in aggregates [`sum`](https://code.kx.com/q/ref/sum/) and [`avg`](https://code.kx.com/q/ref/avg/).
    
    
```q
q)select total:sum qty, mean:avg qty from sp
total mean    
--------------
3100  258.3333
```
### 9.3.4.2 Grouping without Aggregation¶

The By phrase groups rows having common values in specified column(s), much like `GROUP BY` in SQL. The result of a query including a By phrase is a keyed table whose key column(s) are those in the By phrase. This is well-defined because the grouping along like values ensures uniqueness of the keys.

The action of the By phrase is a generalization of the built-in function [`group`](https://code.kx.com/q/ref/group/) on lists. (See [§3.12.4](../3_Lists/#3124-group).) A query that groups without aggregation results in nested columns. One way to think of this is that each group of values is folded into a single field in the result.
    
    
```q
q)t:([] c1:`a`b`a`b`c; c2:10 20 30 40 50)
q)t[`c2] group t[`c1]
a| 10 30
b| 20 40
c| ,50
q)select c2 by c1 from t
c1| c2
--| -----
a | 10 30
b | 20 40
c | ,50
```
Tip

Grouping without aggregation is the most common way qbies unintentionally create nested columns. Nested columns are slower, more cumbersome to use (they require a heavy dose of iterators) and are usually unnecessary. And you can’t just cast a nested column to a simple one; you must apply an aggregate or some other operation that flattens a list.

Observe that [`ungroup`](https://code.kx.com/q/ref/ungroup/) can be used to reverse the nested result of grouping without aggregation. It is not quite an inverse since it returns the original records ordered on the by column(s).
    
    
```q
q)ungroup select c2 by c1 from t
c1 c2
-----
a  10
a  30
b  20
b  40
c  50
```
There are use cases that group on specified column(s) and want **all** the remaining columns to be nested in the result. For this use [`xgroup`](https://code.kx.com/q/ref/xgroup/), which takes the symbolic column name(s) to be grouped as left operand and a table as right operand. The result is a keyed table that is that same as listing all the non-grouped columns in the equivalent select.

Using the distribution example,
    
    
```q
q)`p xgroup sp
p | s               qty            
--| -------------------------------
p1| `s$`s1`s2       300 300        
p2| `s$`s1`s2`s3`s4 200 400 200 200
p3| `s$,`s1         ,400           
p4| `s$`s1`s4       200 300        
p5| `s$`s4`s1       100 400        
p6| `s$,`s1         ,100 
```
Again `ungroup` is an inverse up to record order.
    
    
```q
q)ungroup `p xgroup sp
_
```
### 9.3.4.3 Grouping with Aggregation¶

Normally you will group using `by` together with aggregation in the Select phrase. The effect is to aggregate along the groups, collapsing each group of rows into a single record. The result is a keyed table whose key columns are the grouped column(s).
    
    
```q
q)select sum c2 by c1 from t
c1| c2
--| --
a | 40
b | 60
c | 50
```
To group on multiple columns, specify multiple by subphrases, which results in a compound key in the result.
    
    
```q
q)t:([] desk:`a`b`a`b`a`b; acct:`1`2`3`4`1`4; pnl:1.1 -2.2 3.3 4.4 5.5 -.5)
q)select ct:count desk, sum pnl by desk,acct from t
desk acct| ct pnl 
---------| -------
a    1   | 2  6.6 
a    3   | 1  3.3 
b    2   | 1  -2.2
b    4   | 2  3.9
```
Tip

In contrast to SQL, every column in the By phrase is automatically included in the key column(s) of the result and should not be duplicated in the Select phrase.

A By subphrase can be a q expression, meaning that you can group on computed columns. This is very powerful and is not present in SQL. Following is a useful example that averages the observations of our time series in 100 millisecond buckets.
    
    
```q
q)t:([] c1:00:00:00.000+til 1000000;c2:1000000?`a`b;c3:1000000?100.)
q)select avg c3 by 100 xbar c1, c2 from t
c1           c2| c3      
---------------| --------
00:00:00.000 a | 55.26494
00:00:00.000 b | 41.81758
00:00:00.100 a | 48.88826
00:00:00.100 b | 46.10946
```
Tip

In an unexpected special case of implicit grouping with aggregation, specifying a By phrase together with empty Select phrase is equivalent to applying `last` to all columns. A moment’s thought reveals this is quite useful for financial time series, where you often want the most recent value of a group.
    
    
```q
 q)t:([] desk:`a`b`a`b`a`b; acct:`1`2`3`4`1`4; pnl:1.1 -2.2 3.3 4.4 5.5 -.5)
 q)select by desk from t
 desk| acct pnl
 ----| ---------
 a   | 1    5.5
 b   | 4    -0.5 
```
## 9.4 The `exec` Template¶

The syntax of the [`exec`](https://code.kx.com/q/ref/exec/) template is identical to that of `select`.

`exec` <_p s_> <`by` _p b_> `from` _t exp_ <`where` _p w_>

Whereas `select` always returns a table, the result type of `exec` depends on the number of columns in its Select phrase. One column yields a list; more than one column yields a dictionary.

When more than one column is specified the Select phrase, the result is a dictionary mapping column names to the column lists produced. The essential difference from `select` is that the column lists do not have to be rectangular – i.e., they are not required to have the same length – and the resulting dictionary is not flipped into a table. For example, the following query fails with `select` because the proposed column dictionary is not rectangular but it succeeds with exec.
    
    
```q
q)t:([] name:`a`b`c`d`e; state:`NY`FL`OH`NY`HI)
q)select name, distinct state from t
'length
q)exec name, distinct state from t
name | `a`b`c`d`e
state| `NY`FL`OH`HI
```
A common use of `exec` is when there is only one column in the aggregate phrase and no By phrase. The result is the computed column list devoid of other structure – i.e., not a dictionary or table. This is useful to extract a column dynamically. With t as above,
    
    
```q
q)select name from t
name
----
a
b
c
d
e
q)exec name from t
`a`b`c`d`e
```
When using `exec` to extract a single column of a table, you can place constraints on other columns.
    
    
```q
q)exec name from t where state in `NY`HI
`a`d`e
```
## 9.5 The `update` Template¶

### 9.5.1 Basic update¶

The `update` template has identical syntax to `select`.

`update` <_p u_> <`by` _p b_> `from` _t exp_ <`where` _p w_>

The semantic difference is that colons in the Update phrase _p u_ identify modified or new columns instead of simply assigning column names. If the left of a colon is a column that exists in the table, that column is updated with the result of the expression to the right of the colon. If the left of a colon is not a column in the table, a new column of that name with the result of the expression is added to the end of the column list. The original table is not affected.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)update c1:`x`y`z from t
c1 c2
-----
x  10
y  20
z  30
q)t
_
q)update c3:`x`y`z from t
c1 c2 c3
--------
a  10 x
b  20 y
c  30 z
q)t
_
```
The implicit naming conventions are the same as in `select` so you can omit the name and colon for existing columns in many cases. With `t` as above,
    
    
```q
q)(update c2:c2+100 from t)~update c2+100 from t
1b
```
In the examples above, the table was passed by value and so the original was not modified. To modify the table in place, pass it by name.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)update c1:`x`y`z from `t
_
q)t
_
q)update c3:`x`y`z from `t
_
q)t
_
```
Qbies coming from DDL may not immediately appreciate how useful it is that `update` can add new columns dynamically. For example, you can add a “constant” column by taking advantage of the fact that scalars are extended to match vectors. This is **not** a default value for the column in subsequent records.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)update c3:42 from t
_
```
Often you will apply `update` with a Where phrase that limits the scope of the modification.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)update c2:c2+100 from t where c1<>`a
c1 c2
------
a  10
b  120
c  130
```
Tip

The actions in the Where phrase and the Update phrase are vector operations on entire column lists. This is the Zen of `update`.

If you add a new column in `update` with a Where phrase, the fields in the non-selected rows will have the null value of the appropriate type.
    
    
```q
q)update c3:1b from t where c2>15
c1 c2 c3
--------
a  10 0
b  20 1
c  30 1
```
You can use any q expression to the right of the colon providing the size and type match the targeted location.
    
    
```q
q)update c2:42 43 from t where c2>15
_
q)update c2:42 43 44 from t where c2>15
'length
q)update c2:42.0 43 from t where c2>15
'type
```
### 9.5.2 update-by¶

When the By phrase is present, the update operation is performed along groups. This is most useful with aggregate and uniform functions. For an aggregate function, the entire group gets the value of the aggregation on the group.
    
    
```q
q)update avg weight by city from p
p | name  color weight city  
--| -------------------------
p1| nut   red   15     london
p2| bolt  green 14.5   paris 
p3| screw blue  17     rome  
p4| screw red   15     london
p5| cam   blue  14.5   paris 
p6| cog   red   15     london
```
A uniform function is applied along the group in place. This can be used to compute cumulative volume of orders, for example.
    
    
```q
q)update cumqty:sums qty by s from sp
_
```
## 9.6 The `delete` Template¶

The final template, `delete`, allows either rows or columns to be deleted. Its syntax is a simplified form of `select`, with the restriction that either _p cols_ or _p w_ can be present but not both.

`delete` <_p cols_> `from` _t exp_ <`where` _p w_>

If _p cols_ is present as a comma-separated list of columns, the result is _t exp_ with the specified columns removed. If _p w_ is present, the result is _t exp_ after records meeting the criteria of _p w_ are removed. Someone always asks, can you delete rows and column simultaneously? But the rest of us meditating on the Zen of q realize this makes no sense.

When the table is passed by value, the operation is on a copy. When the table is passed by name, the operation is in place. Deleting from a copy,
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)delete c1 from t
_
q)delete from t where c2>15
_
q)t
_
```
To delete in place,
    
    
```q
q)delete from `t where c2=30
_
q)delete c2 from `t
_
q)t
_
```
Tip

When you want to select all but a few columns, it is easier to delete the ones you don’t want than list all the ones you do.
    
    
```q
q)t:([] c1:1 2; c2:`a`b; c3:1.1 2.2; c4:2015.01.01 2015.01.02)
q)(select c1, c2, c4 from t)~delete c3 from t
1b
```
## 9.7 Sorting¶

Recall that tables and keyed tables comprise lists of records and therefore have an inherent order. A table or keyed table can be reordered by sorting on any column(s). In contrast to SQL, there is no equivalent to `ORDER BY` in the select template. Instead, built-in functions that sort tables are applied after `select`.

### 9.7.1 `xasc`¶

The binary [`xasc`](https://code.kx.com/q/ref/xasc/) takes a scalar, or list of, symbolic column name(s) as its left operand and a table or table name as its right operand. It returns the records of the table sorted ascending on the specified column(s). The sort column list is specified in major-to-minor sort order.

To work on a copy, pass by value.
    
    
```q
q)t:([] c1:`a`b`c`a; c2:20 10 40 30)
q)`c2 xasc t
c1 c2
-----
b  10
a  20
a  30
c  40
q)`c1`c2 xasc t
c1 c2
-----
a  20
a  30
b  10
c  40
q)t
_
```
To sort in place, pass by name.
    
    
```q
q)`c1`c2 xasc `t
_
q)t
_
```
### 9.7.2 `xdesc`¶

The binary [`xdesc`](https://code.kx.com/q/ref/xdesc/) behaves exactly as `xasc`, except that the sort is performed in descending order. 
    
    
```q
q)t:([] c1:`a`b`c`a; c2:20 10 40 30)
q)`c1`c2 xdesc t
c1 c2
-----
c  40
b  10
a  30
a  20
```
### 9.7.3 Mixed Sort¶

We point out that `xasc` and `xdesc` are stable sorts, meaning that the order of two records having the same sort key value is preserved in the result. This makes it possible to compose ascending and descending sort to obtain mixed sorts. For example, to sort `c2` descending within `c1` ascending in `t` above,
    
    
```q
q)`c1 xasc `c2 xdesc t
_
```
## 9.8 Renaming and Rearranging Columns¶

Since a table is the flip of a column dictionary, its columns are both named and ordered. There are built-in primitives to rename and reorder columns. Their names are unfortunately chosen and their usage may seem awkward at first since they focus on the left-most columns. Nonetheless, they work well in practice.

We use the following table in this section.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
```
### 9.8.1 `xcol`¶

The binary [`xcol`](https://code.kx.com/q/ref/xcol/) takes a scalar, or list of, symbolic column name(s) as its left operand (_names_) and a table or keyed table (_source_) as its right operand. The columns in _names_ must appear in _source_. The result is a table obtained by renaming the left-most columns of source according to _names_.
    
    
```q
q)`new1`new2 xcol t
_
q)t
_
```
There is no pass-by-name version of `xcol`. To modify the source, reassign it.

Tip

You can use constructs such as the following to rename isolated columns if your table has many columns and the targeted columns area not left-most.
    
    
```q
q)@[cols[t]; where cols[t]=`c2; :; `new2] xcol t
_
q)@[cols[t]; where cols[t] in `c1`c3; :; `new1`new3] xcol t
_ 
```
### 9.8.2 `xcols`¶

Now you will see what we mean about the unfortunate naming convention. The binary [`xcols`](https://code.kx.com/q/ref/xcols/) takes a scalar, or list of, symbolic column name(s) as its left argument (_names_) and a table (_source_) as its right argument. The output is the result of reordering the columns in _names_ so that those in _names_ occur at the beginning – i.e., left-most in the display. Columns not specified in _names_ are left in original order at the end – i.e., right-most.

Notes:

  * The _source_ operand **cannot** be a keyed table.
  * There is no pass-by-name version of `xcols`. To modify the source, you must reassign it.



For example,
    
    
```q
q)`c3 xcols t
_
q)`c3`c2 xcols t
_
q)t
_
```
## 9.9 Joins¶

The essence of relational database design is normalizing data using relations and keys and then reassembling with joins. Normalization eliminates duplicate data, which takes up space and is hard to keep consistent. Joins restore the original flat rectangular form that makes data easy to work with (there’s a reason spreadsheets are so popular). Simple use cases include a master-detail relation or a lookup table.

In SQL the relational structure with primary and foreign keys is static. It must be defined in a separate language (DDL) before the data can be used. Foreign key/primary key relations must be pre-established in order for joins to take place.

In q, tables are first-class entities in the language. You can define tables and relations statically, but it is easy to create them dynamically. It is even possible to join tables that could have a relation but don’t.

A join sews back together along a foreign key/primary linkage data that has been factored into normal form. A join can be classified as inner or outer. An _inner join_ pairs **only** records in both operands having matching keys. A _left outer_ Join includes **all** records in the left operand, pairing them with records having matching key in the right operand, should such exist. A _right outer_ Join reverses left and right in this description.

In q there are built-in inner and left outer joins; if you need a right join, reverse the operands. There is no operator for a full outer join but you can construct one; be careful what you wish for with large tables.

Joins can also be classified by how key matching is determined. Most joins are _equijoins_ , meaning that the keys must be equal. In q there are also non-equijoins, called _as of_ joins, in which a key is tested for less-than-or-equal against keys in another table.

### 9.9.1 Implicit Join¶

Given a primary key table _m_ with key column(s) _k_ and a table _d_ with a foreign key linking to _k_ , a left join can be expressed in various SQL notations. For example,
    
    
```q
m LEFT JOIN d ON m.k = d.k
```
A `SELECT` statement for this join refers to columns in the join by using dot notation based on the constituent tables.

`SELECT` d.cold, m.colm `FROM` m `LEFT JOIN` d `WHERE` m.k `=` d.k

As we saw in [Chapter 8](../8_Tables/), a foreign key in q is accomplished with an enumeration over the key column(s) of a keyed table. A left join is implicit in the following query on the detail table.

`select col`d, `k.col`m `from d`

For example, in the `sp.q` distribution script, the table `sp` has foreign keys to both `s` and `p`. We can extract columns from the left join with `s` by issuing a query against `sp` and using dot notation on the foreign key to get columns in `s`.
    
    
```q
q)select sname:s.name, qty from sp
_
```
This generalizes to the situation where `d` has multiple foreign keys. For example, in the `sp.q` distribution script, we can select records from the join of `sp`, `s` and `p` from a query against `sp`.
    
    
```q
q)select sname:s.name, pname:p.name, qty from sp
_
```
Implicit joins extend to the situation in which the targeted keyed table itself has a foreign key to another keyed table.
    
    
```q
q)emaster:([eid:1001 1002 1003 1004 1005] currency:`gbp`eur`eur`gbp`eur)
q)update eid:`emaster$1001 1002 1005 1004 1003 from `s
q)select s.name, qty, s.eid.currency from sp
_
```
### 9.9.2 Ad hoc Left Join (`lj`)¶

To create an ad-hoc left outer join between tables that **could** have a foreign-key relationship, use the binary `lj`. When the foreign key exists, that linkage is used; otherwise, the linkage is constructed dynamically. The join is 2-3 times faster if the foreign key already exists.

If you intend to perform the join more than a few times, it pays to create the foreign key up front.

The right operand is a keyed table (_target_) and the left operand is a table or keyed table (_source_) having either a foreign key to target or column(s) that match the key column(s) of target in name and type. The result is all the records and columns of _source_ augmented with the records and columns of _target_ along matching keys. For those records in _source_ having no matching key, the augmented columns contain null values.

In the following example we see all the records of `t` in the result, with null values in the `kt` column(s) where there is no matching key.
    
    
```q
q)t:([] k:1 2 3 4; c:10 20 30 40)
q)kt:([k:2 3 4 5]; v:200 300 400 500)
q)t lj kt
k c  v
--------
1 10
2 20 200
3 30 300
4 40 400
```
Observe that when the source table has a foreign key, an ad-hoc left join is equivalent to listing all columns from both tables in an implicit join.
    
    
```q
q)kt:([k:1 2 3 4 5]; v1:10 20 30 40 50; v2:1.1 2.2 3.3 4.4 5.5)
q)tf:([] k:`kt$1 2 3 4; c:10 20 30 40)
q)(tf lj kt)~select k,c,k.v1,k.v2 from tf
1b
```
When the tables source and target have duplicate non-key columns, the operation has `upsert` semantics.

That is, the values in the right operand (target) columns prevail over those in the left operand (source). This is different from SQL where the result contains both columns with suffixes to ensure unique names.
    
    
```q
q)t:([] k:1 2 3 4; v:10 20 30 40)
q)kt:([k:2 3 4 5]; v:200 300 400 500)
q)t lj kt
k v
-----
1 10
2 200
3 300
4 400
```
You can also use `lj` with a left operand keyed table.
    
    
```q
q)kt1:([k:1 2 3 4]; v:10 0N 30 40)
q)kt:([k:2 3 4 5]; v:200 300 400 500)
q)kt1 lj kt
_
```
### 9.9.3 Column Lookup¶

You can perform a column lookup against a keyed table within a query without using a join. The insight is that a keyed table is a dictionary whose key list comprises its key records, so it will perform the lookup provided we put the column in an anonymous table conforming to those key records – see [§8.4.5](../8_Tables/#845-retrieving-multiple-records). Here we demonstrate the case where the lookup column names do not natively match the key columns, so we rename columns to match in the anonymous table.
    
    
```q
q)t:([] k:1 2 3 4; c:10 20 30 40)
q)kt:([k1:2 3 4 5]; v:2.2 3.3 4.4 5.5)
q)select c, v:kt[([] k1:k); `v] from t
c  v
------
10
20 2.2
30 3.3
40 4.4
```
Especially for a single column, this is simpler (and more impressive to your colleagues) than the equivalent join.
    
    
```q
q)select c,v from t lj `k xkey select k:k1,v from kt
c  v
------
10
20 2.2
30 3.3
40 4.4
```
Here is an example using compound keys and column renaming to match the lookup table.
    
    
```q
q)t:([] f:`rocky`bullwinkle; l:`squirrel`moose; c:10 20)
q)kt:([fn:`rocky`bullwinkle`fearless; ln:`squirrel`moose`leader] v:1.1 2.2 3.3)
q)select c, v:kt[([] fn:f; ln:l); `v] from t
c  v
------
10 1.1
20 2.2
```
### 9.9.4 Ad Hoc Inner Join (`ij`)¶

Given a primary key table _m_ with key column(s) _k_ and a table _d_ with a foreign key linking to _k_ , an inner join can be expressed in various SQL notations,
    
    
```q
m,d WHERE m.k = d.k

m INNER JOIN d ON m.k = d.k
```
A `SELECT` statement for this join refers to columns in the join by using dot notation based on the constituent tables.

`SELECT` d.cold, m.colm `FROM` m,d `WHERE` m.k `=` d.k

The binary _inner-join_ operator `ij` performs an inner join between two tables that **could** have a foreign key relationship.

The same performance observations hold as for `lj`.

As with `lj`, the right operand is a keyed table (_target_) and the left operand is a table or keyed table (_source_) having column(s) that are either foreign key(s) over _target_ or exactly match the key column(s) of _target_ in name and type. The matching is done via the foreign key or by common column name(s) between _source_ and the key column(s) of _target_ if there is no foreign-key relationship**.** The result contains the columns from source and target joined along common keys.

The following example shows that `ij` returns just those records with matching keys in both tables.
    
    
```q
q)t:([] k:1 2 3 4; c:10 20 30 40)
q)kt:([k:2 3 4 5]; v:2.2 3.3 4.4 5.5)
q)t ij kt
k c  v
--------
2 20 2.2
3 30 3.3
4 40 4.4
```
As with `lj`, upsert semantics holds for duplicate columns.
    
    
```q
q)t:([] k:1 2 3 4; v:10 20 30 40)
q)kt:([k:2 3 4 5]; v:200 300 400 500)
q)t ij kt
k v
-----
2 200
3 300
4 400
```
You can also use `ij` with a left operand keyed table.
    
    
```q
q)kt1:([k:1 2 3 4]; v:10 0N 30 40)
q)kt:([k:2 3 4 5]; v:200 300 400 500)
q)kt1 ij kt
_
```
### 9.9.5 Equijoin `ej`¶

The triadic _equijoin_ operator `ej` corresponds to a SQL inner join between tables in the second and third parameters along specified column names in the first parameter. The right operand does not have to be a keyed table. Unlike `ij`, **all** matching records in the right table appear in the result. As with any join, upsert semantics holds on duplicate columns.
    
    
```q
q)t1:([] k:1 2 3 4; c:10 20 30 40)
q)t2:([] k:2 2 3 4 5; c:200 222 300 400 500; v:2.2 22.22 3.3 4.4 5.5)
q)t1 ij `k xkey t2
k c   v
---------
2 200 2.2
3 300 3.3
4 400 4.4
q)ej[`k;t1;t2]
k c   v
-----------
2 200 2.2
2 222 22.22
3 300 3.3
4 400 4.4
```
### 9.9.6 Plus Join (`pj`)¶

_Plus join_ `pj` is a left join that replaces upsert semantics for duplicate column names with addition. This is useful when you have two tables with identical schemas having all non-key columns numeric. For example, you have an organizational hierarchy and you want to roll up numeric results.

The operands of `pj` are the same as for `lj` with the additional requirement that all non-key columns are numeric. The semantics are that duplicate columns are added along matching keys and missing or null values are treated as zero.

For example,
    
    
```q
q)t:([] k:`a`b`c; a:100 200 300; b:10. 20. 30.; c:1 2 3)
q)kt:([k:`a`b] a:10 20; b:1.1 2.2)
q)t pj kt
k a   b    c
------------
a 110 11.1 1
b 220 22.2 2
c 300 30 3
```
Observe that this is the same as,
    
    
```q
q)kt1:([k:`a`b`c] a:10 20 0; b:1.1 2.2 0.0; c:0 0 0)
q)t pj kt1
k a   b    c
------------
a 110 11.1 1
b 220 22.2 2
c 300 30 3
```
### 9.9.7 Union Join¶

The equivalent of an ordinary SQL union on tables with matching schemas is simply `,`. Indeed, it joins two lists of compatible records.
    
    
```q
q)t1:([] c1:`a`b; c2:1 2)
q)t2:([] c1:`c`d; c2:3 4)
q)t1,t2
c1 c2
-----
a  1
b  2
c  3
d  4
```
_Union join_ [`uj`](https://code.kx.com/q/ref/uj/) is more powerful in that it vertically combines **any** two tables, or keyed tables. The records of the right operand are appended to those of the left operand in the following manner. The result table is widened with new columns of the same name and type for the columns of the right operand that do not appear in the left operand. The initial records in the result come from the left operand and have nulls in the new columns. Records from the right operand have their field values under the appropriate columns. An example is worth a thousand words.
    
    
```q
q)t1:([] c1:`a`b`c; c2: 10 20 30)
q)t2:([] c1:`x`y; c3:8.8 9.9)
q)t1 uj t2
c1 c2 c3 
---------
a  10    
b  20    
c  30    
x     8.8
y     9.9
```
Continuing the previous example, you can use `uj/` to combine a list of disparate tables.
    
    
```q
q)t3:([] c1:`e`f`g; c2:50 60 70; c3:5.5 6.6 7.7)
q)(uj/) (t1; t2; t3)
_
```
You can also apply `uj` to keyed tables, where upsert semantics hold on both rows and columns.
    
    
```q
q)kt1:([k:1 2 3] v1:`a`b`c; v2:10 20 30)
q)kt2:([k:3 4] v2:300 400; v3:3.3 4.4)
q)kt1 uj kt2
k| v1 v2  v3 
-| ----------
1| a  10     
2| b  20     
3| c  300 3.3
4|    400 4.4
```
Tip

The operation of `uj` is expensive so it should only be used when necessary; in particular, when the tables conform, use `,` or `raze`. Be especially cautious when low latency is paramount.

### 9.9.8 As-of Joins¶

As-of joins are so-named because they most often join tables along time columns to obtain a value in one table that is current as of a time in another table. As-of joins are non-equijoins that match on less-than-or-equal. They are not restricted to time values.

The fundamental as-of join is the triadic function [`aj`](https://code.kx.com/q/ref/aj/). It joins tables along common columns using most recent values. The syntax of `aj` is,

`aj`[`_c 1_...`_c n_;_t 1_;_t 2_]

where _`c 1...`cn_ is a simple list of symbol column names common to _t 1_ and _t 2_, which are the tables to be joined. There is no requirement for any of the join columns to be keys but the join will be faster on keys. The columns of both tables are brought into the result.

The semantics of `aj` are as follows. The match on all specified columns **except the last** is by equals. Assuming the records are sequenced by _c n_ in both tables, for a given _c n_ value in _t 1_, the match picks the greatest _c n_ in _t 2_ less than or equal to the given value in _t 1_. Specifically, if the _c n_ columns are sequenced temporal values, for each _c n_ value in _t 1_ the match picks the _t 2_ row whose _c n_ value is in effect “as of” the time in _t 1_.

The canonical example for `aj` is matching trades with quotes. To know if a trade represents best execution, you want to compare the trade price to the current quote – i.e., the most recent quote up to and including the time of the trade. When matching the trades and quotes, you clearly want an equijoin on the date and symbol and a non-equijoin on the time.
    
    
```q
q)show t:([] ti:10:01:01 10:01:03 10:01:04;sym:`msft`ibm`ge;qty:100 200 150)
ti       sym  qty
-----------------
10:01:01 msft 100
10:01:03 ibm  200
10:01:04 ge   150
q)show q:([] ti:10:01:00 10:01:01 10:01:01 10:01:02;sym:`ibm`msft`msft`ibm;px:100 99 101 98)
ti       sym  px 
-----------------
10:01:00 ibm  100
10:01:01 msft 99 
10:01:01 msft 101
10:01:02 ibm  98
q)aj[`sym`ti;t;q]
ti       sym  qty px 
---------------------
10:01:01 msft 100 101
10:01:03 ibm  200 98 
10:01:04 ge   150
```
It is worth stepping through this example in detail. We begin with the first record in `t`. We look for the quote records matching ``msft` as of the trade time 10:01:01. There are two such records, both stamped 10:01:01 but the last one has price 101, which is reflected in the result. Next we look for the last quote record for ``ibm` as of 10:01.02, which is the quote record stamped 10:00:02 with price 98. Finally there are no quote records matching ``ge` at 10:01:04.

If you want the time of the matching quote in the result instead of the time of the trade, use [`aj0`](https://code.kx.com/q/ref/aj0/).
    
    
```q
q)aj0[`sym`ti;t;q]
ti       sym  qty px 
---------------------
10:01:01 msft 100 101
10:01:02 ibm  200 98 
10:01:04 ge   150 
```
The simpler function [`asof`](https://code.kx.com/q/ref/asof/) performs the same match as `aj` but with a table against a single record. The result picks out the remaining columns in the matched row of the table.
    
    
```q
q)t:([] ti:10:01:01 10:01:03 10:01:04; sym:`msft`ibm`ge; qty:100 200 150; px:45 160 55)
q)t
_
q)t asof `sym`ti!(`ibm;10:01:03)
qty| 200
px | 160
```
A list of such dictionary records conforms – i.e., is a table – and `asof` matches against each record.
    
    
```q
q)t asof ([] sym:`msft`ibm; ti:10:01:01 10:01:03)
qty px
-------
100 45
200 160
```
As-of joins are also useful with non-numeric data. Suppose we have a table containing a history of state changes – e.g., employee promotions. An as-of join will determine the state “as of” any point in time.
    
    
```q
q)promotions:([] name:`nuba`devi`nuba`devi`nuba;
    dt:2000.01.01 2005.02.14 2010.02.01 2012.08.12 2015.11.01;
    title:`associate`analyst`director`cfo`retired)
q)promotions asof `name`dt!(`nuba; 2009.07.04)
title| associate
q)promotions asof `name`dt!(`devi; 2015.12.01)
_
q)events:([] name:`nuba`devi; dt: 2009.07.04 2015.12.01)
q)aj[`name`dt; events; promotions]
_
```
### 9.9.9 Window Join¶

Window joins are a generalization of as-of joins and are specifically geared for analyzing the relationship between trades and quotes in finance. The idea is that you want to investigate the behavior of quotes in a neighborhood of each trade. For example, to determine how well a trade was executed, you need to examine the range of bid and ask prices that were prevalent around the trade time.

Writing such a query manually would be cumbersome. Instead, q provides the built-in window join [`wj`](https://code.kx.com/q/ref/wj/) that computes on an interval around each trade. Specifying the set up takes a bit of work but then all the hard work is done for you.

So that things fit on the page, we use a simple example involving trades for one ticker symbol ``aapl` and times that are in seconds. The general case is essentially the same. First we create the trades and quotes tables; note that they are required to be unkeyed.
    
    
```q
q)show t:([]sym:3#`aapl;time:09:30:01 09:30:04 09:30:08;price:100 103 101)
_
q)show q:([] sym:8#`aapl;
    time:09:30:01+(til 5),7 8 9;
    ask:101 103 103 104 104 103 102 100;
    bid:98 99 102 103 103 100 100 99)
_
```
We construct fixed-width windows of 2 seconds before and one second after each trade time. (There is no requirement that the windows be of uniform width.)
    
    
```q
q)w:-2 1+\:t `time
q)w
_
```
For readability we place the names of the sym and time columns in a variable.
    
    
```q
q)c:`sym`time
```
The actual form for `wj` is:
    
    
```q
wj[w;c;t;(q;(f0;c0);(f1;c1))]
```
Here `w` is a list of windows, `c` is the list of sym and time column names, `t` is the trades table. The final parameter is a list with the quotes table and a pair of lists, each with an aggregate function and a column name.

To see all the values in each window, pass the identity function `::` in place of the aggregates. The result is similar to grouping without aggregate in a query and is helpful to see what is happening within each window.
    
    
```q
q)wj[w;c;t;(q;(::;`ask);(::;`bid))]
sym  time     price ask             bid           
--------------------------------------------------
aapl 09:30:01 100   101 103         98 99         
aapl 09:30:04 103   103 103 104 104 99 102 103 103
aapl 09:30:08 101   104 103 102     103 100 100
```
Now we apply `wj` with actual aggregates for the maximum ask and minimum bid over each window. Here you see the results of the aggregate functions run within each window.
    
    
```q
q)wj[w;c;t;(q;(max;`ask);(min;`bid))]
sym  time     price ask bid
---------------------------
aapl 09:30:01 100   103 98 
aapl 09:30:04 103   104 99 
aapl 09:30:08 101   104 100
```
As opposed to the aggregates being run over only the quote values within each window, you can also include the quotes that are current at the beginning of the window. This matters if there is no quote precisely at the beginning time of the window. For this case use [`wj1`](https://code.kx.com/q/ref/wj1/), whose syntax is identical to `wj`.
    
    
```q
q)wj1[w;c;t;(q;(::;`ask);(::;`bid))]
sym  time     price ask             bid           
--------------------------------------------------
aapl 09:30:01 100   101 103         98 99         
aapl 09:30:04 103   103 103 104 104 99 102 103 103
aapl 09:30:08 101   103 102         100 100
```
## 9.10 Parameterized Queries¶

Relational databases have stored procedures, which are database programs incorporating SQL statements. The programming languages are not part of the SQL standard. They vary significantly across vendors but are generally third-generation imperative languages.

This situation forces a programmer to make a choice: learn a proprietary language to place functionality close to the data, or extract the data into an application server to perform calculations. Various multi-tier architectures have evolved to address this problem, but they increase system cost and complexity.

This separation is obviated in kdb+, since q is the stored procedure language and it has the power and performance to process big data. Any q function that operates on a table is effectively a stored procedure. Function parameters can be used to supply specific values for queries.

In particular, the `select`, `exec`, `update` and `delete` templates can be invoked within a function with parameters to yield a _parameterized query_.

Restrictions

Parameterized queries with templates have restrictions:

  * A column cannot be passed as a parameter since columns are part of the syntactic sugar of the template. They only become symbols under the covers.
  * Multiple parameters cannot be implicit – i.e., they must be declared explicitly. Although this seems to work for a single parameter, we recommend making **all** parameters explicit.



We follow our own advice in the following examples.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
q)select from t where c2>15
_
q)proc:{[sc] select from t where c2>sc}
q)proc 15
_
q)proc2:{[nms;sc] select from t where c1 in nms, c2>sc}
q)proc2[`a`c; 15]
_
```
You **can** pass a table as a parameter to a template, either by value or by name.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
q)proc3:{[t;nms;delta] update c2+delta from t where c1 in nms}
q)proc3[t;`a`c;100]
_
q)t
_
q)proc3[`t;`a`c;100]
_
q)t
_
```
Tip

There is no need to restrict stored procs in q to the templates. Any expression that operates on tables can serve.
    
    
```q
 q)t:([] c1:`a`b`c; c2:10 20 30; c3:1.1 2.2 3.3)
 q)procf:{[cs] cs#t}
 q)procf `c1`c2
 _ 
```
You can effectively parameterize column names in two ways, only one of which is good practice. First, you can mimic a common technique from SQL in which the query is created dynamically: build the query text in a string and then invoke the interpreter programmatically using `value`. This is comparatively slow. Worse, it exposes your application to injection attacks, since any q expression that appears inside the text will be executed.

The preferred method is to use the functional form for queries – see §9.12 – which is fast and secure. In functional form, all columns are referred to by symbolic name, so column names can be passed as symbols. In fact, **any** component of a query can be passed as an argument in functional form.

## 9.11 Views¶

A SQL view is effectively a query expression whose result set can be used like a table. Views are useful for encapsulating data – e.g., hiding columns, or simplifying complex queries.

A q-sql _view_ is a named table expression created as an alias with the double-colon operator `::`. It is common to use the templates in views but this is not a limitation.

In the following example, contrast `u`, which is a q variable that is assigned the result of a query, to the view (alias) `v` that is the query expression itself.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)u:select from t where c2>15
q)v::select from t where c2>15
q)u
_
q)v
_
q)update c2:15 from `t where c1=`b
`t
q)u
c1 c2
_
q)v
c1 c2
-----
c  30
```
Observe that when the underlying table changed, `u` did not change but the next reference to `v` does reflect the update.

To find the underlying query of a view, or any alias, apply the function [`view`](https://code.kx.com/q/ref/view/) to the symbol alias name.
    
    
```q
q)view `v
"select from t where c2>15"
q)a:42
q)b::a
q)view `b
_
```
To list all the views in a namespace, use the function [`views`](https://code.kx.com/q/ref/views/) with the context name. For example, to list the views in the root,
    
    
```q
q)views `.
`b`v
```
## 9.12 Functional Forms¶

In the experience of the author, functional form is the most difficult q topic for most qbies. Among the reasons for this are:

  * The high level of abstraction and generalization
  * There is nothing like it in SQL or other programming languages
  * In vintage Arthurian fashion, information density is maximized



Full disclosure: functional form is difficult. Along with iterators and generalized application, it completes the Big Three aspects of q that separate q pretenders from contenders. Fortunately there is a cheat that can be helpful in most situations.

The function [`parse`](https://code.kx.com/q/ref/parse/) can be applied to a string containing a template query to produce a parse tree whose items (nearly) work in the equivalent functional form. A complication is that the operators are displayed in k form instead of q.

Tip

The constraint portion of the result of `parse` applied to a string containing a q-sql query template generally has an extra level of `enlist` that should be removed for the corresponding functional form. The console display of the parse tree shows the k form of `enlist` – i.e., `,` – so you will actually see `,,`.

The functional forms of the four templates `select`, `exec`, `update` and `delete` are powerful because they allow all constituents to be parameterized. They can be used for any query but are especially handy for queries that are generated or completed programmatically. The q interpreter parses the syntactic sugar of the templates into their equivalent functional forms, so there is no performance difference.

Tip

We recommend writing table operations in fundamental or template form unless you anticipate needing to parameterize column names. The folks who assume your code will thank you.

There are two functional forms, one for `select` and `exec`, the other for `update` and `delete`. The types of the arguments passed determine the overload. The forms are,
    
    
```q
?[t;c;b;a] / select and exec

![t;c;b;a] / update and delete
```
where

  * `t` is a table or the name of a table
  * `a` is a dictionary of aggregates
  * `b` is a dictionary of groupbys or a flag controlling other aspects of the query
  * `c` is a list of constraints.



The expressions in `a`, `b` and `c` can involve columns of `t` and also any variables that are in scope. The rules for expression interpretation are:

  * Columns are always represented by their symbolic names
  * Consequently, any literal symbols, or lists of symbols, appearing in the expressions must be distinguished. This is done by enlisting them. Really.



We warned you this wasn’t going to be easy.

We shall use the following sample tables in our examples.
    
    
```q
q)t:([] c1:`a`b`a`c`a`b`c; c2:10*1+til 7; c3:1.1*1+til 7)
q)ft:{([] c1:`a`b`a`c`a`b`c; c2:10*1+til 7; c3:1.1*1+til 7)}
```
### 9.12.1 Functional select¶

We begin with the functional form corresponding to the simplest select query, which uses (yet another) [overload of `?`](https://code.kx.com/q/basics/funsql/#select).
    
    
```q
q)select from t
_
q)?[t; (); 0b; ()]
_
q)?[t; (); 0b; ()]~select from t
1b
```
In this degenerate functional form,

  * The constraint parameter is the empty list
  * The by parameter is the special value `0b` indicating there is no grouping in a select query
  * The aggregate parameter takes the special value of the general empty list, indicating there is no aggregate phrase



Observe that we can pass the table by name and we can also parameterize the table by substituting any q expression that returns a table or keyed table.
    
    
```q
q)?[`t; (); 0b; ()]
_
q)?[ft[]; (); 0b; ()]
_
```
Now we add a constraint – i.e., a Where phrase. The constraint parameter must be a **list** of q [parse trees](https://code.kx.com/q/basics/parsetrees/), one item for each subphrase in the Where phrase. The parse tree for each subphrase can be obtained by converting the expression fully to prefix form and then transforming each function into a list with it followed by all its arguments. Remember, columns names become symbols and literal symbols get enlisted.
    
    
```q
q)select from t where c2>35,c1 in `b`c
_
q)?[t; ((>;`c2;35); (in;`c1;enlist `b`c)); 0b; ()]
_
```
Next we demonstrate aggregation without grouping. The aggregate parameter is a dictionary whose keys are column names and whose value list is a list of parse trees, one for each subphrase of the aggregate phrase. Again, column names are symbols, and literal symbols get enlisted.

While q will assign default columns names in the templates, you must explicitly provide all columns names in functional form.
    
    
```q
q)select max c2, c2 wavg c3 from t
_
q)?[t; (); 0b; `maxc2`wtavg!((max;`c2); (wavg;`c2;`c3))]
_
```
Next we demonstrate grouping without aggregation. As with the aggregate parameter, we construct a dictionary with column names and parse trees of the By subphrases. In this particular example, we demonstrate how to handle the case of grouping on a single column. Recall that a singleton dictionary requires the key and value to be enlisted – this enlist is separate from enlisting literal symbols in functional form.
    
    
```q
q)select by c1 from t
_
q)?[t; (); (enlist `c1)!enlist `c1; ()]
_
```
Finally we put the pieces together. We find code to be more readable if you separately construct the three parameters as variables and pass these in to the functional form.
    
    
```q
q)select max c2, c2 wavg c3 by c1 from t where c2>35,c1 in `b`c
_
q)c:((>;`c2;35); (in;`c1;enlist `b`c))
q)b:(enlist `c1)!enlist `c1
q)a:`maxc2`wtavg!((max;`c2); (wavg;`c2;`c3))
q)?[t;c;b;a]
_
```
Now that wasn’t so bad, was it? OK, maybe it was.

Next we show the functional form for the special case `select distinct`. In this case set the _by_ parameter to `1b`.
    
    
```q
q)t:([] c1:`a`b`a`c`b`c; c2:1 1 1 2 2 2; c3:10 20 30 40 50 60)
q)select distinct c1,c2 from t
_
q)?[t; (); 1b; `c1`c2!`c1`c2]
_
```
The extended form `select[n]` adds a fifth parameter to the functional form.
    
    
```q
q)t:([] c1:`a`b`c; c2:10 20 30)
q)select[2] from t
_
q)?[t;();0b;();2]
_
```
The extended form `select[>_c i_]` adds two additional parameters to the functional form. The first is the initial value for the comparison and the second is a list with the k form – seriously! – of the comparison operator for the sort along with the column name for the sort.
    
    
```q
q)t:([] c1:`a`b`c`a; c2:10 20 30 40)
q)select[>c1] c1,c2 from t
_
q)?[t;();0b;`c1`c2!`c1`c2; 0W; (>:;`c1)]
_
```
### 9.12.2 Functional exec¶

The functional form for `exec` is nearly identical to that of `select`. Slight variations in the parameters indicate which is intended. Since the constraint parameter carries over unchanged, we omit its discussion in this section.

The functional form for `exec` on a single result column depends on whether you want a list or dictionary to be returned. Assuming there is no grouping, use the empty list for the _by_ parameter. For a list result, specify the aggregate as a parse tree; for a dictionary result, specify a dictionary mapping the result name to its parse tree.
    
    
```q
q)t:([] c1:`a`b`c`a; c2:10 20 30 40; c3:1.1 2.2 3.3 4.4)
q)exec distinct c1 from t
_
q)?[t; (); (); (distinct; `c1)]
_
q)exec c1:distinct c1 from t
_
q)?[t; (); (); (enlist `c1)!enlist (distinct; `c1)]
_
```
For `exec` on multiple columns without grouping, specify the _by_ parameter as the general empty list and the aggregate parameter as a dictionary mapping column names to a list of parse trees. 
    
    
```q
q)exec distinct c1, c2 from t
_
q)?[t; (); (); `c1`c2!((distinct; `c1); `c2)]
_
```
To group on a single column, specify its symbol name in the _by_ parameter.
    
    
```q
q)exec c2 by c1 from t
_
q)?[t; (); `c1; `c2]
_
```
More complex grouping in an exec seems to revert to the equivalent select.

### 9.12.3 Functional update¶

The syntax of functional form of `update` is identical to that of `select` except that `!` is used in place of `?`. In the following examples you will need to keep track of the different uses of `enlist`:

  * making a list of parse trees from a single parse expression
  * creating singleton dictionaries
  * distinguishing literal symbols from column names.


    
    
```q
q)t:([] c1:`a`b`c`a`b; c2:10 20 30 40 50)
q)update c2:100 from t where c1=`a
_
q)c:enlist (=;`c1;enlist `a)
q)b:0b
q)a:(enlist `c2)!enlist 100
q)![t;c;b;a]
_
q)update c2:sums c2 by c1 from t
c1 c2
-----
a 10
b 20
c 30
a 50
b 70
q)![`t; (); (enlist `c1)!enlist `c1; (enlist `c2)!enlist(sums; `c2)]
_
q)t
_
```
Observe that we switched to passing by name in the last functional query.

### 9.12.4 Functional delete¶

The syntax of functional `delete` is a simplified form of functional update.

`![t;c;0b;a]`

where `t` is a table, or the name of a table, `c` is a list of parse trees for Where subphrases and `a` is a **list** of column names. Either `c` or `a`, but not both, must be present. If `c` is present, it specifies which rows are to be deleted. If `a` is present it is a list of symbol column names to be deleted. 

In the former case you must specify `a` as an empty list of symbols.

As before, distinguish the various uses of `enlist` in the examples.
    
    
```q
q)t:([] c1:`a`b`c`a`b; c2:10 20 30 40 50)
q)delete from t where c1=`b
_
q)![t;enlist (=;`c1;enlist `b);0b;`symbol$()]
_
q)delete c2 from t
_
q)![`t;();0b;enlist `c2]
`t
q)t
_
```
Again observe that we switched to call by name in the last query.

## 9.13 Examples¶

At this point we know enough about tables to do some damage. You should go back and (re)read [§1. Q Shock and Awe](../1_Q_Shock_and_Awe/). It will now be straightforward.

### 9.13.1 The trades Table¶

We shall demonstrate further examples in this section based on the `trades` table created there. Here we encapsulate the steps in a function so that it can be run for an arbitrary list of tickers and an arbitrary number of rows. On the author’s two-year-old MacBook Pro, it takes a few seconds to create a table with 10,000,000 rows.
    
    
```q
mktrades:{[tickers; sz]
  dt:2015.01.01+sz?31;
  tm:sz?24:00:00.000;
  sym:sz?tickers;
  qty:10*1+sz?1000;
  px:90.0+(sz?2001)%100;
  t:([] dt; tm; sym; qty; px);
  t:`dt`tm xasc t;
  t:update px:6*px from t where sym=`goog;
  t:update px:2*px from t where sym=`ibm;
  t}
trades:mktrades[`aapl`goog`ibm; 10000000]
```
No multiline expressions in session

You can’t actually define multiline functions in the q session like that. But you can in a script. 

See [§13 Commands and System Variables](../13_Commands_and_System_Variables/) _Ed._

### 9.13.2 The instrument Table¶

The `instr` table is a static keyed table containing basic information about the companies whose financial instruments (stocks in our case) are traded. Its schema has fields for the stock symbol, the name of the company and the industry classification of the company.
    
    
```q
q)instr:([sym:`symbol$()] name:`symbol$(); industry:`symbol$())
```
We populate `instr` using `upsert`.
    
    
```q
q)`instr upsert (`ibm; `$"International Business Machines"; `$"Computer Services")
`instr
q)`instr upsert (`msft; `$"Microsoft"; `$"Software")
`instr
q)`instr upsert (`goog; `$"Google"; `$"Search")
`instr
q)`instr upsert (`aapl; `$"Apple"; `$"Electronics")
`instr
q)instr
_
```
Now we make `sym` a foreign key to trades.
    
    
```q
q)update `instr$sym from `trades
`trades
q)meta trades
c  | t f a
---| ---------
dt | d s
tm | t
sym| s instr
qty| j
px | f
```
### 9.13.3 Basic Queries¶

Here we demonstrate basic operations against the trades and instrument tables.

We can count the total number of trades in several ways. The last two versions cause q to do the most work and should be avoided. Observe that q does not carry the name of its own virtual column `i` to the result of `select`.
    
    
```q
q)count trades
_
q)count select from trades
10000000

q)exec count i from trades
10000000
q)select count i from trades
x
-------
10000000
```
Let’s count the number of trades for an individual symbol.

Tip

For splayed and partitioned tables, only the `select` expression works.
    
    
```q
q)exec count i from trades where sym=`ibm
_
q)select count i from trades where sym=`ibm
_
```
We can count the number of trades across all symbols. The second version unkeys the result.
    
    
```q
q)select count i by sym from trades
sym | x
----| -------
aapl| 3333498
goog| 3330409
ibm | 3336093
q)() xkey select count i by sym from trades
_
```
We find one day's trades for AAPL.
    
    
```q
q)select from trades where dt=2015.01.15,sym=`aapl
_
```
We find all lunch-hour trades for GOOG. Note that `within` includes both endpoints. Also recall that you must use general-list notation if the endpoints are variables.
    
    
```q
q)select from trades where sym=`goog, tm within 12:00:00 13:00:00
_
q)noon:12:00:00
q)thirteen00:13:00:00
q)select from trades where sym=`goog, tm within (noon;thirteen00)
_
```
We find the maximum daily price for AAPL. Due to our uniform distribution, it is statistically constant.
    
    
```q
q)select maxpx:max px by dt from trades where sym=`aapl
dt        | maxpx
----------| -----
2015.01.01| 110
2015.01.02| 110
2015.01.03| 110
2015.01.04| 110
..
```
We find the minimum and maximum trade price over the trading range for each symbol by company name. We point out that this is actually grouping on a computed column since the foreign key is resolved to the instrument table via an implicit join.
    
    
```q
q)select lo:min px, hi:max px by sym.name from trades
name                           | lo  hi
-------------------------------| -------
Apple                          | 90  110
Google                         | 540 660
International Business Machines| 180 220
```
We find the total and average trade volume for specific symbols. Due to our simplistic construction, the volumes are statistically the same.
    
    
```q
q)select totq:sum qty, avgq:avg qty by sym from trades where sym in `ibm`goog
sym | totq        avgq
----| --------------------
goog| 16670706340 5005.603
ibm | 16702382420 5006.57
```
We find the high, low and close over one-minute intervals for Google.
    
    
```q
q)select hi:max px,lo:min px,open:first px, close:last px by dt,tm.minute from trades where sym=`goog
dt         minute| hi     lo     open   close
-----------------| --------------------------
2015.01.01 00:00 | 658.98 540.72 610.98 561.66
2015.01.01 00:01 | 659.4  542.82 594.24 595.86
2015.01.01 00:02 | 652.5  540    625.26 583.5
..
```
We demonstrate how to use our own functions in queries. Suppose we define a funky average that weights items by their position. Notice that we use intra-line assignment to avoid counting `x` three times.
    
    
```q
q)favg:{(sum x*1+til ctx)%ctx*ctx:count x}
```
Then we can apply `favg` just as we did the built-in q function `avg`.
    
    
```q
q)select favgpx:favg px by sym from trades
sym | favgpx
----| --------
aapl| 49.99748
goog| 300.0155
ibm | 100.0009
```
### 9.13.4 Meaty Queries¶

In this section, we demonstrate more interesting q-sql examples against the trades table.

We find volume-weighted average price by day and for 100-millisecond buckets for AAPL. Note that latter takes a few seconds for 10,000,000 trades on the author’s laptop.
    
    
```q
q)select vwap:qty wavg px by dt from trades where sym=`ibm
dt        | vwap
----------| --------
2015.01.01| 200.064
2015.01.02| 200.0207
2015.01.03| 200.0109
..
q)select vwap:qty wavg px by dt,100 xbar tm from trades where sym=`ibm
dt         tm          | vwap
-----------------------| ------
2015.01.01 00:00:00.700| 194.8
2015.01.01 00:00:01.300| 200.96
2015.01.01 00:00:03.900| 215.34
..
```
We use `fby` to select records attaining the maximum price each day for each sym.
    
    
```q
q)select from trades where px=(max;px) fby sym
dt         tm           sym  qty  px
-------------------------------------
2015.01.01 00:20:05.835 goog 9750 660
2015.01.01 00:33:19.125 goog 3150 660
2015.01.01 00:42:13.379 goog 8790 660
2015.01.01 00:42:58.623 aapl 6090 110
..
```
We use `favg` from the previous section to demonstrate that user functions can appear in any phrase of the query.
    
    
```q
q)select from trades where px<2*(favg;px) fby sym
dt         tm           sym  qty  px
----------------------------------------
2015.01.01 00:00:00.448 goog 6940 540.18
2015.01.01 00:00:00.602 aapl 540  94.63
2015.01.01 00:00:00.754 ibm  3100 194.8
..
```
We find the average daily volume and price for all instruments and store the result for the following example.
    
    
```q
q)show atrades:select avgqty:avg qty, avgpx:avg px by sym, dt from trades
sym  dt        | avgqty   avgpx
---------------| -----------------
aapl 2015.01.01| 4997.978 99.99409
aapl 2015.01.02| 5006.318 100.0012
aapl 2015.01.03| 5002.49  100.0019
aapl 2015.01.04| 5012.752 99.97018
..
```
We find the days when the average price went up. Note that create our own `deltas0` to return 0 for the initial item since the behavior of the built-in [`deltas`](https://code.kx.com/q/ref/deltas/) doesn’t work well for this purpose. Observe that the nested result columns scroll off the page.
    
    
```q
q)deltas0:{first[x] -': x}
q)select dt, avgpx by sym from atrades where 0<deltas0 avgpx
sym | dt
----| ------------------------------------------------------------------
aapl| 2015.01.02 2015.01.03 2015.01.05 2015.01.07 2015.01.08 2015.01.10..
goog| 2015.01.01 2015.01.02 2015.01.03 2015.01.05 2015.01.07 2015.01.00..
ibm | 2015.01.05 2015.01.07 2015.01.08 2015.01.10 2015.01.11 2015.01.13..
```
To compact the display, take only the first few field values.
    
    
```q
q)select 2#dt, 2#avgpx by sym from atrades where 0<deltas0 avgpx
sym | dt                    avgpx
----| ---------------------------------------
aapl| 2015.01.02 2015.01.03 100.0012 100.0019
goog| 2015.01.01 2015.01.02 599.8873 600.0021
ibm | 2015.01.05 2015.01.07 200.0634 200.0022
```
In order to demonstrate operations on nested columns, we denormalize `trades` into a keyed table with one row and complex columns for each symbol. We can do this either by grouping without aggregation using select or by using [`xgroup`](https://code.kx.com/q/ref/xgroup/). The difference is that `xgroup` does not automatically sort on the result key column.
    
    
```q
q)dntrades:select dt,tm,qty,px by sym from trades
q)dntrades~`sym xasc `sym xgroup trades
1b
```
We display the first two items of each field to make the structure more evident.
    
    
```q
q)select 2#dt,2#tm,2#qty,2#px by sym from trades
sym | dt                    tm                        qty       px
----| -----------------------------------------------------------------------
aapl| 2015.01.01 2015.01.01 00:00:00.602 00:00:00.840 540 1260  94.63 92.87
goog| 2015.01.01 2015.01.01 00:00:00.448 00:00:01.039 6940 7260 540.18 560.04
ibm | 2015.01.01 2015.01.01 00:00:00.754 00:00:01.377 3100 5150 194.8 200.96
```
In such a table with compound columns – i.e., lists of simple lists of the same type – you will need iterators for column operations. Lots of iterators.
    
    
```q
q)select sym,cnt:count each dt,avgpx:avg each px from dntrades
sym  cnt     avgpx
---------------------
aapl 3333498 99.99694
goog 3330409 600.02
ibm  3336093 200.0041
```
We can apply our own unary `favg` function.
    
    
```q
q)select sym, favgpx:favg each px from dntrades
sym  favgpx
-------------
aapl 49.99748
goog 300.0155
ibm  100.0009
```
To find the volume-weighted average price by we use the Each iterator `'` with the binary [`wavg`](https://code.kx.com/q/ref/wavg/#wavg).
    
    
```q
q)select sym,vwap:qty wavg' px from dntrades
sym  vwap
-------------
aapl 99.99915
goog 600.0493
ibm  200.0061
```
Note that this iterator generalizes to _n_ -adic functions for any _n_ >1.

We find the profit of the ideal transaction over the month for each symbol. This is the maximum amount of money that could be made with complete foreknowledge of the market. In other words, find the largest profit obtainable by buying at some traded price and selling at the highest subsequently traded price. To solve this, we reverse the perspective. For each traded price, we look at the minimum of prices that preceded it. The largest such difference is our answer. The maximum draw down is dual.
    
    
```q
q)select max px-mins px by sym from trades
sym | px
----| ---
aapl| 20
goog| 120
ibm | 40
q)select min px-maxs px by sym from trades
sym | px
----| ----
aapl| -20
goog| -120
ibm | -40
```
### 9.13.5 Excursion – Pivot Table¶

The objective is to pivot a table on a column (or columns) as in Excel. The construct revolves (pun intended) around three columns. The `distinct` values of the _pivot_ column become the column names in the result. Each field in the _value_ column is placed under the pivot column whose name appears in its original row. The destination row of a value column field is determined by the _key_ column field in its original row. This is a pivot in the sense that the original vertical arrangement is transformed to horizontal and rows transform to columns. As usual, an example is worth a thousand words.

We begin with the simple table.
    
    
```q
q)show t:([]k:1 2 3 2 3; p:`a1`a2`a1`a3`a2; v:100 200 300 400 500)
k p  v
--------
1 a1 100
2 a2 200
3 a1 300
2 a3 400
3 a2 500
```
The desired result `pvt` is as follows. Observe that this is actually a keyed table but it is simple to unkey it with [`xkey`](https://code.kx.com/q/ref/xkey/3xkey).
    
    
```q
q)pvt
k| a1  a2  a3 
-| -----------
1| 100        
2|     200 400
3| 300 500    
```
We break the solution into multiple steps, beginning with our sample table and then successively generalizing to handle a wider class of use cases.

  1. Collect the unique values of the pivot column `p` into a list `P`.
         
```q
     q)P:exec distinct p from t
```
  2. Write a query that extracts the key-value pairs for `p` and `v` grouped by `k`.
         
```q
     q)exec p!v by k from t
     _
```
  3. Enhance the previous query to produce a keyed table by rectangularizing the dictionaries by filling missing values using `P#`. Magic happens. Observe that we need to name the resulting key column explicitly.
         
```q
     q)exec P#p!v by k:k from t
     k| a1  a2  a3 
     -| -----------
     1| 100        
     
     2|     200 400
     3| 300 500    
```
  4. Write the query to extract the unique values of the pivot column in functional form.
         
```q
     q)P:?[t; (); (); (distinct; `p)]
     _
```
  5. Convert the pivot query to functional form.
         
```q
     q)?[t;(); (1#`k)!1#`k; (#;`P;(!;`p;`v))]
     _
```
  6. Place the previous functional forms in a function that takes the table and the column names as parameters and returns the pivot result.
         
```q
     q)dopivot:{[t; kn; pn; vn]
       P:?[t; (); (); (distinct; pn)];
       ?[t;(); (1#kn)!1#kn; (#;`P;(!;pn;vn))]}
     q)dopivot[t;`k;`p;`v]
     _
```
  7. Write an expression that converts the fields of the pivot column to valid names when they are not – e.g., integers. In general, producing unique, valid names is a non-trivial exercise. Find a solution that (at least) works on `tn` below.
         
```q
     q)tn:([] k:1 2 3 2 3; p:(`a1;2;`a1;3;2); v:100 200 300 400 500)
     q)mkNames:{
       x:(::),x;
       x:1_x:@[x; where not 10h=type each x; string];
       `$@[x; where not any x[;0] within/: ("AZ";"az"); "X_",]}
     q)dopivot:{[t; kn; pn; vn]
       t:![t; (); 0b; (1#pn)!enlist (`mkNames; pn)];
       P:?[t; (); (); (distinct; pn)];
       ?[t;(); (1#kn)!1#kn; (#;`P;(!;pn;vn))]}
     q)dopivot[tn;`k;`p;`v]
     _
```
  8. Next, we generalize to the case when there are multiple entries in `v` for a single key value. For example, applied to `tr` below, our current `dopivot` misses the 1000 value in `v`.
         
```q
     q)tr:([]k:1 2 3 2 3 1; p:`a1`a2`a1`a3`a2`a1; v:100 200 300 400 500 1000)
     q)dopivot[tr;`k;`p;`v] / misses 1000 value
     k| a1  a2  a3
     -| -----------
     1| 100        
     
     2|     200 400
     3| 300 500    
```
We would like to apply an aggregate function such as `sum` to obtain,
         
```q
     k| a1   a2  a3
     -| ------------
     1| 1100
     2|      200 400
     3| 300  500
```
Modify `dopivot` to take an aggregate function `agg` and apply it to `v` as part of the pivot process. Test it on `tr` using `sum`.
         
```q
     q)dopivot:{[t; agg; kn; pn; vn]
       t:![t; (); 0b; (1#pn)!enlist (`mkNames; pn)];
       t:?[t; (); (kn,pn)!kn,pn; (1#vn)!enlist (agg;vn)];
       P:?[t; (); (); (distinct; pn)];
       ?[t; (); (1#kn)!1#kn; (#;`P;(!;pn;vn))]}
     q)dopivot[tr;sum;`k;`p;`v]
     _
```
  9. We would like to handle the case of compound keys – e.g., `k1` and `k2` in `tk` below.
         
```q
     q)k1:1 2 3 2 3 1 1
     q)k2:10 20 30 40 50 60 10
     q)p:`a1`a2`a1`a3`a2`a1`a1
     q)v:100 200 300 400 500 1000 10000
     q)show tk:([]k1:k1; k2:k2; p:p; v:v)
     k1 k2 p  v
     --------------
     1  10 a1 100
     2  20 a2 200
     3  30 a1 300
     2  40 a3 400
     3  50 a2 500
     1  60 a1 1000
     1  10 a1 10000
```
Modify `dopivot` to take a list of key column names and test it on `tk`.
         
```q
     q)dopivot:{[t; agg; ks; pn; vn]
       ks,:();
       t:![t; (); 0b; (1#pn)!enlist (`mkNames; pn)];
       t:?[t; (); (ks,pn)!ks,pn; (1#vn)!enlist (agg;vn)];
       P:?[t; (); (); (distinct; pn)];
       ?[t;(); ks!ks; (#;`P;(!;pn;vn))]}
     q)dopivot[tk;sum;`k1`k2;`p;`v]
     _
```
  10. Modify `dopivot` to accept a string argument representing a valid Where phrase for the input table and use it to constrain the pivot. Test it against `tk` with the phrase `k1<>2`.
         
```q
     q)dopivot:{[t; agg; wh; ks; pn; vn]
       ks,:();
       c:enlist parse wh;
       t:?[t; c; 0b; (cols t)!cols t];
       t:![t; (); 0b; (1#pn)!enlist (`mkNames; pn)];
       t:?[t; (); (ks,pn)!ks,pn; (1#vn)!enlist (agg;vn)];
       P:?[t; (); (); (distinct; pn)];
       ?[t; (); ks!ks; (#;`P;(!;pn;vn))]}
     q)dopivot[tk;sum;"k1<>2";`k1`k2;`p;`v]
     _
```
  11. Finally, modify `dopivot` to accept an empty list in the aggregate or the where parameters. In the former case, use `first` as the default aggregate; in the latter, perform no constraint. Test against `tk` with empty list arguments for both.
         
```q
     q)dopivot:{[t; agg; wh; ks; pn; vn]
       ks,:();
       agg:first agg,first;
       c:$[count wh; enlist parse wh; ()];
       t:?[t; c; 0b; (cols t)!cols t];
       t:![t; (); 0b; (1#pn)!enlist (`mkNames; pn)];
       t:?[t; (); (ks,pn)!ks,pn; (1#vn)!enlist (agg;vn)];
       P:?[t; (); (); (distinct; pn)];
       ?[t; (); ks!ks; (#;`P;(!;pn;vn))]}
     q)dopivot[tk;();"";`k1`k2;`p;`v]
     _
```
We test the efficiency of `dopivot` by applying it to a modestly large table with a million rows of timeseries data. Take that, Excel!
    
    
```q
q)t:`date xasc ([] date:2015.01.01+1000000?10; sym:1000000?`aapl`ibm`intc; qty:1+1000000?100)
q)dopivot[t; sum; ""; `date; `sym; `qty] / ymmv
date      | aapl    ibm     intc
----------| -----------------------
2015.01.01| 1688672 1666730 1659389
2015.01.02| 1695257 1688088 1679517
2015.01.03| 1656634 1692806 1688195
2015.01.04| 1686011 1708072 1687046
2015.01.05| 1676870 1683170 1699435
2015.01.06| 1673952 1676820 1687553
2015.01.07| 1683041 1680907 1692974
2015.01.08| 1661363 1688660 1675261
2015.01.09| 1692126 1693217 1690017
2015.01.10| 1698429 1699250 1679042
```
It is an instructive exercise to write `unpivot`, which takes a pivoted table and reverts the columns back to rows.

Back to top 
