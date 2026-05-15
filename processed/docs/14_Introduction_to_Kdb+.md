# 14 Introduction To Kdb+

# 14. Introduction to Kdb+¶

## 14.0 Overview¶

In this chapter we provide an introduction to kdb+, which is the database offering from KX. Roughly speaking, kdb+ is what happens when q tables are persisted and then mapped back into memory for operations. Of course, things are more complicated than that.

Important

Many examples in this chapter use directories that are parented directly to the root. This is done for pedagogical reasons and also to make paths short enough to fit on a page without wrapping. Unless you are creating kdb+ databases on a machine over which you have complete control, you will not be able to create directories in the root. When emulating the examples here, you should change the paths to relative paths that live under a directory that you control. You can usually do this by removing the leading / in the path.

`/`## 14.1 Tables in Memory and Serialization¶

It is possible to maintain a table entirely in memory, provided you have enough physical memory to hold it. There is one problem with this from a database perspective:

- An in-memory table is ephemeral â meaning that all modifications are lost if the q process dies.

One solution is to serialize the table to persistent storage using set or similar mechanisms. In this section we recapitulate material from previous chapters from this perspective.

`set`### 14.1.1 Tables and Keyed Tables¶

A table is the flip of a column dictionary, in which address slots are reversed but no data is moved during the transpose. For example, here is a table with two simple list columns.

`flip````q
q)flip `s`v!(`a`b`c;100 200 300)
_
```

Table definition syntax permits tables to be defined in more readable format.

```q
q)([] s:`a`b`c; v:100 200 300)
_
```

The schema of a table has the same form but with empty columns â i.e., no rows.

```q
q)([] s:`symbol$(); v:`int$())
_
q)t:([] s:`a`b; v:10 20)
_
```

It is good practice to prototype the empty lists in a schema; unfortunately, this is not possible if the corresponding columns are not simple lists.

The type of any table is 98h and the function meta summarizes the column names, types and attributes in a result keyed table.

`98h``meta````q
q)meta ([] s:`symbol$(); v:`int$())
_
```

A keyed table is a dictionary that establishes positional correspondence between a table of (presumably unique) keys and a table of values.

```q
q)([] id:1001 1002 1003)!([] s:`a`b`c; v:100 200 300)
id | s v
----| -----
1001| a 100
1002| b 200
1003| c 300
```

Table definition syntax is more compact.

```q
q)([id:1001 1002 1003] s:`a`b`c; v:100 200 300)
_
```

The type of any keyed table is 99h, since it is a dictionary and meta applies exactly as with tables.

`99h``meta`### 14.1.2 Foreign Keys and Link Columns¶

A foreign key is one or more table columns that are enumerated over the key component of a keyed table. For example, the column ID in the table below is a foreign key over kt. Note that the foreign-key column is identified by the name of its target table in the result of meta.

`kt``meta````q
q)kt:([id:1001 1002 1003] s:`a`b`c; v:100 200 300)
q)t:([]; id:`kt$1002 1001 1003 1001; q:100 101 102 103)
q)meta t
c | t f a
--| ------
id| j kt
q | j
```

A query on a table having a foreign key can access columns in the keyed table via dot notation.

```q
q)select id.v, q from t
v q
-------
200 100
100 101
300 102
100 103
```

A link column is similar to a foreign key, in that its entries are indices of rows in a table, but you must perform the lookup manually. The advantages of link columns are:

- The target can be a table or keyed table.
- The target can be the table containing the link column.
- Link columns can be splayed or partitioned, whereas foreign keys cannot.

Here is the previous foreign-key example redone with a link column against a table.

```q
q)tk:([] id:1001 1002 100; s:`a`b`c; v:100 200 300)
q)t:([]; id:`tk!(exec id from tk)?1002 1001 1003 1001; q:100 101 102 103)
q)meta t
c | t f a
--| ------
id| i tk
q | j
```

Important

As of this writing (Sep 2015) integers are 64-bit in q3+, enumerations are 32-bit. In particular, the integers stored in a link column are truncated to 32 bits by the ! operator. Observe that type of the link column ID in the previous example is I even though the result of the ? operation is of type j. One consequence is that you cannot use links columns with tables of more than 232 records.

`!``I``?``j`Here is an example that uses a link column to implement a hierarchical structure in a table. The column pid is a link column that relates a row to its parent row.

`pid````q
q)tree:([] id:0 1 2 3 4; pid:`tree!0N 0 0 1 1; v:100 200 300 400 500)
q)select from tree where pid=0 / find children of root
_
```

### 14.1.3 Serializing Tables¶

It is possible to persist any table (or keyed table) using the general q serialization/deserialization capability of set and get. There is no restriction on table or column types.

`set``get````q
q)`:/data/t set ([] s:`a`b`c; v:100 200 300)
`:/data/t
q)\\
>q
q)t:get `:/data/t
q)t
_
```

You can serialize foreign keys and link columns and bring them back into memory.

```q
q)kt:([id:1001 1002 1003] s:`a`b`c; v:100 200 300)
q)tk:([] id:1001 1002 100; s:`a`b`c; v:100 200 300)
q)`:/data/kt set kt
`:/data/kt
q)`:/data/tk set tk
`:/data/tk
q)`:/data/t1 set ([]; id:`kt$1002 1001 1003 1001; q:100 101 102 103)
`:/data/t1
q)`:/data/t2 set ([]; id:`kt!(exec id from tk)?1002 1001 1003 1001; q:100 101 102 103)
`:/data/t2
q)\\
>q
q)kt:get `:/data/kt
q)tk:get `:/data/tk
q)t1:get `:/data/t1
q)t2:get `:/data/t2
```

### 14.1.4 Operating on Serialized Tables¶

You operate on a serialized table by loading it into memory with get or \l.

`get``\l````q
q)`:/data/t set ([] s:`a`b`c; v:100 200 300)
`:/data/t
q)\l /data/t
`t
q)select from t
_
q)t1: get `:/data/t
q)select from t1
_
```

Alternatively, you can perform a query on a serialized table by specifying its file handle as the table name.

```q
q)`:/data/t set ([] s:`a`b`c; v:100 200 300)
`:/data/t
q)select from `:/data/t where s in `a`c
s v
-----
a 100
c 300
q)`:/data/t upsert (`x;42)
`:/data/t
q).[`:/data/t;();,;([] s:`y`z; v:400 500)]
`:/data/t
q)select from `:/data/t where s in `x`y`z
_
```

Similar operations are available on keyed tables.

```q
q)`:/data/kt set ([k:`a`b`c] v:10 20 30)
`:/data/kt
q)`:/data/kt upsert (`b;200)
`:/data/kt
q)`:/data/kt upsert (`x;42)
`:/data/kt
q)select from `:/data/kt
k| v
-| ---
a| 10
b| 200
c| 30
x| 42
```

Note

The limitation to using a serialized table or keyed table is that, behind the scenes, the operations load it into memory and write it back out. Amongst other things, this means that anyone wanting to work with it must be able to fit it into memory in its entirety.

### 14.1.5 The Database View¶

Persisting a table via serialization creates a single file in the OS file system. Letâs raise our perspective to the 50,000-foot level. From this height, we see the table as a single point.

Big Picture (0): We think of a serialized table as a zero-dimensional persisted form since it is stored in a single file and is not decomposed.

Here the âª represents the contents of the table as an opaque entity. This contrasts with higher-dimensional forms we shall encounter later.

## 14.2 Splayed Tables¶

In the previous section, we saw that it is possible to persist tables using serialization. From a database perspective there are (at least) two issues with operating on serialized tables due to the fact that the entire table is loaded into memory.

- The entire table must fit into memory on each userâs machine.
- Operations against the persisted table will be slow due to reloading the entire table each time.

### 14.2.0 Splaying a Table¶

When a table is too large to fit into memory as a single entity, we can persist its components into a directory. This is called splaying the table because the table is pulled apart into its constituent columns (it is an interesting exercise to look up the derivation of this word in English). Splaying solves the memory/reload issue because a splayed table is mapped into memory; columns are loaded on demand then memory is released when no longer needed. Tables with many columns especially benefit from splaying since most queries refer to only a handful of columns and only those columns will actually be loaded.

A splayed table corresponds to a directory whose name is the table name. Each column list of the table is serialized into a file whose name is the column name.

You donât have any choice in the names.

A list of the symbolic column names is serialized to the hidden file .din the directory to record column order. This is the only metadata stored by kdb+; all other metadata is read from directory and file names.

`.d`Customarily the splayed directory is created one level down from a directory that serves as the root of the database.

```q
/root
/tablename <- splayed table directory
.d <- file with column names
column1name <- column data file
column2name <- column data file
â¦
```

From the 50,000-foot level, we see the following on the file system.

Big Picture (1): We think of a splayed table as a persisted form that is cut vertically along columns.

Geometrically, the persisted table is 1-dimensional â there is a point for each column in the persisted image of the table.

### 14.2.1 Creating Splayed Tables¶

We start with a very simple example of creating a splayed table. Make sure you include the trailing / in the file handle; otherwise, you will serialize the table into a single file.

`/````q
q)`:/db/t/ set ([] v1:10 20 30; v2:1.1 2.2 3.3)
`:/data/t/
q)\ls -a /db/t
,"."
".."
".d"
"v1"
"v2"
```

It is also possible to create a splayed table with upsert, or with the equivalent generalized application, using the file handle as the table name. When the file does not exist, these act like set.

`upsert``set````q
q)`:/db/t2/ upsert ([] v1:10 20 30; v2:1.1 2.2 3.3)
_
q).[`:/db/t3/; (); ,; ([] v1:10 20 30; v2:1.1 2.2 3.3)]
_
```

Reading the constituents of the splayed directory with get demonstrates that they are simply serialized q entities.

`get````q
q)get `:/db/t/v1
10 20 30
q)get `:/db/t/v2
_
q)get `:/db/t/.d
`v1`v2
```

Of course, if your notion of fashion is a hair shirt, you can splay your table manually. In a fresh /db directory,

`/db````q
q)t:([] v1:10 20 30; v2:1.1 2.2 3.3)
q)cs:cols t
q){[cname] (hsym `$"/db/t",string cname) set t cname} each cs
_
q)`:/db/t/.d set cs
_
```

There are serious restrictions on what can be splayed.

- Tables can be splayed. Keyed tables cannot.
This might seem to preclude relational capabilities since it eliminates persisting a foreign key relation. But the day is saved by link columns, which can be persisted. See Â§14.1.2.
- Only columns that are simple lists or compound lists can be splayed. By compound list we mean a list of simple lists of uniform type.
The isnât too limiting in practice since data often comes in a uniform format that can be readily put into simple or compound lists. Incidentally, the reason for this restriction is that working with mapped general lists would be much slower than working with simple or compound lists, especially for very large data sets. See Â§14.2.4 for more on nested columns.
- All symbol columns must be enumerated.
This restriction might seem to complicate life but there are conventions and utilities that eliminate most of the pain. See Â§14.2.2.

Tables can be splayed. Keyed tables cannot.

This might seem to preclude relational capabilities since it eliminates persisting a foreign key relation. But the day is saved by link columns, which can be persisted. See Â§14.1.2.

Only columns that are simple lists or compound lists can be splayed. By compound list we mean a list of simple lists of uniform type.

The isnât too limiting in practice since data often comes in a uniform format that can be readily put into simple or compound lists. Incidentally, the reason for this restriction is that working with mapped general lists would be much slower than working with simple or compound lists, especially for very large data sets. See Â§14.2.4 for more on nested columns.

All symbol columns must be enumerated.

This restriction might seem to complicate life but there are conventions and utilities that eliminate most of the pain. See Â§14.2.2.

### 14.2.2 Splayed Tables with Symbol Columns¶

The convention for symbol columns in splayed (and partitioned) tables is that all symbol columns in all tables are enumerated over the list sym, which is serialized into the root directory.

The author prefers to use a projected form of the utility .Q.en for enumerating symbols. Its first parameter is the file handle of the root directory (the location of the sym file) and its second parameter is the table whose symbol columns you wish enumerated. Here is a simple example.

`.Q.en````q
q)`:/db/t/ set .Q.en[`:/db;] ([] s1:`a`b`c; v:10 20 30; s2:`x`y`z)
`:/db/t/
q)\ls /db
"sym"
,"t"
q}\\
>q
q)\l /db
q)sym
`a`b`c`x`y`z
q)select from t
_
```

We offer several observations on the action of .Q.en in this example.

`.Q.en`- If there is a sym list in memory, it is overwritten
- If there is a sym list on disk it is locked and then loaded into memory
- If no sym list exists in memory or on disk an empty one is created.
- All symbols in all symbol columns of the table are conditionally enumerated over the sym list in memory.
- Once the enumeration is complete the sym list in memory is serialized to the root and the file is unlocked.

Once again, you could don your hair shirt and do symbol enumeration manually.

```q
q)t:([] s1:`a`b`c; v:10 20 30; s2:`x`y`z)
q)sym:()
q)`:/db/t/ set @[t;exec c from meta t where "s"=t;`sym?]
_
q)sym
_
\\
>q
q)\l /db
q)select from t
_
```

### 14.2.3 Splayed Tables with Nested Columns¶

The only nested columns that can be splayed are what we call compound lists â i.e., lists of simple lists of uniform type. The most common example is a list of strings, which is a list of lists of char. A compound column is indicated by an upper case letter in the result of meta. For example, here is a table having two compound columns, which can be splayed in a fresh directory /db.

`meta``/db````q
q)meta ([] ci:(1 2 3; enlist 4; 5 6); cstr:("abc";enlist"d";"ef"))
c   | t f a
----| -----
x   | J
cstr| C
q)`:/db/tcomp2/ set ([] ci:(1 2 3; enlist 4; 5 6); cstr:("abc";enlist"d";"ef"))
_
```

By contrast, the following table cannot be splayed and results in an error.

