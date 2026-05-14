# File System in kdb+ and q

## Overview

kdb+ interacts with the filesystem through two main mechanisms: **one-shot operations** and **persistent connection handles**. The documentation notes that "Handles are more efficient for multiple operations on a file," and filepaths display with forward slashes across all operating systems.

## One-Shot Operations

These operations handle individual file interactions without maintaining open connections:

**Data File Operations:**
- `get`/`set` — read/write or memory-map data files
- `value` — read a data file
- `hcount` — determine file size
- `hdel` — delete files or folders
- `hsym` — convert symbols to file symbols

**Text and Binary I/O:**
- `0:` operator — read/write text characters
- `read0` — read characters
- `1:` operator — read/write bytes
- `read1` — read bytes
- `2:` operator — load shared objects

**Table Operations:**
- `save`/`load` — persist and retrieve variables
- `rsave`/`rload` — handle splayed tables
- `dsave` — save multiple tables

## Setting and Getting

The `set` and `get` keywords treat files as persistent variables:

```q
q)`:data/foo`:data/bar set'(42;"thin white duke")
`:data/foo`:data/bar
q)get `:data/foo
42
q)get `:data/bar
"thin white duke"
```

## File Utilities

Key functions for file management include `hcount` (file size), `hdel` (deletion), and `hsym` (symbol conversion).

## Text and Binary Operations

The distinction between `0` (text) and `1` (binary) operators provides flexibility for different file types. Text operations handle character data, while binary operations manage byte-level access.

## Persistent Connections

Opening a file connection returns an integer **handle**. System handles 0, 1, and 2 represent console, stdout, and stderr respectively.

**Connection Operations:**
- `hopen` — open file connections
- `hclose` — close connections
- Applying a handle appends bytes; applying its negative appends text

### Text Example

```q
q)show h:hopen `:foo/bar.txt
12i
q)neg[h] "hear the lark and hearken"
-12i
q)-12i "to the barking of the dog fox"
-12i
q)hclose h
q)read0 `:foo/bar.txt
"hear the lark and hearken"
"to the barking of the dog fox"
```

### Bytes Example

```q
q)hopen ":foo/hello.dat"
7i
q)7i 0x68656c6c6f776f726c64
7i
q)hclose 7i
q)read1 `:foo/hello.dat
0x68656c6c6f776f726c64
```

## Relative Filepaths

The system searches for relative filepaths in this sequence: current directory, `QHOME`, then `QLIC`.
