# get, set – Reference

_Read or set the value of a variable or a kdb+ data file_

## get

_Read or memory-map a variable or kdb+ data file_

```q
get x     get[x]
```

Where `x` is:
- the name of a global variable as a symbol atom
- a file or folder named as a symbol atom or vector

Returns its value. Signals a `type` error if the file is not a kdb+ data file.

Used to map columns of databases in and out of memory when querying splayed databases, and can read q log files, etc.

```q
q)a:42
q)get `a
42

q)\l trade.q
q)`:NewTrade set trade                  / save trade data to file
`:NewTrade
q)t:get`:NewTrade                       / t is a copy of the table
q)`:SNewTrade/ set .Q.en[`:.;trade]     / save splayed table
`:SNewTrade/
q)s:get`:SNewTrade/                     / s has columns mapped on demand
```

`value` is a synonym for `get`. By convention, `value` is used for other purposes, but the two are completely interchangeable.

```q
q)value "2+3"
5
q)get "2+3"
5
q)a:1 2 3
q)get `a
1 2 3
q)get `q`w`e!(1 2;3 4;5 6)
1 2
3 4
5 6
q)get (+;1;2)
3
```

Related: `eval`, `value`

## set

_Assign a value to a global variable or persist an object as a file or directory_

```q
nam set y                 set[nam;y]                /set global var nam
file set y                set[file;y]               /serialize y to file
dir set t                 set[dir;t]                /splay t to dir
(file;lbs;alg;lvl) set y  set[(file;lbs;alg;lvl);y] /write y to file, compressed/encrypted
(dir;lbs;alg;lvl) set t   set[(dir;lbs;alg;lvl);t]  /splay t to dir, compressed/encrypted
(dir;dic) set t           set[(dir;dic);t]          /splay t to dir, compressed/encrypted
```

**Parameters:**
- `alg` – integer atom (compression/encryption algorithm)
- `dic` – dictionary (compression/encryption specifications)
- `dir` – filesymbol (directory in the filesystem)
- `file` – filesymbol (file in the filesystem)
- `lbs` – integer atom (logical block size)
- `lvl` – integer atom (compression level)
- `nam` – symbol atom (valid q name)
- `t` – table
- `y` – any q object

**Examples:**

```q
q)`a set 42                         / set global variable
`a
q)a
42

q)`:a set 42                        / serialize object to file
`:a

q)t:([]tim:100?23:59;qty:100?1000)  / splay table
q)`:tbl/ set t
`:tbl/

q)(`:ztbl;17;2;6) set t             / serialize compressed
`:ztbl

q)(`:ztbl/;17;2;6) set t            / splay table compressed
`:ztbl/

q)(`:ztbl/;17;16;6) set t           / splay table encrypted (since v4.0 2019.12.12)
`:ztbl/
```

Anymap write detects consecutive deduplicated (address matching) top-level objects, skipping them to save space (since v4.1t 2021.06.04, v4.0 2023.01.20).

```q
q)a:("hi";"there";"world")
q)`:a0 set a
`:a0
q)`:a1 set a@where 1000 2000 3000
`:a1
q)(hcount`$":a0#")=hcount`$":a1#"
0b
```

**Notes:**
- Since 4.1t 2023.09.29, 4.0 2023.11.03: empty vectors without attributes are deduplicated automatically when writing anymap
- Since 4.1t 2021.06.04, 4.0 2023.01.20: improved memory efficiency of writing nested data sourced from type 77 (anymap) files

```q
q)`:a set 500000 100#"abc"
q)system"ts `:b set get`:a" / was 76584400 bytes, now 8390208
```

### Splayed table

To splay a table `t` to directory `dir`:
- `dir` must be a filesymbol ending with `/`
- `t` must have no primary keys
- columns of `t` must be vectors or compound lists
- symbol columns in `t` must be fully enumerated

### Format

`set` saves data in binary tag+value format, retaining data structure and value.

```q
q)`:data/foo set 10 20 30
`:data/foo
q)read0 `:data/foo
"\376 \007\000\000\000\000\000\003\000\000\000\000\000\000\000"
"\000\000\000\000\000\000\000\024\000\000\000\000\000\000\000\036\000..
```

**Warning:** Setting variables in KX namespaces (`.h`, `.j`, `.Q`, `.q`, `.z`, and single-character namespaces) can result in undesired and confusing behavior.

### Compression/Encryption

For compressed and/or encrypted operations:

```q
(fil;lbs;alg;lvl) set y   / write y to fil, compressed and/or encrypted
(dir;lbs;alg;lvl) set t   / splay t to dir, compressed and/or encrypted
```

Arguments `lbs`, `alg`, and `lvl` are compression and/or encryption parameters.

**Example:** Splay table `t` to directory `ztbl/` with gzip compression:

```q
q)(`:ztbl/;17;2;6) set t
`:ztbl/
```

For dictionary-based specification:

```q
(dir;dic) set t            / splay t to dir, compressed
```

Dictionary keys are either column names of `t` or the null symbol `` ` ``. Values are integer vectors: `lbs`, `alg`, and `lvl`.

**Example:**

```q
q)m1:1000000
q)t:([]a:m1?10;b:m1?10;c:m1?10;d:m1?10)

q)/ Specify compression for cols a, b and defaults for others
q)show dic:``a`b!(17 5 3;17 2 6;17 2 6)
 | 17 5 3
a| 17 2 6
b| 17 2 6
q)(`:ztbl/;dic) set t               / splay table compressed
`:ztbl/
```

**Performance:** Compression may speed up or slow down `set` execution. Impact depends mainly on data characteristics and storage speed.

---

**Related:** File system, File compression, Data at rest encryption (DARE)