```q
q)meta ([] c:(1;1,1;`1))
c| t f a
-| -----
c|
q)`:/db/tcomp2/ set ([] c:(1;1,1;`1))
k){$[@x;.[x;();:;y];-19!((,y),x)]}
'type
q.q))
```

Tip

You might reasonably think that you can use the result of meta to determine whether your table can be splayed by checking for a blank in the t column. You would be wrong since meta only examines the initial item of a column list and improperly reports some mixed columns as compound.

`meta``t``meta````q
q)meta ([] c:(1 2 3 ;1,1;`1))
c| t f a
-| -----
c| J
q)`:/db/tbad/ set ([] c:(1 2 3 ;1,1;`1))
k){$[@x;.[x;();:;y];-19!((,y),x)]}
'type
q.q))
```

The following function will tell you which columns of a table cannot be splayed.

```q
{where {(ts~1#0h)|1<count ts:distinct `type each`x} each flip x}
```

When you splay a table with compound columns, q creates not one but two files for each compound column. For example, in a fresh directory /db, we splay a table with a single string column and then inspect the splayed directory.

`/db````q
q)`:/db/tstr/ set ([] c:("abc";enlist "d";"ef"))
_
q)system "ls -a /db/tstr"
,"."
".."
".d"
,"c"
"c#"
```

Observe that there are two files associated with the compound column  â namely, c and c#. If you examine these files you will discover that the âsharpâ file contains the binary data of the original list in flattened form and the non-sharp file is a serialized q list of integers representing the lengths of each sublist of the original list.

`c``c#````q
q)"c"$read1 hsym `$"/db/tstr/c#"
"abcdef"
q)read1 hsym `$"/db/tstr/c"
0xfe205700000000000300000000000000030000000000000004000000000000000600000000000000
```

Interestingly, get reconstitutes the list from the non-sharp file.

`get````q
q)get hsym `$"/db/tstr/c"
"abc"
,"d"
"ef"
```

The purpose of writing compound columns as two files is to speed up operations against them when the splayed table is mapped into memory. Of course, the processing wonât be as fast as for a simple column, but it is still plenty fast for most purposes.

One question that always arises when designing a kdb+ database is whether to store text data as symbols or strings. The advantage of symbols is that they have atomic semantics and, since they are integers under the covers once they are enumerated, processing is quite fast. The main issue with symbols is that if you make all text into symbols, your sym list gets enormous and the advantages of enumeration disappear.

In contrast, strings do not pollute the sym list with one-off instances and are reasonably fast. The disadvantage is that they are not first class and you must revert to teenage years by using like to match them.

`like`Recommendation

Only make text columns into symbols when the fields will be drawn from a small, reasonably stable domain and there is significant repetition in their use. When in doubt, start with a string column. It is much easier to convert a string column to symbols that it is to remove symbols from the sym list.

A text column that is drawn from a fixed list or a lookup table is an ideal candidate. So are invariant keys, provided the key domain is small and will not grow unreasonably. On the other hand, fields such as comments or notes should always be strings.

### 14.2.4 Basic Operations on Splayed Tables¶

Let us create a splayed table in a fresh /db.

`/db````q
q)`:/db/t/ set .Q.en[`:/db;] ([] s1:`a`b`c; v:10 20 30; s2:`x`y`z)
_
```

To operate on a splayed table you can map it into memory in one of two ways.

Note the lack of a trailing / in both techniques of mapping the table below.

`/`You can specify a splayed table on the q startup command immediately after the q executable.

```q
$q /db/t
```

```q
q)select from t
_
```

Alternatively, you can use \l to map the table.

`\l`Confusingly, folks say they are âloadingâ the table, even though none of the table data is actually brought into memory in this step.

```q
q)\l /db/t
`t
q)select from t
_
```

The illusion that the table is actually in memory after it is mapped is convincing. Many fundamental table operations work on splayed tables.

```q
q)\a
,`t
q)meta t
c | t f a
--| -----
s1| s
v | j
s2| s
q)cols t
`s1`v`s2
q)type t
98h
q)count t
3
```

You can index records.

```q
q)t[0]
s1| `sym$`a
v | 10
s2| `sym$`x
```

You cannot use dot notation to extract columns from a splayed table but you can extract a column with symbol indexing.

```q
q)t.s1
'type
q)t `s1
`sym$`a`b`c
```

You can use both select and exec templates on a splayed table.

`select``exec````q
q)select v from t where s1=`c
_
q)exec s1 from t
_
```

Tip

This contrasts with partitioned tables where you can only use select.

`select`### 14.2.5 Operations on a Splayed Directory¶

As of this writing (Sep 2015), the table operations available against the file handle of a splayed table are: select, exec, upsert, xasc, `attr# (apply an attribute).

`select``exec``upsert``xasc```attr#`With t splayed as above,

`t````q
q)select from `:/db/t
_
q)exec v from `:/db/t
_
q)`v xdesc `:/db/t
_
q)@[`:/db/t;`s1;`p#]
_
q)\l /db
q)meta t
_
```

We point out a source of confusion to qbies. Specifically, the behavior of update on a splayed table that has been mapped into memory. Starting with a fresh directory /db, create a splayed table, map it into memory and then update it.

`update``/db````q
q)`:/db/t/ set .Q.en[`:/db;] ([] s1:`a`b`c; v:10 20 30; s2:`x`y`z)
_
q)\l /db
q)update v:300 from `t where s1=`c
`t
q)select from t
s1 v s2
---------
a 10 x
b 20 y
c 300 z
```

But a rude surprise awaits when we remap the table.

```q
q)\l /db
q)select from t
s1 v  s2
--------
a  10 x
b  20 y
c  30 z
```

Updates applied to a mapped table are only visible in the workspace and are not reflected on disk. There may be scenarios where this is useful but we advise avoiding it.

And now we arrive at a fundamental limitation of kdb+.

Really Important

It is not possible to use built-in operations to update data in persisted splayed tables.

You read that correctly. Kdb+ is intended to store data that is not updated or deleted once it has been written. We shall see in the next section how to append to a splayed table, which makes it possible to process updates and deletes in a bitemporal fashion, but this capability is not available out of the box.

### 14.2.6 Appending to a Splayed Table¶

Since upsert acts as insert on regular (non-keyed) tables and only non-keyed tables can be splayed, we use upsert with the splayed directory name in order to append records to a splayed table on disk. This is a good thing, since insert doesnât work on splayed tables. Also, because symbol columns must be enumerated for splayed tables, it is best to make rows into tables.

`upsert``insert``upsert``insert`In a fresh /db,

`/db````q
q)`:/db/t/ set .Q.en[`:/db;] ([] s1:`a`b`c; v:10 20 30; s2:`x`y`z)
_
q)`:/db/t/ upsert .Q.en[`:/db;] ([] s1:`d`e; v:40 50; s2:`u`v)
_
q)`:/db/t upsert .Q.en[`:/db;] enlist `s1`v`s2!(`f;60;`t)
_
q)`:/db/t upsert .Q.en[`:/db;] flip `s1`v`s2!flip ((`g;70;`r);(`h;80;`s))
_
q)\l / db
q)select from t
_
```

You can use upsert in this fashion to build large splayed tables incrementally. The following example can be enhanced to create a splayed table step-by-step as batches of new records arrive. In a fresh /db,

`upsert``/db````q
q)batch:{[rt;tn;recs] hsym[`$rt,"/",tn,"/"] upsert .Q.en[hsym `$rt;] recs}
q)dayrecs:{([] dt:x; ti:asc 100?24:00:00; sym:100?`ibm`aapl; qty:100*1+100?1000)}
q)appday:batch["/db";"t";]
q)appday dayrecs 2015.01.01
_
q)appday dayrecs 2015.01.02
_
q)appday dayrecs 2015.01.03
_
q)\l /db
q)select from t
_
```

### 14.2.7 Manual Operations on a Splayed Directory¶

Although there are no built-in operations to update splayed tables on disk, you can perform such operations by manipulating the serialized files.

Important

The examples shown here should be used with caution, as none of the operations are atomic; they are simply file-system manipulation. Even read-only users could see inconsistent data, so things are best done when no other users are accessing the database.

Here is an example of how to simulate an update .. where on a single column. In a fresh /db,

`update .. where``/db````q
q)`:/db/t/ set ([] ti:09:30:00 09:31:00; p:101.5 33.5)
_
q)`:/db/t/p set .[get `:/db/t/p; where 09:31:00=get `:/db/t/ti; :;42.0]
_
q)\l /db/t
_
q)select from t
_
```

To add a new column of nulls to an existing splayed table, we first get the row count from any column file. Then we revise the .d file with the new column name. This example is easily modified to place arbitrary data in the new column.

`.d````q
q)`:/db/t/s set (count get `:/db/t/ti)#`
_
q)`:/db/t/.d set get[`:/db/t/.d] union `s
_
```

To delete a column, remove its file and revise the .d file to reflect its absence.

`.d````q
q)system "rm /db/t/s"
q)`:/db/t/.d set get[`:/db/t/.d] except `s
_
q)\l /db
q)select from t
_
```

While you can sort a splayed table on disk, suppose you want to implement your own custom sort. We show how to create the standard sort. First create the appropriate sort index with iasc or idesc and then use this to re-index all the column files. Here we perform a descending sort by the ti column.

`iasc``idesc``ti````q
q)cs:system "ls /db/t"
q)I:idesc `:/db/t/ti
q){pth set get[pth:hsym `$"/db/t/",x] I} each cs
`:/db/t/ix`:/db/t/p`:/db/t/ti
q)\l /db
q)select from t
_
```

### 14.2.8 Working with sym Files¶

On occasion you may need to perform operations involving the sym files. For example, you may want to move a table from one database to another. Or you may find that your sym domain has become polluted due to symbol columns that should have been strings. Or you may have unwisely chosen to have many different enumeration domains and you wish to consolidate them into one. All these situations can be resolved by careful operations on the sym file(s).

Updating sym files is delicate and should be done with great care, only after you have made a complete backup of the root directory. Corrupting the sym file will almost certainly render your database useless.

In our first example we show how to merge a table from one database into another. Suppose we have root directories /db1 and /db2, both with the standard sym file in the root and with splayed tables t1 and t2, respectively. In fresh directories /db1 and /db2,

`/db1``/db2``t1``t2``/db1``/db2````q
q)`:/db1/t1/ set .Q.en[`:/db1;] ([] s:`a`b`c; v:10 20 30)
_
q)`:/db2/t2/ set .Q.en[`:/db2;] ([] s:`c`d`e; v:300 400 500)
_
q)get `:/db1/sym
`a`b`c
q)get `:/db2/sym
`c`d`e
```

In order to copy table t2 to the /db1 root, we un-enumerate all its symbol columns and then re-enumerate them in the target database.

`t2``/db1````q
q)symcols2:exec c from meta `:/db2/t2 where t="s"
q)t2noenum:@[select from `:/db2/t2;symcols2;value]
q)`:/db1/t2/ set .Q.en[`:/db1] t2noenum
_
q)get `:/db1/sym
`a`b`c`d`e
```

In another example, suppose that we mistakenly made a comment column into symbols instead of strings, with the result being a pollution of the sym domain with many single-use items. In a fresh /db, we save t1 with a valid symbol column and then screw up on the comment field in t2.

`/db``t1``t2````q
q)`:/db/t1/ set .Q.en[`:/db;] ([] s:`a`b`c; v:10 20 30)
_
q)`:/db/t2/ set .Q.en[`:/db;] ([] c:1 2 3; comment:`$("abc";"de";"fg"))
_
q)get `:/db/sym
`a`b`c`abc`de`fg
```

First we load the database and remove the offending comments from the sym list.

```q
q)\l /db
q)sym:sym except exec comment from t2
q)sym
_
```

Next we re-enumerate the re-typed culprit table over the clean sym list and re-splay it. Note that we use (`sym?) for the enumeration since it is possible that some of the comments actually overlapped with symbols from other tables.

``sym?````q
q)reenum:{@[x;exec c from meta x where t="s";`sym?]}
q)`:/db/t2/ set reenum ([] s:`a`b`c; comment:("abc";"de";"fg"))
```

Next we un-enumerate, re-enumerate and re-splay the remaining tables.

```q
q)ts:system["a"] except `t2
q)unenum:{@[select from x; exec c from meta x where t="s";value]}
q){(hsym `$"/db/",string[x],"/") set reenum unenum x} each ts
_
```

Finally we overwrite the sym file with the correct sym list that is now in memory and reload the database to check our handiwork.

```q
q)`:/db/sym set sym
q)\l /db
q)meta t1
_
q)meta t2
_
```

### 14.2.9 Splayed Tables with Link Columns¶

We previously pointed out that you cannot splay a keyed table, and therefore cannot have a foreign-key relation between splayed tables. However, you can splay a link column and then use dot notation just as you would with a foreign key. You must do the work of creating the index yourself, just as with link columns with tables in memory.

In our first example, we create the link at the same time as we splay the tables. This is the same as creating the link on in-memory tables and then splaying them. In a fresh /db,

`/db````q
q)t1:([] c1:`c`b`a; c2: 10 20 30)
q)t2:([] c3:`a`b`a`c; c4: 1. 2. 3. 4.)
q)update t1lnk:`t1!t1[`c1]?t2[`c3] from `t2
_
q)`:/db/t1/ set `.Q.en[`:/db;] t1
_
q)`:/db/t2/ set `.Q.en[`:/db;] t2
_
q)\\
$q
q)\l /db
q)meta t2
c | t f a
-----| ------
c3 | s
c4 | f
t1lnk| i t1
q)select c3,t1lnk.c2,c4 from t2
_
```

Now we redo this example, assuming that the tables have already been splayed. You could map the database into memory but letâs work directly with the files. We have the additional step of appending the link columns to the .d file for t2.

`.d``t2````q
q)`:/db/t1/ set .Q.en[`:/db;] ([] c1:`c`b`a; c2: 10 20 30)
_
q)`:/db/t2/ set .Q.en[`:/db;] ([] c3:`a`b`a`c; c4: 1. 2. 3. 4.)
_
q)`:/db/t2/t1link set `t1!(get `:/db/t1/c1)?get `:/db/t2/c3
_
q).[`:/db/t2/.d;();,;`t1link]
_
q)\l /db
q)meta t2
_
```

### 14.2.10 Query Execution on Splayed Tables¶

As we noted in the introduction of the chapter, splayed tables are a memory win, in that the table is mapped into memory and columns are actually loaded only as needed. Since a typical query requires only a handful of columns, this can greatly reduce the amount of memory required to process a table with many columns.

In addition to the memory win, if the same columns are referenced again soon, there is a good chance that their data is cached, either in memory or by the storage controller. This can be a significant performance win.

There is also a more subtle performance win in how the where phrase is implemented. As mentioned in Chapter 11, each where subphrase produces a boolean vector that acts as mask, on which positions will be examined in the subsequent grouping and aggregation. When the table is mapped, this can also significantly reduce the amount of data that is loaded into memory.

## 14.3 Partitioned Tables¶

To recap the previous section, a splayed table has been split vertically along its columns and is subsequently mapped into memory. When a query is executed against the mapped table, column data is loaded as needed and is presumably cached for subsequent access. This reduces the tableâs memory footprint, especially for tables with many columns.

Some timeseries data is so large that even the individual columns may not fit into memory â for example, daily trades and quotes for an entire exchange. In this case, we can further decompose the table by slicing horizontally â called partitioning in kdb+. For example, the solution for trades and quotes is to slice into daily portions. The result is a collection of daily splayed directories, one for each day for which data exists.

All partitioned tables are splayed but not all splayed tables are partitioned.

### 14.3.1 Partitions¶

A partitioned table is a splayed table that is further decomposed by grouping records having common values along a column of special type. The allowable special column types have the property that the underlying value is an integer: date, month, year and long.

The slice of records having a given value is splayed into a directory, called a partition, whose name is that common value. In the canonical finance example, historical trades (or quotes) are stored in daily partition directories â remember a q date is an integer under the covers.

Big Picture (2): We think of a partitioned table as a two-dimensional persisted form since it is cut in two directions: vertically by splaying along columns and horizontally by slicing into partitions.

Here each âª represents a (partial) column of a partition slice as an opaque entity.

As with splaying, kdb+ implements partitioning using the OS directory structure. Each partition must be stored in a directory immediately under the database root directory. The directory name is the common column value for all the rows in the partition. Each partition directory in turn contains a subdirectory holding a splayed table with the records for that slice. The two-dimensional decomposition is reflected in two levels of nested directories under the root.

Since each splayed table subdirectory name is the name of the table, these names must be consistent across all partitions. Likewise, as portions of a single table, all splayed slices should have identical schemas. This means that each slice directory contains column files with the same names and internal data types, as well as identical .d files.

`.d````q
/root
    /partitionvalue1
        /tablename
            .d
            column1name
            column2name
            â¦
    /partitionvalue2
        /tablename
            .d
            column1name
            column2name
            â¦
        â¦
