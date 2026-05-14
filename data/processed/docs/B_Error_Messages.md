# Appendix B. Error Messages

## B.1 Runtime Errors

| Error | Example | Description |
|-------|---------|-------------|
| Access | | Attempt to read files above directory, run system commands or failed usr/pwd |
| Assign | `cos:12` | Attempt to reuse a reserved word |
| Conn | | Too many incoming connections (1022 max) |
| Domain | `til -1` | Argument out of domain |
| Glim | | As of q3.2 there is no limit on number of `` `g# `` attributes |
| Length | `(til 2)+til 3` | Incompatible list lengths |
| limit | | Attempt to create list longer than allowable maximum (2 billion in q2.*) or trying to serialize an object > 2GB in any version |
| loop | `a::b::a` | Circular reference loop |
| mismatch | | Columns cannot be aligned for operation |
| mlim | | More than 999 nested columns in splayed table |
| nyi | | Not yet implemented |
| os | | Operating system error |
| pl | | peach can't handle parallel lambda's (2.3 only) |
| Q7 | | Unimplemented op on file nested array |
| rank | `+[2;3;4]` | Invalid rank or valence |
| s-fail | `` `s#3 2 `` | Invalid attribute setting |
| splay | | Unimplimented op on splayed table |
| stack | | Exhausted stack space |
| stop | | User interrupt (ctrl-c) or time limit (-T) |
| stype | `'42` | Invalid type used to signal |
| type | `til 4.2` | Wrong type |
| value | | Missing value |
| vd1 | | Attempted multithread update |
| wsfull | | malloc failed. ran out of swap (or addressability on 32bit). or hit `-w` limit |
| xxx | `'xxx` | *xxx* undefined |

## B.2 Parse Errors

| Error | Example | Description |
|-------|---------|-------------|
| ( ) [ ] { } " | | Unpaired item |
| branch | | A branch (if; do; while; `$[.;.;.]`) more than 255 byte codes away |
| char | | Invalid character |
| constants | | Too many constants (96 max) |
| globals | | Too many global variables (255 max) |
| locals | | Too many local variables (24 max) |
| params | | Too many parameters (8 max) |

## B.3 System Errors

| Error | Examples | Description |
|-------|----------|-------------|
| *xxx:yyy* | *xxx* is a kdb+ message; *yyy* is the OS message. *xxx* can be: addr, close, conn, p (from -p), snd, rcv, *filename* (invalid) |

## B.4 License Errors

| Error | Example | Description |
|-------|---------|-------------|
| cores | | Exceeded number of licensed cores |
| exp | | Expiry date passed |
| host | | Unlicensed host |
| k4.lic | | k4.lic file not found, check QHOME/QLIC |
| os | | Unlicensed OS |
| srv | | Attempt to use client-only license in server mode |
| upd | | Attempt to use version of kdb+ more recent than update date |
| user | | Unlicensed user |
| wha | | Invalid system date |
