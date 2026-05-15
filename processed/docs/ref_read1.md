# read1: Read Bytes from File or Named Pipe

## Syntax

```q
read1 f           read1[f]
read1 (f;o)       read1[(f;o)]
read1 (f;o;n)     read1[(f;o;n)]
read1 h           read1[h]
read1 (fifo;n)    read1[(fifo;n)]
```

## Parameters

- `f`: a file symbol
- `o`: offset as a non-negative integer/long
- `h`: a system or process handle
- `fifo`: a communication handle to a Fifo
- `n`: length as a non-negative integer/long

## File Operations

Returns bytes from the specified file:

- **File symbol alone**: Entire file content
- **File symbol with offset `(f;o)`**: Content from offset onwards
- **File symbol, offset, and length `(f;o;n)`**: Up to `n` bytes starting at offset `o`

### Examples

```q
q):test.txt 0:("hello";"goodbye")      / write some text to a file
q)read1`:test.txt                       / read in as bytes
0x68656c6c6f0a676f6f646279650a
q)"c"$read1`:test.txt                   / convert from bytes to char
"hello\ngoodbye\n"

q)/ Read 500000 lines, chunks of (up to) 100000 at a time
q)d:raze{read1(`:/tmp/data;x;100000)}each 100000*til 5 
```

### Compression

Compressed files are automatically decompressed:

```q
q)(`:file;17;2;9)1:100#0x0
`:file
q)read1`:file
0x0000000000000000000000000000000000000000000000000..
```

## Named Pipe Operations

Available since V3.4.

- **List `(fifo;length)`**: Returns specified number of bytes from the named pipe
- **Integer atom `fifo`**: Blocks until EOF and returns accumulated bytes

### Examples

```q
q)h:hopen`$":fifo:///etc/redhat-release"
q)"c"$read1(h;8)
"Red Hat "
q)"c"$read1(h;8)
"Enterpri"
q)system"mkfifo somefifo";h:hopen`fifo:somefifo; 0N!read1 h; hclose h
```

---

**Related Topics**: File system, Interprocess communication