```

Important

Since a partition directory name factors out the common value for all records in its slice, do not include the partition column when you splay a partition slice â e.g., do not include a date column for a daily partitioned table. Instead, kdb+ infers the name, value and type from the partition directory name and creates a virtual column from this information. The name of the virtual column is set by q and cannot be controlled.

### 14.3.2 Partition Domain¶

We call the type of the virtual column for the partition the partition domain. As noted previously, the partition domain must have an underlying integral value.

You cannot use a symbol column as a partition domain, even if the symbols are enumerated.

For example, suppose we have daily timeseries data. A daily partitioning could look as follows.

2015.01.01, 2015.01.02, â¦

The virtual column is date in this case. Under each partition directory, there is a subdirectory t with that dayâs slice of records.

`t````q
/db
    /2015.01.01
        /t          <- trade records for 2015.01.01
    /2015.01.02
        /t          <- trade records for 2015.01.02 â¦
```

Important

A kdb+ database can only have a single partition domain. This means that you must create separate databases if you need partitions of different granularity. For example, you cannot have daily and monthly partitions in one database.

There is no requirement that there be entries for every value in the partition domain. For example, there may be no trades for holidays.

```q
/db
    /2014.12.31
        /t
    /2015.01.02
        /t
    â¦
```

If we partition by month,

2015.01m, 2015.02m, â¦

the virtual column name is month.

`month`For daily data that requires frequent multi-year queries, partitioning by year may be better.

2010, 2011, â¦

In this case, the virtual column is year even though there is no year type in q. The values are recognized as valid q years if they are less than 10,000.

`year`You can also partition by an arbitrary int such as a bin number.

0, 1, â¦

This results in a virtual column int.

`int`### 14.3.3 Creating Partitioned Tables¶

For simplicity, our first example has no symbols. In a fresh /db,

`/db````q
q)`:/db/2015.01.01/t/ set ([] ti:09:30:00 09:31:00; p:101 102f)
_
q)`:/db/2015.01.02/t/ set ([] ti:09:30:00 09:31:00; p:101.5 102.5)
_
q)\l /db
q)t / never do this at home!
date ti p
-------------------------
2015.01.01 09:30:00 101
2015.01.01 09:31:00 102
2015.01.02 09:30:00 101.5
2015.01.02 09:31:00 102.5
```

The table appears to be in the workspace, along with the virtual date columns, but this is an illusion. It is actually mapped into memory. The request to display t forces all columns for all days to be loaded into memory.

`t`Important

Always qualify the partition column in the first where sub-phrase in any query against a partitioned table. If you do not, you will cause all partitions to be loaded into memory and will probably live-lock the server. Well before this completes, your colleagues will be at your desk with pitchforks and burning torches.

A more responsible way to create the previous display is,

```q
q)select from t where date within 2015.01.01 2015.01.02
_
```

Here we partition tables with symbols in a fresh /db.

`/db````q
q)`:/db/2015.01.01/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; s:`ibm`msft; p:101 33f)
_
q)`:/db/2015.01.02/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; s:`ibm`msft; p:101.5 33.5)
_
q)\l /db
q)select from t where date within 2015.01.01 2015.01.02
_
```

### 14.3.4 Working with Partitioned Tables¶

Continuing with the last example of the previous section, we can apply some basic operations on a partitioned table that has been mapped. Observe that the virtual partition column is included in the results of meta and cols.

`meta``cols````q
q)count t
_
q)meta t
c   | t f a
----| -----
date| d
ti  | v
s   | s
p   | f
q)cols t
`date`ti`s`p
q)type t
98h
```

However, none of the following work on partitioned tables, even though they work on splayed tables.

```q
t[0j]
t[;`p]
0#t
exec from t
select[1] from t
`p xasc t
```

Tip

The fact that exec doesnât work on partitioned tables is annoying but the workaround is,

`exec`exec â¦ from select â¦ from â¦

```q
exec â¦ from select â¦ from â¦
```

The select template, or the equivalent functional form, is the way to access data for a partitioned table. We have already seen how to retrieve the records for consecutive days. Here is the query to retrieve a dayâs slice.

```q
q)select from t where date=2015.01.01
_
```

Here is how to retrieve the first or last day without hardcoding the date.

```q
q)select from t where date=first date
_
q)select from t where date=max date
_
```

Always place the partition column constraint first.

```q
q)select from t where date=2015.01.01, ti<09:30:30
_
```

You can group by the partition column. Note we have wrapped the line for display.

```q
q)select hi:max p, lo:min p by date from t
where date within 2015.01.01 2015.01.02
_
```

### 14.3.5 The Virtual Column i in Partitioned Tables¶

`i`In a partitioned table, the virtual column i does not refer to absolute row number as it does with in-memory and splayed tables. Instead, it refers to the relative row number within a partition. Thus, a constraint on i alone would apply across all partitions and the result will contain that row in each partition slice â probably not what you want and almost certainly a bad idea (colleagues with pitchforks again).

`i``i`Continuing with the previous example, the following query retrieves the initial record from each of the specified partitions.

```q
q)select from t where date in 2015.01.01 2015.01.02, i=0
_
```

The following queries retrieve the first and last records in the table, respectively.

```q
q) select from t where date=first date, i=0
_
q) select from t where date=max date, i=max i
_
```

See Â§14.4.6 for using .Q.ind to index across partitions.

`.Q.ind`### 14.3.6 Query Execution on Partitioned Tables¶

Query execution against a partitioned table is more complex than execution against a splayed table. Understanding how it works is necessary to achieve good design and performance with kdb+ databases.

Recall that the motivation for partitions was to avoid loading entire columns into memory. Behind the scenes, kdb+ achieves this as follows.

- Analyze the where phrase to determine which partition slices are targeted by the query
- Process the remaining where sub-phrases to determine the column sub-domains that must be loaded.
- Process the query separately against the requisite partition slices to obtain partial results.
- Combine the partial results to obtain the final result.

Reading only partial column slices of a table having many partitions is clearly a big memory and performance win. But things are even more interesting with regard to the third point above. If you have started q without slaves, the query is executed sequentially against the partitions and the time to complete is (roughly) the sum of the time against each of the partition slices.

By contrast, if you have started q with slaves, the query will be executed concurrently, once slice per slave. Thus, you can see significant speedup provided the query is compute-bound and the slaves are running on separate cores. On the other hand, if the query is I/O bound and you have only a single I/O channel to the data, you may see no speedup.

Next we discuss how kdb+ executes some types of queries concurrently.

### 14.3.7 Map-Reduce¶

We start with a brief introduction to map-reduce. Suppose we want to compute some aggregation â say the sum or the average â of a large list of numbers and have at our disposal a crop of interns armed with smartphones. Our strategy is to decompose the overall task into two stages. In the first stage, we split the list into roughly (but not exactly) equal size sublists, distributing one to each intern along with instructions on what to compute. In the second step, we collect the (partial) results from each intern and combine them to get the overall result. This is what good managers do, after all.

Think about how to do this for sum before continuing. For the sum, you could instruct each intern to compute the sum of her list and then compute the sum of the individual sums. The same simple approach works for count or maximum.

For average, things aren't quite so simple. If you simply have each intern compute an average of her list, you will need to weight each partial average by the size of the sublist to get the final average. Thus you need the count too. You quickly realize that you can distribute the average computation more effectively. Instead of asking each intern to return the average and count for her list, ask her to compute the sum and the count. In the reduction step, you obtain the overall average as the sum of the partial sums divided by the sum of the partial counts.

More generally, map-reduce decomposes an operation on a (presumably large) list into two suboperations, opmap and opreduce. In the first step, called 'map', opmap is performed on each sublist to obtain a list of partial results. In the second step, called "reduce", the partial result lists are combined with opreduce to obtain the final result. A good exercise (and interview question) is to express sorting a list using map-reduce. In greater generality, map-reduce may apply the two steps recursively â i.e., the reduce step may itself involve a map-reduce, etc.

It is easy to see how map-reduce applies to a query against a partitioned table, since the table is a list of records sliced into sub-lists by the partitioning. The challenge is to decompose the query into a map step and a reduce step. The solution depends on whether the query involves aggregation.

If there is no aggregation, the result of the query on each partition is simply the computed columns for the list of the records in the partition slice matching the constraint. In other words, produce a partial result table by computing the columns of the query across partitions. Because all the partial result tables conform, union the partial result tables in order of their virtual partition column values. In summary: fan the query across the partitions and union the ordered results.

Things are more interesting when the query contains aggregation. For aggregates that kdb+ recognizes as map-reducible, it applies the map operation across partition slices to obtain partial results. It then applies the reduce operation across the partial result tables to obtain the final result table.

At the time of this writing (Sep 2015), the aggregates that kdb+ can decompose with map-reduce are: avg, cor, count, cov, dev, distinct, first, last, max, med, min, prd, sum, var, wavg, wsum.

`avg``cor``count``cov``dev``distinct``first``last``max``med``min``prd``sum``var``wavg``wsum`To make this more concrete, we re-examine computing an average against a daily partitioned trades table.

```q
select avg size from trade where date within â¦
```

Because kdb+ recognizes avg as an aggregate it can decompose with map-reduce, it performs the calculation in two steps. Specifically, opmap computes the sum and record count for each date, while opreduce divides the sum of the daily sums by the sum of the daily counts to give the average. The user gets the expected result and is none the wiser unless q was started with slaves, in which case the result maybe faster than linear.

`avg`### 14.3.8 Multiple Partitioned Tables¶

Recall that there can be only one partition domain in a given kdb+ root â i.e., daily, monthly, yearly or long. However, multiple tables can share this partitioning.

For example, quotes and trades can coexist in a daily partition. That layout would look something like,

```q
/db
    /2015.01.01
        /trade
        /quote
    /2015.01.02
        /trade
        /quote
```

Important

Although not all potential partition values need be populated, any value that is populated must contain slices for all tables.
The following layout is in error.

/db
    /2015.01.01 <- this is a bad partition!
        /quote
    /2015.01.02
        /trade
        /quote

```q
/db
    /2015.01.01 <- this is a bad partition!
        /quote
    /2015.01.02
        /trade
        /quote
```

We demonstrate how to create a simplified database with tables standing for trades and quotes in a fresh /db directory. Notice that we wrap the lines that do not fit within this page margin.

`/db````q
q)`:/db/2015.01.01/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; sym:`ibm`msft;p:101 33f)
_
q)`:/db/2015.01.02/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; sym:`ibm`msft;p:101.5 33.5)
_
q)`:/db/2015.01.01/q/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; sym:`ibm`msft;b:100.75 32.75; a:101.25 33.25f)
_
q)`:/db/2015.01.02/q/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:30:00; sym:`ibm`msft;b:101.25 33.25; a:101.75 33.75)
_
```

Mapping the root, we now find both are available.

```q
q)select from t where date within 2015.01.01 2015.01.02
_
q)select from q where date within 2015.01.01 2015.01.02
_
```

Next we add a historical slice for q on 2014.12.31 but neglect to add the corresponding slice for t. Things seem fine when we map the root and query q on the newly added date.

```q
q)`:/db/2014.12.31/q/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; sym:`ibm`msft; b:101. 33.; a:101.5 33.5f)
`:/db/2014.12.31/q/
q)\l /db
q)select from q where date=2014.12.31
_
```

But we get a nasty surprise when we query t on the missing date.

`t````q
q)select from t where date=2014.12.31
k){0!(?).@[x;0;p1[;y;z]]}
'./2014.12.31/t/ti: No such file or directory
.
?
(+`ti`sym`p!`:./2014.12.31/t;();0b;())
q.Q))
```

We could remedy this by splaying an empty copy of t on that date. Instead we use the utility .Q.chk that fills all missing slices with empty tables from the most recent partition. We remap and find things are fine.

`t``.Q.chk````q
q).Q.chk `:/db
_
q)\l /db
q)select from t where date=2014.12.31
_
```

Tip

