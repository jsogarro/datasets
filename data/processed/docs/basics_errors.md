# Errors in kdb+ and q

## Runtime Errors

Runtime errors occur during code execution. Common examples include:

**File and Access Issues:**
- `{directory}/q.k. OS reports: No such file or directory` — The `q.k` file wasn't found in the `QHOME` directory
- `access` — Attempted to read files above directory or run unauthorized system commands

**Type and Domain Errors:**
- `domain` — Operation outside valid range (e.g., `til -1`)
- `type` — Wrong datatype provided to a function
- `cast` — Value not found in enumeration

**Data Structure Errors:**
- `length` — Arguments don't conform to required dimensions
- `rank` — Invalid rank for operation
- `mismatch` — Columns can't align during table operations

**IPC and Connection Errors:**
- `conn` — Too many connections (limit previously 1022, OS-dependent after 4.1t)
- `hop` — `hopen` handle request failed
- `timeout` — Connection timeout occurred

**Performance Limits:**
- `wsfull` — Memory allocation failed or swap exhausted
- `-w abort` — Hit workspace memory limit
- `stack` — Ran out of stack space (consider using accumulators instead of recursion)

**Threading Restrictions:**
- `nosocket` — Sockets only usable in main thread
- `sys` — System calls blocked outside main thread
- `noupdate` — Global updates blocked by `-b` flag or in threads

## System Errors

System errors originate from file operations and IPC, formatted as `XXX:YYY` where `XXX` is kdb+ context and `YYY` is the OS error message.

## Parse Errors

Parse errors occur during expression evaluation or file loading:

- `[({])}"`  — Unclosed bracket, brace, parenthesis, or quote
- `branch` — Branch statement exceeds 65025 bytes
- `char` — Invalid character encountered (watch for non-breaking spaces)
- `globals` — Too many global variables defined
- `locals` — Too many local variables (8 max parameters)
- `limit` — Too many constants or general limit exceeded

## License Errors

License validation failures on launch include:

- `exp` — License expiration date is prior to system date
- `host` — Hostname doesn't match license specification
- `k4.lic` — License file not found in expected locations
- `cores`/`cpu` — License insufficient for available hardware
- `wha` — System date predates kdb+ version date
- `wrong q.k version` — q binary and q.k file versions mismatch

## Error Handling

Three primary mechanisms for managing errors:

1. **System Command** — Use `\` (abort) to clear one execution stack level
2. **Exit Functions** — Call `exit` to terminate the process or set `.z.exit` callback
3. **Error Control** — Deploy Signal to raise errors or Trap/Trap At operators to catch them

Reference the [Debugging](../debug/) documentation for more detailed troubleshooting approaches.
