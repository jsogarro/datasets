# eval, reval

Evaluate parse trees

## eval

Evaluate a parse tree

```q
eval x     eval[x]
```

Where `x` is a parse tree, returns the result of evaluating it.

The `eval` function serves as the complement to `parse` and evaluates the parse trees it produces. You can also construct and evaluate parse trees explicitly.

### Examples

```q
q)parse "2+3"
+
2
3
q)eval parse "2+3"
5
q)eval (+;2;3)      / constructed explicitly
5
```

## reval

Restricted evaluation of a parse tree

```q
reval x     reval[x]
```

The `reval` function operates similarly to `eval`, behaving as if the command-line option `-b` were active during evaluation.

### Usage

A practical application appears inside the message handler `.z.pg`, useful for access control to prevent sync messages from updating:

```q
q).z.pg:{reval(value;enlist x)} / define in process listening on port 5000
q)h:hopen 5000 / from another process on same host
q)h"a:4"
'noupdate: `. `a
```

### Restrictions

Behaves as if command-line options `-u 1` and `-b` were active; also blocks system calls that change state. This means:

- All file system writes are blocked
- Read access limited to files in working directory and below
- Prevention of global amendments
- `exit` keyword is blocked (since V4.1t 2021-07-12)
- Blocks `hopen` of a file (since 4.1t 2021.10.13, 4.0 2023.08.11)

### Example

```q
q)h:hopen 4000 / to a server started with -u 1 -p 4000
q)h"reval(hopen;enlist`:somefile)"
'access: somefile
```