If you neglect to place a table slice in the most recent partition, the table will effectively disappear from your database since kdb+ inspects only that partition to determine which tables are present.

Continuing with our previous (repaired) example we add a trade slice for 2015.01.03 but neglect to add a quotes slice. When we remap, the table q is nowhere to be found.

`q````q
q)`:/db/2015.01.03/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; sym:`ibm`msft;p:101 33f)
_
q)\l /db
q)\a
,`t
```

Here is one way to fill an empty slice.

```q
q)`:/db/2015.01.03/q/ set 0#select from `:/db/2015.01.02/q
_
q)\l /db
q)\a
`q`t
```

### 14.3.9 Examples of Other Partition Domain Types¶

We present simple examples of partitions with the other permissible types. For simplicity, we do not include symbol columns; to do so, merely add the call to .Q.en as in previous examples.

`.Q.en`Yearly partitioning can be a good solution when you regularly need to retrieve multiple years of daily data. Note that we explicitly include the date column but not the year column, which is virtual. In a fresh /db,

`/db````q
q)`:/db/2015/t/ set ([] date:2015.01.01 2015.01.02; p:101 102f)
`:/db/2015/t/
q)`:/db/2014/t/ set ([] date:2014.01.01 2014.01.02; p:101.5 102.5)
`:/db/2014/t/
q)\l /db
q)select from t where year within 2014 2015
_
```

Monthly partitioning can be a good compromise between daily and yearly when you have both single day and year-to-date queries. We explicitly include the date column but not the month column, which is virtual.

Tip

Do not include a trailing / in the partition directory name but do include it in the queries.

`/`In a fresh /db,

`/db````q
q)`:/db/2015.01/t/ set ([] date:2015.01.01 2015.01.02; p:101 102f)
`:/db/2015.01/t/
q)`:/db/2015.02/t/ set ([] date:2015.02.01 2015.02.02; p:101.5 102.5)
`:/db/2015.02/t/
q)\l /db
q)select from t where month within 2015.01m 2015.02m
_
```

You can partition by a long to slice into arbitrary bins. The data need not be daily. In a fresh /db,

`/db````q
q)`:/db/1/t/ set ([] ti:09:30:00 09:31:00; p:101 102f)
_
q)`:/db/2/t/ set ([] ti:09:30:00 09:31:00; p:101.5 102.5)
_
q)\l /db
q)select from t where i within 1 2
_
```

### 14.3.10 Partitioned Tables with Links¶

You can create links between tables in the same partitioned database with the following restriction.

Note

A link column in the slice of a partitioned table must be intra-partition â i.e., it must refer to another table in the same slice. In particular, you cannot link across days in a daily partitioned database.

Since the daily slices are splayed tables, the mechanics of creating links for partitioned tables are the same as for splayed tables. We first demonstrate how to create a link as the slices are created in memory and splayed. Again we avoid symbols for simplicity; to include them, add a call to .Q.en as in previous examples.

`.Q.en`We create two days of linked tables in a fresh /db.

`/db````q
q)t1:([] id:101 102 103; v:1.1 2.2 3.3)
q)`:/db/2015.01.01/t1/ set t1
_
q)`:/db/2015.01.01/t2/ set ([] id:`t1!t1[`id]?103 101 101 102; n:10 20 30 40)
_
q)`:/db/2015.01.02/t1/ set t1
_
q)`:/db/2015.01.02/t2/ set ([] id:`t1!t1[`id]?105 104 104; n:50 60 70)
_
```

Now we map the root and use dot notation on the link column.

```q
q)\l /db
q)select date,n,id.v from t2 where date in 2015.01.01 2015.01.02
_
```

Adding a link column to persisted partitions is only a bit more complicated. Here we create a link on (enumerated) symbol columns. In a fresh /db we first create the partitioned tables without the link.

`/db````q
q)`:/db/2015.01.01/t1/ set .Q.en[`:/db;] ([] id:`c`b`a; c1: 10 20 30)
_
q)`:/db/2015.01.02/t1/ set .Q.en[`:/db;] ([] id:`x`a; c1: 40 50)
_
q)`:/db/2015.01.01/t2/ set .Q.en[`:/db;] ([]id:`a`b`a`c; c2: 1 2 3 4.)
_
q)`:/db/2015.01.02/t2/ set .Q.en[`:/db;] ([] id:`x`a`x; c2:5 6 7.)
_
```

Presumably at a later time, we add the link column to t2 in each partition, making sure to update the .d files.

`t2``.d````q
q)`:/db/2015.01.01/t2/t1lnk set
`t1!get[`:/db/2015.01.01/t1/id]?get[`:/db/2015.01.01/t2/id]
_
q)`:/db/2015.01.01/t2/.d set get[`:/db/2015.01.01/t2/.d],`t1lnk
_
q)`:/db/2015.01.02/t2/t1lnk set
`t1!get[`:/db/2015.01.02/t1/id]?get[`:/db/2015.01.02/t2/id]
_
q)`:/db/2015.01.02/t2/.d set get[`:/db/2015.01.02/t2/.d],`t1lnk
_
```

Now we can map the root and use dot notation.

```q
q)select date,id,t1lnk.c1,c2 from t2 where date<=2015.01.02
_
```

## 14.4 Segmented Tables¶

We saw in the previous section that queries against partitioned tables execute separately across the partitions. When q is started without slaves this is a memory win but not a performance win, except for the reduced amount of data manipulation required. When q is started with slaves this can offer significant performance enhancement for compute-bound queries.

In large timeseries databases, the queries are often I/O-bound. In this case, the multiple slaves for a partitioned query will mostly be waiting on I/O (or Godot). The solution requires multiple I/O channels so that data retrieval and processing can occur in parallel. Kdb+ provides another level of data decomposition to enable parallel processing in this scenario.

Traditional kdb+ parlance did not distinguish between this level and the basic partitioning we encountered in the last section. We find this confusing and, at the suggestion of Simon Garland of KX, have introduced the terminology of segmentation.

### 14.4.1 Segments¶

Segmentation is an additional level of structure on top of partitioning. Segmentation spreads a partitioned tableâs records across multiple directories that have the same structure as the root directory in a partitioned database. Each pseudo-root, called a segment, is thus a directory that contains a collection of partition directories. The segment directories are presumably on independent I/O channels so that data retrieval can occur in parallel.

You can use any criteria to decompose partition slices, as long as the results are conforming record subsets that are disjoint and complete â i.e., they reconstitute the original table with no omissions or duplication. The decomposition can be along rows, along partitions or by some combination thereof, but it cannot occur only along columns since all records must conform across the decomposition.

Important

You must ensure that the segments conform and are complete and disjoint, since kdb+ will not check this when you write the data files. In particular, overlapping segments will result in duplicate records in query results and an incomplete decomposition will result in dropped records.

Big Picture (3): .We view a segmented table as a three-dimensional persisted form: the table is cut vertically by splaying, sliced horizontally by partitions and is additionally segmented across physical locations. The primary purpose of the third dimension is to allow operations against the tables to take advantage of parallel I/O and concurrent processing. Following is an abstract representation of segmentation:

segment1

segment2

â¦

Here each âª represents a (partial) column of a partition slice as an opaque entity.

In contrast to the partitioned table layout in which partitions reside under the root, the segment directories must not reside under the root. The only portion of a segmented table (other than the sym file for enumerated symbol columns) that lives in the root is a file called par.txt containing the paths of the physical locations of the segments, one segment path per line.

`par.txt`Here is how the abstract segmentation discussed above would be laid out on the file system.

```q
/db
    [sym]
    par.txt
=============== <- channel 1
/segment1
    /partition*
        /table*
        /table*
        â¦
    /partition*
        /table*
        /table*
        â¦
=============== <- channel 2
/segment2
    /partition*
        /table*
        /table*
        â¦
    /partition*
        /table*
        /table*
        â¦
=============== â¦
```

To make this more concrete, we demonstrate how to segment daily trades (and eventually quotes) in several useful ways. To begin, we create segments by bucketing trades into alternating days of the week.

```q
/1                  <- drive 1
    /2015.01.01
        /t          <- dayâs trades
    /2015.01.03
        /t          <- dayâs trades
    â¦
=============
/2                  <- drive 2
    /2015.01.02
        /t          <- dayâs trades
    /2015.01.04
        /t          <- dayâs trades
    â¦
```

This segmentation represents grouping of partitions, so it is orthogonal to the partitioning. It is clearly complete and disjoint and is easily generalizable to round-robining every n business days.

We could alternatively create segments by splitting the daily slices into records with symbols starting with a-m and those starting with n-z. Here we are decomposing based on the values in a symbol column, which we could not do with simple partitioning.

```q
/am                 <- drive 1
    /2015.01.01
        /t          <- dayâs trades for syms a-m
    /2015.01.02
        /t          <- dayâs trades for syms a-m
    â¦
=============
/nz <- drive 2
    /2015.01.01
        /t          <- dayâs trades for syms n-z
    /2015.01.02
        /t          <- dayâs trades for syms n-z
    â¦
```

This segmentation clearly results in complete, disjoint subsets. It is not orthogonal to the partitioning because a single day's trades span multiple segments. It is easily generalizable to n segments by splitting the alphabet into n roughly equal portions.

Alternately, we can create segments by splitting the daily slices into trades from NYSE and trades from NASDAQ.

```q
/nyse               <- drive 1
    /2015.01.01
        /t          <- dayâs trades for nyse
    /2015.01.02
        /t          <- dayâs trades for nyse
    â¦
=============
/nasd               <- drive 2
    /2015.01.01
        /t          <- dayâs trades for nasd
    /2015.01.02
        /t          <- dayâs trades for nasd â¦
```

The segmentation is also not orthogonal to the partitioning since a single day's trades span multiple segments. It is clearly complete and disjoint and is easily generalizable to multiple exchanges â e.g., all Asian exchanges.

Finally, we provide an example of a non-uniform segmentation, in which some partitions span segments and others do not. The A segment contains trades from the beginning of day 1 until lunchtime of day 2. The B segment contains the trades from lunchtime of day 2 through the end of day 3. The C segment contains all trades from day 4. We rely on our understanding of the business logic to know that segmentation is disjoint and complete.

```q
/seg A              <- drive 1
    /2015.01.01
        /t          <- entire dayâs trades
    /2015.01.02
        /t          <- morning trades
=============
/seg B <- drive 2
    /2015.01.02
        /t          <- afternoon trades
    /2015.01.03
        /t          <- entire dayâs trades
=============
/seg C <- drive 3
    /2015.01.04
        /t          <- entire dayâs trades
```

This sort of segmentation can be useful if your query patterns are not uniform across days.

### 14.4.2 Segmentation vs. Partitions¶

We collect the following into tabular form so that you can use it for meditation.

### 14.4.3 Creating Segmented Tables¶

There is no one-size-fits-all utility to create segments. Instead, you write a q program that places a subset of each partition slice into a segment. You can create segments and partitions at the same time by including logic in your data load script to extract slice subsets and place them in the appropriate directory on the appropriate drive.

Along with creating the partitions, you must also specify the locations of the segments in an ASCII text file par.txt located in the root. Each line of par.txt contains the path of one segment directory; symlinks are acceptable.

`par.txt``par.txt`The segment paths must not be under the root. Qbies who make this mistake will prefer Vogon poetry.

We illustrate the process by creating segmented tables for each example in Â§14.4.1. Our intent is to demonstrate how to create the segments and observe how execution of the query pieces the segment results back together. Initially we create the segments without concern for placing them on multiple drives and running multiple slaves, as we are not (yet) considering performance. That comes shortly.

We begin with the example in which daily slices are segmented alternately across two bins. This will result in roughly even access and will give good performance assuming the queries are uniformly distributed across days. The storage layout looks like,

```q
/1
    /2015.01.01
        /t
    /2015.01.03
        /t
/2
    /2015.01.02
        /t
    /2015.01.04
        /t
```

The corresponding par.txt file is,

`par.txt````q
/1
/2
```

In a fresh directories /1, /2 and /db we construct the segments and write the par.txt file.

`/1``/2``/db``par.txt````q
q)`:/1/2015.01.01/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; s:`ibm`t; p:101 17f)
_
q)`:/2/2015.01.02/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; s:`ibm`t; p:101.5 17.5)
_
q)`:/1/2015.01.03/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; s:`ibm`t; p:103 16.5f)
`:/1/2015.01.03/t/
q)`:/2/2015.01.04/t/ set .Q.en[`:/db;]
 ([] ti:09:30:00 09:31:00; s:`ibm`t; p:102 17f)
_

q)`:/db/par.txt 0: ("/1"; "/2")
_

q)\l /db
q)select from t where date within 2015.01.01 2015.01.04
_
```

Next we segment trades by symbol range. The storage layout looks like,

```q
/am
    /2015.01.01
    /2015.01.02
/nz
    /2015.01.01
    /2015.01.02
```

The corresponding par.txt file is,

`par.txt````q
/am
/nz
```

We begin with fresh directories /am, /an, and /db and a utility function to extract records for symbols within a range. Then we create the segments and write out the par.txt file.

`/am``/an``/db``par.txt````q
q)extr:{[t;r] select from t where (`$1#'string sym) within r}

q)t1:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`t; p:101 17f)
q)`:/am/2015.01.01/t/ set extr[t1;`a`m]
_
q)`:/nz/2015.01.01/t/ set extr[t1;`n`z]
_

q)t2:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`t; p:101.5 17.5)
q)`:/am/2015.01.02/t/ set extr[t2;`a`m]
_
q) `:/nz/2015.01.02/t/ set extr[t2;`n`z]
_

q)`:/db/par.txt 0: ("/am"; "/nz")
`:/db/par.txt

q)\l /db
q)select from t where date within 2015.01.01 2015.01.02
_
```

Next we segment trades by exchange. The storage layout looks like,

```q
/nyse
    /2015.01.01
        /t
    /2015.01.02
        /t
/nasd
    /2015.01.01
        /t
    /2015.01.02
        /t
```

The corresponding par.txt file is,

`par.txt````q
/nyse
/nasd
```

Segment construction is completely analogous to that of the previous example. We begin with fresh directories /nyse, /nasd, /db and a utility to extract records by exchange.

`/nyse``/nasd``/db````q
q)extr:{[t;e] select from t where ex=e}

q)t1:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; s:`ibm`aapl; p:101 17f;ex:`n`o)
q)`:/nyse/2015.01.01/t/ set extr[t1;`n]
_
q)`:/nasd/2015.01.01/t/ set extr[t1;`o]
_

q)t2:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; s:`aapl`ibm; p:143 102f; ex:`o`n)
q)`:/nyse/2015.01.02/t/ set extr[t2;`n]
_
q)`:/nasd/2015.01.02/t/ set extr[t2;`o]
_

q)`:/db/par.txt 0: ("/nyse"; "/nasd")
_

q)\l /db
q)select from t where date within 2015.01.01 2015.01.02
_
```

Finally we show how to distribute a day across segments. The storage layout looks like,

```q
/seg A
    /2015.01.01
    /2015.01.02
/seg B
    /2015.01.02
    /2015.01.03
/seg C
    /2015.01.04
```

The corresponding par.txt file is,

`par.txt````q
/A
/B
/C
```

In fresh directories /A, /B, /C and /db,

`/A``/B``/C``/db````q
q) t1:.Q.en[`:/db;] ([] ti:09:30:00 12:31:00; s:`ibm`t; p:101 17f)
q)`:/A/2015.01.01/t/ set t1
_
q)t2:.Q.en[`:/db;] ([] ti:09:31:00 12:32:00; s:`ibm`t; p:102 18f)
q)`:/A/2015.01.02/t/ set select from t2 where ti<=12:00:00
_
q)`:/B/2015.01.02/t/ set select from t2 where ti>12:00:00
_
q)t3:.Q.en[`:/db;] ([] ti:09:33:00 12:33:00; s:`ibm`t; p:103 19f)
q)`:/B/2015.01.03/t/ set t3
_
q)t4:.Q.en[`:/db;] ([] ti:09:34:00 12:35:00; s:`ibm`t; p:104 20f)
q)`:/C/2015.01.04/t/ set t4
_

q)`:/db/par.txt 0: ("/A";"/B";"/C")
_

q)\l /db
q)select from t where date within 2015.01.01 2015.01.04
_
```

### 14.4.4 Multiple Segmented Tables¶

Multiple tables that share a partition can also be segmented. While there is no requirement that the tables be distributed similarly across the segmentation, they should do so if you expect to use links or joins between them. In this section, we create (vastly simplified) trades and quotes that share a segmentation in order to do joins.

Our actual segmentation layout is designed to allow parallel retrieval for aj. We first observe that it performs an equijoin on the sym and date columns, while it performs a non-equijoin on the time column. Thus we want all the time values for a given symbol and date in one segment. Consequently, segmentation by symbol range will allow parallelization of aj across symbols.

`aj``aj`Here is a simple version of this scheme.

```q
/a_m <- segment for first portion of alphabet
    /2015.01.01     <- the specific day
        /t          <- that dayâs trades for symbols a-m
        /q          <- that dayâs quotes for symbols a-m
    /2015.01.02     <- the specific day
        /t          <- that dayâs trades for symbols a-m
        /q          <- that dayâs quotes for symbols a-m
=================
/n_z                <- segment for second portion of alphabet
    /2015.01.01     <- the specific day
        /t          <- that dayâs trades for symbols n-z
        /q          <- that dayâs quotes for symbols n-z
    /2015.01.02     <- the specific day
        /t          <- that dayâs trades for symbols n-z
        /q          <- that dayâs quotes for symbols n-z
=================
```

The corresponding par.txt file is,

`par.txt````q
/a_m
/n_z
```

Setting up the segments and partitions is a matter of getting the details right. In our simplified example, we pretend that the directories /amand /nz reside on different drives. In fresh directories /am, /nz and /db,

`/am``/nz``/am``/nz``/db````q
q)extr:{[t;r] select from t where (`$1#'string sym) within r}
q)/ day 1
q)t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`t; p:101 17f)
q)q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`ibm`t`ibm; p:100.5 17 101)
q)`:/am/2015.01.01/t/ set extr[t;`a`m]
_
q)`:/nz/2015.01.01/t/ set extr[t;`n`z]
_
q)`:/am/2015.01.01/q/ set extr[q;`a`m]
)
q)`:/nz/2015.01.01/q/ set extr[q;`n`z]
_

q)/ day 2
q)t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`t`ibm; p:17.1 100.9)
q)q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`t`ibm`t;p:17 100.8 17.1)
q)`:/am/2015.01.02/t/ set extr[t;`a`m]
_
q)`:/nz/2015.01.02/t/ set extr[t;`n`z]
_
q)`:/am/2015.01.02/q/ set extr[q;`a`m]
_
q)`:/nz/2015.01.02/q/ set extr[q;`n`z]
_
q)`:/db/par.txt 0: ("/am"; "/nz")
_

q)\l /db

q)dr:2015.01.01 2015.01.02
q)select from t where date within dr
_
q)select from q where date within dr
_
```

Now letâs do the aj. We could be naÃ¯ve and pull all requisite data into the workspace, but this would be inefficient in memory and slow, assuming it would fit.

`aj````q
q)aj[`date`sym`ti;select from t where date within dr;
 select from q where date within dr]
_
```

Instead we start q with slaves and use peach to run things in parallel across the segmentation.

`peach````q
$q -s 2
KDB+ ...
```

```q
q)aj1:{aj[`sym`ti;select from t where date=d; select from q where date=d]}
q)raze aj1 peach 2015.01.01 2015.01.02
_
```

### 14.4.5 Query Execution against Segmented Tables¶

The performance goal is to scale out by taking advantage of parallel I/O and concurrent processing. We would ideally like to achieve 100% saturation of the I/O channels and 100% utilization of each core. How do we approach these levels on a kdb+ server? The key insight in kdb+ design and tuning is that a vector calculation on in-memory data is much faster than retrieving data from storage. This suggests our first two design principles.

- Maximize the number of independent I/O channels to retrieve data in parallel.
- Maximize server memory in order to allocate each slave thread as much memory as it needs.

Maximize the number of independent I/O channels to retrieve data in parallel.

Maximize server memory in order to allocate each slave thread as much memory as it needs.

Suppose we can satisfy these two objectives by having storage devices attached to the kdb+ server over n independent I/O channels â for example, the authorâs laptop has SSD storage that appears as multiple independent I/O channels. In this scenario, we are led to our next observation.

- Create n segments to spread data retrieval across the n channels in order to maximize I/O parallelization.

To ensure that data can be processed from all n channels simultaneously and that no two threads contend for data, we are led to our final objective.

- Have (at least) n slave threads

The precise form of segmentation and number of slaves will depend on the actual data and queries.

Now assuming we have such an environment, does kdb+ execute a query against a segmented table in our scenario of n segments and n slaves.? Essentially, it decomposes a qualifying query into two steps via map-reduce:

- Map: a revised form of the original query that executes on each segment
- Reduce: aggregate the segment results

The use of map-reduce allows kdb+ to perform preliminary calculations as close to the data as possible and to perform aggregation centrally at the last step.

To begin, kdb+ compares the queryâs requisite partitions to the segment layout in par.txt and determines the footprint of the target partitions on each segment. The result is a nested list, each item being the partition list for one segment.

`par.txt`To execute the map step, kdb+ creates a revised query containing the map sub-operation from the original query, which it dispatches to all n slaves via peach. Each slave is provided the partition list for one segment and computes the revised query for its segment. For example, the revised query for avg is:

`peach``avg`Compute the sum and count of the sublist

With this knowledge, we examine execution within one slave, where the revised query is applied against a segmentâs partition footprint. Here kdb+ sequentially applies the map sub-operations of the original query across the targeted partitions to obtain partition result tables that it then collects into a list representing one segment result.

Now we stand back and examine execution across the slaves by squinting to make partition detail disappear. At this level, the original queryâs map step has n slaves retrieving segment data in parallel and calculating segment results. Once all slaves complete, the nested list of segment results is flattened and reordered by partition value.

Finally, kdb+ employs the original query reduce step to combine the full list of ordered partition results into the query result table. Whew!

Note

Kdb+ treats a vanilla partitioned table â i.e., without a par.txt â as having a single segment. The astute reader will realize that the description in Â§14.3.3 is actually the degenerate case of this section.

`par.txt`### 14.4.6 Balancing Slaves and Cores¶

In our quest to reach 100% saturation of I/O and CPU, we consider how to optimize the use of slaves and cores. As seen in the previous section, the query via map-reduce provides good progress toward our original objective of I/O saturation. The slaves can load data from all n channels in parallel without contention.

We now investigate channel and core utilization. Since kdb+ will only use as many slaves to process a query as there are segments in the query footprint, we consider two cases.

I/O-bound: Assuming that the query has light calculation compared to data retrieval (common in kdb+), having n slaves on n cores is close to optimal: most of the time, all n slaves will be waiting for data. When a partition load completes, there will be a brief burst of computation, followed by another long load. So we conclude for this scenario:

n channels => n segments => n slaves => n cores

Balanced I/O-compute: Consider the scenario in which both the I/O and calculation are intensive. While one slave is waiting on data, another slave on the same core could be crunching; conversely, while one slave is crunching another slave on that core could be loading data. Thus to maximize channel and core utilization, we actually want 2n slaves on n cores. We conclude that in this scenario we should have 2n segments, two per channel. On average, there will be one slave per core loading data and one slave per core crunching the data it has just loaded.

n channels => 2n segments => 2n slaves => n cores

These conclusions rely on many implicit assumptions that we have glossed over. In practice, you should view them as guidelines, with the goal of feeding data to kdb+ as fast as possible. The optimum configuration for your situation will depend on your particular query mix. For example, queries that do VWAP calculations are closer to the first scenario, whereas queries doing regression analysis are closer to the second.

A good strategy is to construct your initial configuration using one of the above scenarios. Then load a good approximation of your anticipated data and query mix, and simulate a realistic user load. Observe the I/O saturation and CPU utilization and adjust the number of slaves and cores allocated to the q process accordingly.

### 14.4.7 Sample Performance Data¶

Our examples are based on simulated trade data randomly generated to match one month of US equity data spanning August and September 2014. The data is in the table trade, whose most significant columns time, sym, tp and ts represent respectively arrival time, instrument symbol, trade price and trade size columns. The trade data is partitioned by date.

`trade``time``sym``tp``ts`The following output is generated from a version of kdb+ that has been instrumented to show the intermediate steps during query execution. The first example query shows how the simple aggregate avg tp is decomposed into the pair sum and count in the map step, followed by division of the sums of sums by the sum of counts for the reduce step. Here the data is not segmented â i.e., there is no par.txt file.

`avg tp``par.txt`The added instrumentation exposes 3 stages of execution of a query.

- Analyze query:


Decompose query into map and reduce components (if appropriate)


Determine and apply partition constraints


Map query onto segments and partitions (query plan)
- Execute map step, if appropriate
- Compute final result (reduce step)

Analyze query:

- Decompose query into map and reduce components (if appropriate)
- Determine and apply partition constraints
- Map query onto segments and partitions (query plan)

Decompose query into map and reduce components (if appropriate)

Determine and apply partition constraints

Map query onto segments and partitions (query plan)

Execute map step, if appropriate

Compute final result (reduce step)

```q
q)select avg tp from trade where date in -3#date
"--- map/reduce: input aggregates, map query, reduce query ---" (,`tp)!,(avg;`tp)
`0`1!((sum;($["f"];`tp));(#:;`i))
(,`tp)!,(%;(sum;`0);(sum;`1))

"--- query plan: segments/partitions ---" (`trade;();()!();`0`1!((sum;($["f"];`tp));(#:;`i)))
2014.09.24 2014.09.25 2014.09.28

"--- partition query: query, segment, partition, result ---" (`trade;();()!();`0`1!((sum;($["f"];`tp));(#:;`i)))
`:.
2014.09.24
+`0`1!(,1.419538e+10;,27914938)

"--- partition query: query, segment, partition, result ---" (`trade;();()!();`0`1!((sum;($["f"];`tp));(#:;`i)))
`:. 2014.09.25 +`0`1!(,1.419318e+10;,24485503)

"--- partition query: query, segment, partition, result ---" (`trade;();()!();`0`1!((sum;($["f"];`tp));(#:;`i)))
`:. 2014.09.28 +`0`1!(,1.388645e+10;,20162485)

"--- query plan result ---" (+`0`1!(,1.419538e+10;,27914938);+`0`1!(,1.419318e+10;,24485503); +`0`1!(,1.388645e+10;,20162485))

"--- final result ---" tp -------- 582.5979
```

First note that `0 and `1 are used as intermediate columns. Observe that avg tp is decomposed into a map step.

``0```1``avg tp````q
`1:sum "f"$tp and `0:count i
```

and a reduce step

```q
tp:(sum `0)%sum `1.
```

Also observe that expressions are displayed in their parse-tree format and that count is expressed as its k equivalent: the unary form of #.

`#`The next example uses par.txt to segment the data into four segments: (/d/d1/data, /d/d2/data, /d/d3/data, /d/d4/data). It also uses a by-date clause to sidestep the need to break up the query into map and reduce components (our data is partitioned by date). For clarity, we execute the query only on the last 16 partitions as well as on a subset of symbols. Observe that the "query plan" contains more information than that of the previous example: the query, a list of all partitions, a list of all segments, and a nested list of partitions belonging to segments.

`par.txt``/d/d1/data``/d/d2/data``/d/d3/data``/d/d4/data`q)select avg tp by date from trade where date in -16#date,sym in syms

"--- query plan: segments/partitions ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) 2014.08.18 2014.08.19 2014.08.20 2014.08.21 2014.08.24 2014.08.25 2014.08.26 2014.08.27 2014.08.28 2014.08.31 2014.09.01 2014.09.02 2014.09.03 2014.09.04 2014.09.08 2014.09.09 `:/d/d1/data`:/d/d2/data`:/d/d3/data`:/d/d4/data ((`:/d/d1/data;2014.08.21 2014.08.25 2014.09.02); (`:/d/d2/data;2014.08.18 2014.08.26 2014.09.03); (`:/d/d3/data;2014.08.19 2014.08.27 2014.08.31 2014.09.04 2014.09.08); (`:/d/d4/data;2014.08.20 2014.08.24 2014.08.28 2014.09.01 2014.09.09))

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.21 +(,`tp)!,,15.42632

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.25
+(,`tp)!,,15.04996

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.09.02
+(,`tp)!,,14.16648

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d2/data 2014.08.18
+(,`tp)!,,14.16883
â¦ (some output removed for brevity) â¦
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.09.08
+(,`tp)!,,15.59198

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.08.20
+(,`tp)!,,15.2657

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.08.24 +(,`tp)!,,14.75603

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.08.28
+(,`tp)!,,14.37194

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.09.01
+(,`tp)!,,13.25797

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.09.09
+(,`tp)!,,14.98316

"--- query plan result ---" (+(,`tp)!,,14.16883;+(,`tp)!,,15.05272;+(,`tp)!,,15.2657;+(,`tp)!,,15.42632; +(,`tp)!,,14.75603;+(,`tp)!,,15.04996;+(,`tp)!,,15.69218;+(,`tp)!,,15.53095; +(,`tp)!,,14.37194;+(,`tp)!,,14.32488;+(,`tp)!,,13.25797;+(,`tp)!,,14.16648; +(,`tp)!,,15.58938;+(,`tp)!,,16.1427;+(,`tp)!,,15.59198;+(,`tp)!,,14.98316)

"--- final result ---"
date      | tp
----------| --------
2014.08.18| 14.16883
2014.08.19| 15.05272
2014.08.20| 15.2657
2014.08.21| 15.42632
2014.08.24| 14.75603
2014.08.25| 15.04996
2014.08.26| 15.69218
2014.08.27| 15.53095
2014.08.28| 14.37194
2014.08.31| 14.32488
2014.09.01| 13.25797
2014.09.02| 14.16648
2014.09.03| 15.58938
2014.09.04| 16.1427
2014.09.08| 15.59198
2014.09.09| 14.98316

```q
q)select avg tp by date from trade where date in -16#date,sym in syms

"--- query plan: segments/partitions ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) 2014.08.18 2014.08.19 2014.08.20 2014.08.21 2014.08.24 2014.08.25 2014.08.26 2014.08.27 2014.08.28 2014.08.31 2014.09.01 2014.09.02 2014.09.03 2014.09.04 2014.09.08 2014.09.09 `:/d/d1/data`:/d/d2/data`:/d/d3/data`:/d/d4/data ((`:/d/d1/data;2014.08.21 2014.08.25 2014.09.02); (`:/d/d2/data;2014.08.18 2014.08.26 2014.09.03); (`:/d/d3/data;2014.08.19 2014.08.27 2014.08.31 2014.09.04 2014.09.08); (`:/d/d4/data;2014.08.20 2014.08.24 2014.08.28 2014.09.01 2014.09.09))

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.21 +(,`tp)!,,15.42632

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.25
+(,`tp)!,,15.04996

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.09.02
+(,`tp)!,,14.16648

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d2/data 2014.08.18
+(,`tp)!,,14.16883
```

```q
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.09.08
+(,`tp)!,,15.59198

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.08.20
+(,`tp)!,,15.2657

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.08.24 +(,`tp)!,,14.75603

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.08.28
+(,`tp)!,,14.37194

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.09.01
+(,`tp)!,,13.25797

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data 2014.09.09
+(,`tp)!,,14.98316

"--- query plan result ---" (+(,`tp)!,,14.16883;+(,`tp)!,,15.05272;+(,`tp)!,,15.2657;+(,`tp)!,,15.42632; +(,`tp)!,,14.75603;+(,`tp)!,,15.04996;+(,`tp)!,,15.69218;+(,`tp)!,,15.53095; +(,`tp)!,,14.37194;+(,`tp)!,,14.32488;+(,`tp)!,,13.25797;+(,`tp)!,,14.16648; +(,`tp)!,,15.58938;+(,`tp)!,,16.1427;+(,`tp)!,,15.59198;+(,`tp)!,,14.98316)

"--- final result ---"
date      | tp
----------| --------
2014.08.18| 14.16883
2014.08.19| 15.05272
2014.08.20| 15.2657
2014.08.21| 15.42632
2014.08.24| 14.75603
2014.08.25| 15.04996
2014.08.26| 15.69218
2014.08.27| 15.53095
2014.08.28| 14.37194
2014.08.31| 14.32488
2014.09.01| 13.25797
2014.09.02| 14.16648
2014.09.03| 15.58938
2014.09.04| 16.1427
2014.09.08| 15.59198
2014.09.09| 14.98316
```

The previous example was run with no slaves â i.e., on a single-threaded q process. Consequently, the queries run sequentially across segments and partitions.

Now observe what happens when we match the number of slaves to the number of segments in our database by invoking q with -s 4.

`-s 4````q
q)select avg tp by date from trade where date in -16#date,sym in syms

"--- query plan: segments/partitions ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) 2014.08.18 2014.08.19 2014.08.20 2014.08.21 2014.08.24 2014.08.25 2014.08.26 2014.08.27 2014.08.28 2014.08.31 2014.09.01 2014.09.02 2014.09.03 2014.09.04 2014.09.08 2014.09.09 `:/d/d1/data`:/d/d2/data`:/d/d3/data`:/d/d4/data ((`:/d/d1/data;2014.08.21 2014.08.25 2014.09.02); (`:/d/d2/data;2014.08.18 2014.08.26 2014.09.03); (`:/d/d3/data;2014.08.19 2014.08.27 2014.08.31 2014.09.04 2014.09.08); (`:/d/d4/data;2014.08.20 2014.08.24 2014.08.28 2014.09.01 2014.09.09))

"--- partition query: query, segment, partition, result ---"
"--- partition query: query, segment, partition, result ---"
"--- partition query: query, segment, partition, result ---"
(`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.21

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.08.20 (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.08.19 (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d2/data
2014.08.18
+(,`tp)!,,15.55121

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.08.27
+(,`tp)!,,15.47055 +(,`tp)!,,15.21819

"--- partition query: query, segment, partition, result ---"
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d2/data
2014.08.26 (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.08.24
+(,`tp)!,,14.81711
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d1/data
2014.08.25
+(,`tp)!,,14.92875

"--- partition query: query, segment, partition, result ---" +(,`tp)!,,16.07275
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.08.31
+(,`tp)!,,15.55499 (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.08.28

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d2/data
2014.09.03
+(,`tp)!,,13.43061 +(,`tp)!,,15.29159 +(,`tp)!,,12.64993

"--- partition query: query, segment, partition, result ---"
"--- partition query: query, segment, partition, result ---"
"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
`:/d/d4/data
2014.09.04
2014.09.01
+(,`tp)!,,176.6311
`:/d/d1/data
2014.09.02
+(,`tp)!,,151.7784
+(,`tp)!,,13.67089

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d4/data
2014.09.09
+(,`tp)!,,179.799

"--- partition query: query, segment, partition, result ---" (`trade;,(in;`sym;`syms);(`symbol$())!`symbol$();(,`tp)!,(avg;`tp)) `:/d/d3/data
2014.09.08
+(,`tp)!,,193.7031
+(,`tp)!,,48.75286

"--- query plan result ---" (+(,`tp)!,,15.47055;+(,`tp)!,,15.55121;+(,`tp)!,,15.21819;+(,`tp)!,,14.81711; +(,`tp)!,,16.07275;+(,`tp)!,,15.29159;+(,`tp)!,,15.55499;+(,`tp)!,,14.92875; +(,`tp)!,,12.64993;+(,`tp)!,,13.43061;+(,`tp)!,,13.67089;+(,`tp)!,,151.7784; +(,`tp)!,,176.6311;+(,`tp)!,,179.799;+(,`tp)!,,193.7031;+(,`tp)!,,48.75286)

"--- final result ---"
date      | tp
----------| --------
2014.08.18| 15.47055
2014.08.19| 15.55121
2014.08.20| 15.21819
2014.08.21| 14.81711
2014.08.24| 16.07275
2014.08.25| 15.29159
2014.08.26| 15.55499
2014.08.27| 14.92875
2014.08.28| 12.64993
2014.08.31| 13.43061
2014.09.01| 13.67089
2014.09.02| 151.7784
2014.09.03| 176.6311
2014.09.04| 179.799
2014.09.08| 193.7031
2014.09.09| 48.75286
```

Here the output from four slaves running concurrently is interleaved. One slave executes for each segment, with each slave executing the query on its segment sequentially across its partitions. We see how an appropriately configured server can take advantage of available I/O bandwidth to speed up query execution using segments and slaves.

## 14.5 Utilities for Splaying and Partitioning¶

The .Q namespace contains useful functions for creating and maintaining splayed and partitioned tables. Although KX disclaims support for customer use, nearly everyone uses them and they have become de facto standard. In this section, we describe the more commonly used entries and demonstrate their use.

`.Q`### 14.5.1 .Q.qp¶

`.Q.qp`The unary .Q.qp asks the residency of its table argument. It returns 1b if its argument is a partitioned table mapped into memory, 0b if it is splayed and 0 for anything else. In a fresh /db,

`.Q.qp``1b``0b``/db````q
q)`:/db/tser set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)`:/db/tsplay/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)`:/db/2015.01.01/tpart/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)\l /db
q).Q.qp tser
0
q).Q.qp tsplay
0b
q).Q.qp tpart
1b
```

### 14.5.2 .Q.en¶

`.Q.en`The binary .Q.en takes a symbolic file handle of a root directory as its first argument and a table as its second argument. As a side effect, it creates a list sym comprising the unique items across all symbol columns and writes it to an eponymous file in the specified directory. It returns a table obtained from the original table by enumerating its symbol column(s) over sym.

`.Q.en`In a fresh /db,

`/db````q
q)sym
'sym
q).Q.en[`:/db;] ([] s:`a`b`v;v:10 20 30)
s v
----
a 10
b 20
v 30
q)sym
`a`b`v
q)get `:/db/sym
`a`b`v
```

Here is more detail on the actual sequence of operations for .Q.en.

`.Q.en`- The variable sym is created (in memory) by loading the file sym from the specified root, should such exist, or as the empty symbol list if not. An existing sym variable in the workspace is overwritten in this step.
- All symbol columns of (a copy of) the table are conditionally enumerated over the sym list.
- The sym variable is serialized to the specified root directory.
- The enumerated table is returned from the function application.

`sym``sym``sym``sym`### 14.5.3 .Q.pv¶

`.Q.pv`The variable .Q.pv is a list containing the values of the partition domain â i.e., the values corresponding to the slice directories actually found in the root. This is useful if you need to iterate over all the partitions.

`.Q.pv`In a fresh /db,

`/db````q
q)`:/db/2015.01.01/t/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)`:/db/2015.01.03/t/ set ([] c1:4 5; 4.4 5.5)
`_
q)\l /db
q).Q.pv
2015.01.01 2015.01.03
```

### 14.5.4 .Q.ind¶

`.Q.ind`As we saw in Â§14.3, the virtual column i reflects the relative row number in a partitioned table. That is, the value of i is the offset of the records within the partition slice. How to retrieve records by absolute within a partition or segmented?

`i``i`A masochist might use a q expression to determine the partition and relative row for the absolute row number. It is less painful to use the binary .Q.ind, whose first argument is a partitioned table and whose second argument is a list of long values representing absolute row numbers. The result is a table in memory. You must enlist a single index value.

`.Q.ind`In a fresh /db,

`/db````q
q)`:/db/2015.01.01/t/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)`:/db/2015.01.03/t/ set ([] c1:4 5; c2:4.4 5.5)
_
q)\l /db
q)select from t where date within 2015.01.01 2015.01.03
_
q).Q.ind[t;1 3]
date c1 c2
-----------------
2015.01.01 2 2.2
2015.01.03 4 4.4
q).Q.ind[t;enlist 2]
_
```

### 14.5.5 .Q.dpft¶

`.Q.dpft`The utility .Q.dpft assists in creating partitioned and segmented tables by incorporating the functionality of .Q.en at a slightly higher level. It is convenient when partitions are loaded and written out iteratively.

`.Q.dpft``.Q.en`The first parameter is the symbolic file handle of the database root directory. The second parameter is the q data value that will become the name of the partition subdirectory. The third parameter is the name of the field to which the `p# attribute is applied (usually `sym for trades and quotes). The last parameter is the table name.

``p#```sym`The .Q.dpft function rearranges (a copy of) the named table so that the partition column is first and the parted column is second, and then splays it into the specified partition in the specified directory. When appropriate, it enumerates all symbol columns over sym and saves a sym list in the root. The result is the table name if the operation is successful.

`.Q.dpft``sym`Notes:

- Because the final parameter is a table name, this function cannot be applied to a local variable.
- The table columns must be simple or compound lists.
- The source table cannot be a keyed table.

We recycle our favorite example, using .Q.dpft to write it as a partitioned table into a fresh /db.

`.Q.dpft``/db````q
q)t:([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101.5 33.5)
q).Q.dpft[`:/db;2015.01.01;`sym;`t]
_
q)t:([] ti:09:31:00 09:32:00; sym:`ibm`msft; p:101 33f)
q).Q.dpft[`:/db;2015.01.02;`sym;`t]
_
q)\l /db
q)select from t where date in 2015.01.01 2015.01.02
_
```

### 14.5.6 .Q.fs¶

`.Q.fs`In many cases, splayed tables (as well as partitioned or segmented slices) are too large to fit into memory. Thus, we face the dilemma of how to create such tables, since splaying requires the table to be in memory. The utility .Q.fs comes to the rescue by allowing us to process text files in "chunks."

`.Q.fs`Note

The chunk size used by .Q.fs is hard-coded. If this does not provide adequate performance may wish to use .Q.fsn, which exposes the chunk size as an additional parameter. The optimal chunk size will vary for each application.

`.Q.fs``.Q.fsn`Loosely speaking, .Q.fs applies a function to the records of a text file in âchunksâ instead of processing the entire file in one gulp. It iterates the function over a number of bite-sized record lists. As a trivial warm-up, we create a text file containing the first 100000 non-negative integers, one per line and we chunk the calculation of their squares. (The call to 0N! enables us to see the result of each chunk as it is processed.)

`.Q.fs``0N!````q
q)`:/data/L 0: string til 100000
_
q).Q.fs[{0N!x*x};`:/data/L]
0 1 4 9 16 25 36 49 64 81 100 121 144 169 196 225 256 289 324 361 400 441 484..
560979225 561026596 561073969 561121344 561168721 561216100 561263481 5613108..
2071888324 2071979361 2072070400 2072161441 2072252484 2072343529 2072434576 ..
4536157201 4536291904 4536426609 4536561316 4536696025 4536830736 4536965449 ..
7953785856 7953964225 7954142596 7954320969 7954499344 7954677721 7954856100 ..
588890
```

While chunking provides no material benefit in this trivial example, it is easily adaptable to larger problems. Assume we have a CSV file containing one dayâs pseudo-trade records (here we manufacture one).

```q
q)ct:00:00:00.000+(10*(til 100000))mod `int$24:00:00.000
q)ix:100000?til 2
q)cs:(`ibm`ms) ix
q)cp:(115 25@ix)+-1.+100000?1.
q)trade:([] ti:ct; sym:cs; px:cp)
q)save `:/data/trade.csv
_
q) read0 `:/data/trade.csv
"ti,sym,px"
"00:00:00.000,ibm,114.767"
"00:00:00.010,ibm,114.7566"
"00:00:00.020,ibm,114.1825"
â¦
```

We intend to save this data into a kdb+ daily partition slice, but suppose that loading it all at once results in a WSFULL error because our q process has insufficient memory. We demonstrate how to read the CSV file and write out to the partition slice in chunks. For convenience, we wrap the read and symbol enumeration into a function ldchunk. Then we use .Q.fsto iterate ldchunk over the chunks, upserting each result to the partition directory. Again we instrument with 0N! to see each chunk result.

`ldchunk``.Q.fs``ldchunk``0N!````q
q)ldchunk:{.Q.en[`:/db] flip `time`sym`price!("TSF";",") 0: x}
q).Q.fs[{0N!.[`:/db/2015.01.01/t/;();,;ldchunk x]}] `:/data/trade.csv
`:/db/2015.01.01/t/
`:/db/2015.01.01/t/
`:/db/2015.01.01/t/
â¦
2638673j
```

We verify that the partition is properly written.

```q
q)\l /db
q)select from t where date=2015.01.01
_
```

When performance matters in the real world, we would proceed to sort the completed partition slice by time within sym and apply the `p# attribute to the sym column.

``p#``sym````q
q)`sym`time xasc `:/db/2015.01.01/t
_
q)@[`:/db/2015.01.01/t; `sym; `p#]
_
q)\l /db
q)meta t
c    | t f a
-----| -----
date | d
time | t
sym  | s p
price| f
```

### 14.5.7 .Q.chk¶

`.Q.chk`The utility .Q.chk is a unary function whose argument is the symbolic file handle of a root directory. It examines each partition sub-directory in the root and writes an empty splayed slice of the appropriate form wherever a table is missing in a partition.

`.Q.chk`We reconstruct our trades and quotes example, this time using .Q.dpft. In a fresh /db,

`.Q.dpft``/db````q
q)t:([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101 33f)
q).Q.dpft[`:/db;2015.01.01;`sym;`t]
_
q)t:([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101.5 33.5)
q).Q.dpft[`:/db;2015.01.02;`sym;`t]
_
q)q:([] ti:09:30:00 09:31:00; sym:`ibm`msft;b:100.75 32.75; a:101.25 33.25f)
q).Q.dpft[`:/db;2015.01.01;`sym;`q]
_
q)q:([] ti:09:30:00 09:31:00; sym:`ibm`msft;b:101.25 33.25; a:101.75 33.75)
q).Q.dpft[`:/db;2015.01.02;`sym;`q]
_
```

Now we introduce two new partitions, each containing a slice for just one table.

```q
q)q:([] ti:09:30:00 09:31:00; sym:`ibm`msft;b:101. 33.; a:101.5 34f)
q).Q.dpft[`:/db;2009.01.03;`sym;`q]
_
q) t:([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:102 34f)
q).Q.dpft[`:/db;2009.01.04;`sym;`t]
_
```

This will make the table q disappear since it is not in the most recent partition; also a query for t fail on 2015.01.03. We use .Q.chk to fix things.

`q``t``.Q.chk````q
q).Q.chk `:/db
()
()
,`:/db/2009.01.04
,`:/db/2009.01.03
q)select from t where date=2015.01.03
ti sym p
--------
q)select from q where date=2015.01.04
ti sym b a
----------
```

Notice that .Q.chk tells you which partitions it is fixing but not which tables.

`.Q.chk`### 14.5.8 .Q.view¶

`.Q.view`The unary .Q.view is handy when you are executing queries against partitioned or segmented tables. Recall that multiple tables can share the partitioning. The argument of .Q.view is a list of partition values that acts as a filter for all queries against any partitioned table in the database. Otherwise put, the practical effect of applying .Q.view is to add its argument as a constraint in the first sub-phrase of the where clause of every query. This can guard against runaway queries that ask for all historical data, for example.

`.Q.view``.Q.view``.Q.view`In the following example, we create a daily partitioned table with three days of data in a fresh /db. By using .Q.view to restrict the view, we reign in the unqualified query.

`/db``.Q.view````q
q)`:/db/2015.01.01/t/ set ([] ti:09:30:00 09:31:00; p:101 102f)
`:/db/2015.01.01/t/
q)`:/db/2015.01.02/t/ set ([] ti:09:30:00 09:31:00; p:100 100f)
`:/db/2015.01.02/t/
q)`:/db/2015.01.03/t/ set ([] ti:09:30:00 09:31:00; p:103 104f)
`:/db/2015.01.03/t/

q)\l /db
q).Q.view 2015.01.02 2015.01.03
q)select from t / normally dangerous!
date       ti       p
-----------------------
2015.01.02 09:30:00 100
2015.01.02 09:31:00 100
2015.01.03 09:30:00 103
2015.01.03 09:31:00 104
```

This is especially useful when you are testing or when you expose large historical databases to users who are prone to forget that they must qualify their queries by date. To reset the default view to all partitions, invoke .Q.view as a nullary.

`.Q.view````q
q).Q.view[]
q)select from t / once again gets everything!
date       ti       p
-----------------------
2015.01.01 09:30:00 101
2015.01.01 09:31:00 102
2015.01.02 09:30:00 100
2015.01.02 09:31:00 100
2015.01.03 09:30:00 103
2015.01.03 09:31:00 104
```

You can use the partition value variable .Q.pv in the argument to .Q.view.

`.Q.pv``.Q.view````q
q).Q.view[]
q).Q.pv
2015.01.01 2015.01.02 2015.01.03
q).Q.view -2#.Q.pv
q)select from t / <- normally dangerous
date       ti       p
-----------------------
2015.01.02 09:30:00 100
2015.01.02 09:31:00 100
2015.01.03 09:30:00 103
2015.01.03 09:31:00 104
```

## 14.6 Kdb Database¶

Many kdb+ newcomers find the notion of a kdb+ database shrouded in mystery. In this section, we answer common qbie questions.

- What is a kdb+ database?
- How is it different from an RDBMS?
- How do I create a kdb+ database?
- How do I startup a kdb+ database?
- What happens at startup?

### 14.6.1 Comparing kdb+ to an RDBMS¶

While q-sql makes certain aspects of a kdb+ database seem familiar to a SQL programmer, the two are quite different under the covers. Folks coming into the world of q and kdb+ from a traditional RDBMS environment may feel as if they have entered the Matrix after taking the red pill from Morpheus. The fundamental differences are:

- Kdb+ is based on lists, which are ordered collections allowing duplicates, whereas SQL is based on sets, which are unordered collections of distinct elements.
- Kdb+ stores data as contiguous items in column lists, whereas an RDBMS stores data as fields within non-contiguous rows. Neo in kdb+ says, âThere are no rows.â
- Kdb+ table operations are vector operations on columns, whereas SQL operates on individual fields and rows.

Kdb+ is based on lists, which are ordered collections allowing duplicates, whereas SQL is based on sets, which are unordered collections of distinct elements.

Kdb+ stores data as contiguous items in column lists, whereas an RDBMS stores data as fields within non-contiguous rows. Neo in kdb+ says, âThere are no rows.â

Kdb+ table operations are vector operations on columns, whereas SQL operates on individual fields and rows.

We summarize the major differences between an RDBMS and kdb+ in the following chart.

`INSERT``UPDATE``upsert``COMMIT``ROLLBACK`### 14.6.2 The Physical Layout of a kdb+ Database¶

Here is a Zen koan: What is a kdb+ database? At the most basic level, a kdb+ database is a file system directory (and sub directories) holding q entities. This directory is the root of the database. All constituents of the database are simply q entities saved in files. Database entities either reside at some level under the root or are pointed to from par.txt under the root.

`par.txt`We describe the various kdb+ database components in this section and show examples in the following section. In our descriptions we use /db as our root, but this is arbitrary.

`/db`### 14.6.2.1 The sym File¶

The sym file is an optional serialized q data file containing a list of unique symbols used as the domain for symbol enumeration. Placing the sym file in the root guarantees that it will be loaded into memory at q startup.

```q
/db
    sym â¦
```

There is no strict requirement that there be only one enumeration domain, but conventionally all symbol columns from all tables are enumerated over a single domain sym. The .Q utilities that handle symbol enumeration assume this. If you choose to have multiple enumeration domains, be aware that symbols in the resulting enumerated values will act as expected under = but not with ~ since they have different types. You can resolve the enumerations with value if this is a problem.

`.Q``=``~``value`Important

Pay very careful attention to the sym file, as it is a single point of failure. Corrupting or losing it will result in all your symbol columns being irresolvable. Good practice is to use conditional enumeration â e.g., `:/db/sym? â when you enumerate symbols manually or use .Q utilities that do this for you. Always back up your database before operating on the sym file.

``:/db/sym?``.Q`The sym file will not normally be a choke point when loading historical data from an external source into a kdb+ because conditional enumeration (and the .Q utilities) use file locking to mediate concurrent updates. Alternately, if the symbol domain is known in advance, you can load the sym list into memory and use non-conditional enumeration â i.e., `sym$. For example, one approach is to create a preprocessing utility in Perl to extract all symbols from the source data, import this as a list in q and place the distinct items in sym. Then create the historical partitions running concurrent processes that use unconditional enumerations. Be mindful that unconditional enumeration fails for a symbol not in the domain.

`.Q```sym$`### 14.6.2.2 Other Serialized Files in Root¶

While it is seldom done in practice (perhaps because few kdb+ programmers are aware of the possibility), it is possible to place any serialized q entity into the root and have it loaded into an eponymous file. This can be a more efficient than using a script to initialize variables in the workspace.

If you use this, donât go overboard. Also, it might break .Q.en.

`.Q.en````q
/db
    [sym]
    var1
    var2
    â¦
```

One type of serialized file that can be initialized in this fashion is a reference table (or keyed table) of modest size (up to millions of rows). Such a table will be loaded into memory in its entirety, so lookups or joins on it will be quite fast.

It is possible to (re)load the entire state of the root context automatically by serializing that directory and loading it in this fashion.

### 14.6.2.3 Scripts¶

A script in the root can hold arbitrary q code that will be loaded upon startup. In particular, functions defined in such a script can be viewed as stored procedures for the database. While it is possible to have multiple q scripts in the root, you probably want precise control over the order in which the scripts are executed. Good practice is to keep in the root one startup script that loads scripts residing in libraries elsewhere.

```q
/db
    â¦
    init.q
```

### 14.6.2.4 Splayed Tables¶

Reference tables of intermediate size (up to tens of millions of rows) can be splayed under the root. Splayed table directories must be immediately under the root.

```q
/db
    [sym]
    â¦
    /splay1
    /splay2
    â¦
```

Because splayed tables are mapped and columns are page-faulted into memory as needed, no more than a few columns will be simultaneously resident in the workspace. Assuming that the server has adequate memory available to kdb+, q should expect to find recently accessed columns cached by the OS. In this scenario, performance of lookups against splayed tables can be close to that of in-memory tables.

### 14.6.2.5 Partitioned Tables¶

The partition directories of non-segmented tables must be immediately under the root. Every partition directory contains a subdirectory for each table sharing the partition; in each subdirectory is the splayed slice for that table's partition value. For example, for daily partitioned trades and quotes, a dayâs directory would contain a splayed directory for that dayâs trades and another splayed directory for that dayâs quotes.

```q
/db
    [sym]
    â¦
    /part1
        /trade
        /quote
    /part2
        /trade
        /quote â¦
```

There need not be a directory for every partition value but the partition directories must have a uniform structure. That is, a partition value having any entry must contain splayed directories for all tables in the partition. Splay an empty schema for a table having no records for a partition value.

### 14.6.2.6 Segmented Tables¶

In contrast to normal partitioned tables (i.e., non-segmented tables) in which all records live under the root, no records of a segmented table can reside under the root. Instead, the root contains a text file par.txt having one entry per line. Each entry represents an OS path for a segment directory containing the data in that segment; symlinks are permissible.

`par.txt`Each segment directory contains partition directories whose structure is completely analogous to that of a regular partitioned table. There is one directory per partition value and beneath that, one sub-directory per table. Note that depending on the nature of the segmentation, multiple segment directories may contain directories for the same partition value. The previous restriction about consistency across partition directories applies to segments.

```q
/db
    [sym]
    â¦
    par.txt
    â¦
== drive 1 =============
/seg1
    /part1
    /part2
    â¦
== drive 2 =============
/seg2
    /parta
    /partb
    â¦
== â¦ =============
```

### 14.6.3 Creating and Populating a kdb+ Database¶

Here we demonstrate how to create and populate various kdb+ database configurations. Although the examples are simplistic, they cover most common situations.

There are no wizards for laying out and populating a kdb+ database. Instead, you must write each component in the correct form to the correct location. There are utilities to help with the process but it is your responsibility to ensure that everything is valid. Improperly structured directories or incorrectly written data files will generate run-time errors the first time q code attempts to access them.

Once you have created a kdb+ database, start a q process pointing at the root directory of the database to attach it to the session. Alternately, in a running q instance you can issue \l on the root directory.

`\l`### 14.6.3.1 What Happens at Startup?¶

This question has a deceptively simple answer. When you point q at a directory or file, it applies a single rule:

Whatever it recognizes as its own, it does.

Thus, a serialized q entity is loaded; a splayed, partitioned or segmented table is mapped; a script is executed.

Important

Almost anything that q does not recognize causes it to abort that portion of startup. In particular, if q discovers foreign or undecipherable files in splayed, partitioned or segmented directories, it will not map the tables contained there. Even unexpected hidden files (e.g., those written by the Mac OS Finder) will abort the map.

When you point q startup at a directory, that directory becomes the root directory for the kdb+ database and also the current working directory for the OS. We shall refer to this scenario as kdb+ startup to distinguish it from an arbitrary q session. We shall cover the items that Kdb+ startup finds in the order that it handles them.

- Serialized q entities
- Splayed tables
- Partitioned or segmented tables
- Scripts

Serialized q entities

Splayed tables

Partitioned or segmented tables

Scripts

File handles that are not fully qualified â i.e., relative paths â are interpreted relative to the q home directory.

### 14.6.3.2 Serialized q Entities¶

When kdb+ startup recognizes a serialized q data file, it loads that data into an eponymous variable. The canonical example is the sym file in the root, containing the serialized list of (unique) symbols in the enumeration domain for (all) symbol columns. Multiple serialized entities are loaded in the order in which the OS file system lists them.

In our first example, we manually write out a sym file, exit q and then start kdb+ pointing at the root.

```q
q)`:/db/sym set `a`b`c`z`x`y
`:/db/sym
q)\\
```

```q
$q /db
```

```q
q)sym
`a`b`c`z`x`y
```

Starting with a fresh /db, we place a defined âconstant,â a utility function and a serialized keyed table in the root.

`/db````q
q) `:/db/LIFE set 42
`:/db/LIFE
q)`:/db/f set {x*y}
`:/db/f
q)`:/db/lookup set ([s:`a`b`c] v:1 2 3)
`:/db/lookup
q)\\
```

```q
$q /db
```

```q
q)LIFE
_
q)f
_
q)lookup
_
```

As of this writing (Sep 2015), q will not look for serialized data in directories other than the root. Nor will it automatically load serialized data whose file names have extensions. However, you can load such data files manually using \l or get.

`\l``get`### 14.6.3.3 Splayed Tables¶

When kdb+ startup finds a subdirectory immediately beneath the root that it recognizes as a splayed table, it maps the table into memory. All symbol columns must be enumerated for a table to be splayed. Ensure that the sym file is in the root directory if you do not use a built-in utility.

Starting with a fresh /db, we splay two tables under the root. The first does not have symbol columns; the second does.

`/db````q
q)`:/db/tref/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
_
q)`:/db/cust/ set .Q.en[`:/db;] ([] sym:`ibm`msft`goog; name:`oracle`microsoft`google)
_
q)\\
```

```q
$q /db
```

```q
q)select from tref
c1 c2
_
q)select from cust
_
```

### 14.6.3.4 Partitioned Tables¶

If kdb+ startup finds subdirectories immediately beneath the root whose names constitute valid partition values, it examines them for splayed tables comprising partition slices. Valid partitioned tables are mapped. The presence of partitioned tables is independent of the presence of (plain) splayed tables. Partitioned tables and segmented tables are mutually exclusive.

All symbol columns in every partition slice must be enumerated, customarily over a file sym in the root. Ensure that the sym file is in the root directory if you do not use the built-in utilities.

We recreate the previous example in which multiple tables share a daily partition in a fresh /db.

`/db````q
q)`:/db/2015.01.01/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101 33f)
_
q)`:/db/2015.01.02/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101.5 33.5)
_
q)`:/db/2015.01.01/q/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; b:100.75 32.75; a:101.25 33.25f)
_
q)`:/db/2015.01.02/q/ set .Q.en[`:/db;] ([] ti:09:30:00 09:30:00; sym:`ibm`msft; b:101.25 33.25; a:101.75 33.75)
_
q)\\
```

```q
$ q /db
```

```q
q)\l /db
q
q)select from t where date within 2015.01.01 2015.01.02
-
q)select from q where date within 2015.01.01 2015.01.02
_
```

### 14.6.3.5 Segmented Tables¶

When kdb+ startup finds a par.txt file in the root, it interprets each line as a segment location and it examines the locations for valid segments of partitioned tables. Valid tables are mapped.

`par.txt`Ensure that the segment directories are not located under the root; otherwise you will relive the table scene in the first Alien movie.

The presence of segmented tables is independent of the presence of plain splayed tables. Segmented tables and partitioned tables are mutually exclusive. All symbol columns in every segment must be enumerated. Ensure that the (common) sym file is in the root directory.

We recreate the previous example from Â§14.4 in which multiple tables share segmentation by symbol range. In fresh directories /am, /nz and /db,

`/am``/nz``/db````q
q)extr:{[t;r] select from t where (`$1#'string sym) within r}
q)t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`t; p:101 17f)
q)q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`ibm`t`ibm; b:100.75 16.9 100.8;a:101.25 17.1 101.1)

q)`:/am/2015.01.01/t/ set extr[t;`a`m]
_
q)`:/nz/2015.01.01/t/ set extr[t;`n`z]
_
q)`:/am/2015.01.01/q/ set extr[q;`a`m]
_
q)`:/nz/2015.01.01/q/ set extr[q;`n`z]
_
q)t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`t`ibm; p:17.1 100.9)
q)q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`t`ibm`t;b:17 100.7 17.1;a:17.2 101.25 17.25)

q)`:/am/2015.01.02/t/ set extr[t;`a`m]
_
q)`:/nz/2015.01.02/t/ set extr[t;`n`z]
_
q)`:/am/2015.01.02/q/ set extr[q;`a`m]
_
q)`:/nz/2015.01.02/q/ set extr[q;`n`z]
_
q)`:/db/par.txt 0: ("/am"; "/nz")
`:/db/par.txt
q)\\
```

```q
$q
```

```q
q)\l /db
q)select from t where date within 2015.01.01 2015.01.02
_
q)select from q where date within 2015.01.01 2015.01.02
_
```

### 14.6.3.6 Scripts¶

Files with extension .q are interpreted as q scripts; those with extension .k as k scripts. Scripts found in the root are loaded and executed in alphabetical order. You will probably want better control over the order of execution, so it is best to have in the root a single script that loads other scripts in the desired sequence.

`.q``.k`Note

An invalid expression in a script causes the entire script load to abort. Locating the invalid entry can be non-trivial since by default console display is suppressed during load. Judicious use of 0N! can be helpful. Sometimes using binary search using block comments is an effective approach to locating the offending line. Bottom line: keep scripts short!

`0N!`In our first example, we initialize the workspace using a startup script /db/init.q that we create from q (because we can). You could do it in any text editor.

`/db/init.q````q
q)`:/db/init.q 0: ("LIFE:42";"f:{x*y}";"lookup:([s:`a`b`c] v:1 2 3)")
`:/db/init.q
q)\\
$q /db
q)LIFE
_
q)f
_
q)lookup
_
```

Since scripts are loaded as the last step in kdb+ startup, any loaded script can rely on startup, serialized variables and mapped tables bring present. For example, in a fresh /db we create a splayed table and a startup script from within q and then run the script at startup.

`/db`q)`:/db/cust/ set .Q.en[`:/db;] ([] sym:`ibm`msft`goog;name:`oracle`microsoft`google)
_
q)`:/db/init.q 0: ("sym"; "show select from cust")
_
q)\\
$q /db
`ibm`msft`goog`oracle`microsoft`google
sym  name
--------------
ibm  oracle
msft microsoft
goog google
q)

```q
q)`:/db/cust/ set .Q.en[`:/db;] ([] sym:`ibm`msft`goog;name:`oracle`microsoft`google)
_
q)`:/db/init.q 0: ("sym"; "show select from cust")
_
q)\\
```

```q
$q /db
```

```q
`ibm`msft`goog`oracle`microsoft`google
sym  name
--------------
ibm  oracle
msft microsoft
goog google
q)
```

Now we demonstrate a script that loads other scripts. Start with a fresh /db. We write out a script having a library function (you could do it in a text editor) and then load that script from init.q at startup.

`/db``init.q````q
q)`:/scripts/expr.q 0: enlist ".jab.lib.expr.subst:{x@y}"
_
q)`:/db/init.q 0: enlist "\\l /scripts/expr.q"
_
q)\\
```

```q
$q /db
```

```q
q).jab.lib.expr.subst
{x@y}
```

## 14.7 Putting It All Together¶

In this section, we combine the various constituents of a kdb+ database demonstrated in the previous sections to give two archetypal examples. The first has partitioned tables; the second has segmented tables.

### 14.7.1 Partitioned Database¶

To construct our partitioned database, we combine the steps in previous sections to obtain the following meta-script. (We assume that the scripts in /lib already exist.)

`/lib````q
/ create serialized variables
`:/db/LIFE set 42
`:/db/f set {x*y}
`:/db/lookup set ([s:`a`b`c] v:1 2 3)

/ create splayed tables
`:/db/tref/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
`:/db/cust/ set .Q.en[`:/db;] ([] sym:`ibm`msft`goog; name:`:/db/sym?`oracle`microsoft`google)

/create partitioned tables
`:/db/2015.01.01/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101 33f)
`:/db/2015.01.02/t/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; p:101.5 33.5)
`:/db/2015.01.01/q/ set .Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`msft; b:100.75 32.75; a:101.25 33.25)
`:/db/2015.01.02/q/ set .Q.en[`:/db;] ([] ti:09:30:00 09:30:00; sym:`ibm`msft; b:101.25 33.25; a:101.75 33.75)

/ create load script
`:/db/init.q 0: ("TheUniverse:42";"\\l /lib/math.q";
 "\\l /lib/expr.q")
```

Now execute this script to create the database. The resulting partitioned database has the following layout.

```q
/db
    sym
    LIFE
    F
    Lookup
    /tref
        .d
        c1
        c2
    /cust
        .d
        sym
        name
    /2009.01.01
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
    /2009.01.02
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
    init.q
```

### 14.7.2 Segmented Database¶

To construct a segmented database, we combine previous steps to obtain the following meta-script. (We again assume that the scripts in /lib already exist.)

`/lib````q
/ create serialized variables
`:/db/LIFE set 42
`:/db/f set {x*y}
`:/db/lookup set ([s:`a`b`c] v:1 2 3)

/ create splayed tables
`:/db/tref/ set ([] c1:1 2 3; c2:1.1 2.2 3.3)
`:/db/cust/ set .Q.en[`:/db;] ([] sym:`ibm`msft`goog; name:`oracle`microsoft`google)

/ create segmented tables
extr:{[t;r] select from t where (`$1#'string sym) within r}
t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`ibm`t; p:101 17f)
q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`ibm`t`ibm; b:100.75 16.9 100.8; a:101.25 17.1 101.1)
`:/am/2015.01.01/t/ set extr[t;`a`m]
`:/nz/2015.01.01/t/ set extr[t;`n`z]
`:/am/2015.01.01/q/ set extr[q;`a`m]
`:/nz/20015.01.01/q/ set extr[q;`n`z]
t:.Q.en[`:/db;] ([] ti:09:30:00 09:31:00; sym:`t`ibm; p:17.1 100.9)
q:.Q.en[`:/db;] ([] ti:09:29:59 09:29:59 09:30:00; sym:`t`ibm`t; b:17 100.7 17.1;a:17.2 101.25 17.25)
`:/am/2015.01.02/t/ set extr[t;`a`m]
`:/nz/2015.01.02/t/ set extr[t;`n`z]
`:/am/2015.01.02/q/ set extr[q;`a`m]
`:/nz/2015.01.02/q/ set extr[q;`n`z]

`:/db/par.txt 0: ("/am"; "/nz")

/ create load script
`:/db/init.q 0: ("TheUniverse:6*7"; "\\l /lib/math.q"; "\\l /lib/expr.q")
```

Now execute this script to create the database. The resulting partitioned database has the following layout.

```q
/db
    sym
    LIFE
    F
    Lookup
    /tref
        .d
        c1
        c2
    /cust
        .d
        sym
        name
    par.txt
    init.q
== drive 1 ==
/am
    /2009.01.01
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
    /2009.01.02
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
== drive 2 ==
/nz
    /2009.01.01
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
    /2009.01.02
        /t
            .d
            ti
            sym
            p
        /q
            .d
            ti
            sym
            b
            a
```

## 14.8 Thereâs No Place Like QHOME¶

`QHOME`We discuss how kdb+ interacts with the OS environment when it looks for files.

### 14.8.1 The Environment Variables¶

Three environment variables QHOME, QLIC and QINIT are used by kdb+ at startup.

`QHOME``QLIC``QINIT`The environment variable QHOME specifies the directory where kdb+ expects to find the bootstrap file q.k. By default, it also looks there for the license file k4.lic. If QHOME is not defined, kdb+ falls back to $HOME/q for Unix-based systems and c:\q for Windows.

`QHOME``q.k``k4.lic``QHOME``$HOME/q``c:\q`The environment variable QLIC overrides the default location for the license file. If QLIC is not defined, kdb+ falls back to QHOME (or its fallback).

`QLIC``QLIC``QHOME`The environment variable QINIT specifies the name of the file that is executed immediately after the load of q.k. If QINIT is not defined, kdb+ attempts to load the file q.q from QHOME. If QHOME is not defined or q.q is not found, no error is reported.

`QINIT``q.k``QINIT``q.q``QHOME``QHOME``q.q`The file in QINIT is executed in the root context (i.e., \d).

`QINIT``\d`### 14.8.2 q in da hood¶

Upon startup, the current directory is the directory from which the q session was started.

After startup, an easy way to see where in the file system the q executable resides is to use a function such as the nullary whereami below. Recall that the command \cd acts like the OS cd command by returning the current working directory.

`whereami``\cd``cd````q
whereami:{-1 "cd ~ ",system "cd";}
```

Loading a script that resides elsewhere does not in itself change the current directory; of course, if that script executes a \cd then the current directory may change. In particular, the current directory is not temporarily changed to the location of the target script.

`\cd`Loading a database, either as part of the startup invocation or via a \l command, changes the current directory to the database root. All subsequent relative access is based from there. This is important when starting q with -u as it determines the file hierarchy visibility.

`\l``-u`When asked to load a file whose name is not fully qualified, q first searches the current directory. If the file is not found, it then searches QHOME if it is defined and $HOME otherwise.

`QHOME``$HOME`