# Debugging Facilities in the q Interpreter

## Overview

The q interpreter provides comprehensive debugging capabilities accessed through a debug prompt indicated by `q))`. When an error occurs during lambda execution, the system suspends and enters the debugger, allowing inspection of local values and navigation through the call stack.

## Error Display

Uncaught errors display with context (available since V3.5):

```q
q)2+"hi"
'type
  [0]  2+"hi"
        ^
```

The system shows the error string, stack frame index, source code, and a caret marking the failed primitive. File paths and function names appear when available:

```q
q)myfun"hi"
'type
  [1]  /kdb+3.5/test.q:5: myfun={2+x}
                                  ^
```

Nested anonymous lambdas inherit their enclosing function's name with an `@` suffix.

## Debugger Session

When suspended in the debugger, you can:

- **Inspect variables** in local scope
- **Navigate the stack** using `` ` `` (up) and `.` (down)
- **Access error context** via `.z.ex` (failed primitive) and `.z.ey` (argument list)

```q
q))a*4
24
q))` / up one frame
  [1]  f:{g[x;2#y]}
          ^
```

## Control Commands

**Signal an error** with `'err`:
```q
q))'myerror
```

**Resume execution** with `:e` (defaults to null `::` if omitted):
```q
q)):42
```

**Abort and exit** using `\`:
```q
q))\
q)
```

Note that resume does not return from the enclosing function—execution continues with the provided value as the failed operation's result.

## Stack Frame Operations

### Backtrace

"[`.Q.bt[]`](../../ref/dotq/#bt-backtrace) will dump the backtrace to stdout at any point during execution or debug. It will highlight the current stack frame with `>>`."

```q
q)g:{a:x*2;a+y}
q)f:{{.Q.bt[];x*2}x+1}
q)f 4
  [2]  f@:{.Q.bt[];x*2}
           ^
  [1]  f:{{.Q.bt[];x*2}x+1}
          ^
  [0]  f 4
       ^
10
```

### Frame Information

The `&` command displays current frame details:

```q
q))&
'type
  [1]  g:{a:x*2;a+y}
                 ^
```

### Backtrace with Error Handling

"[`.Q.trp[f;x;g]`](../../ref/dotq/#trp-extend-trap-at) extends [`trap at` (`@[f;x;g]`)](../../ref/apply/#trap-at) to collect backtrace. Along with the error string, `g` gets called with the backtrace object as a second argument."

```q
q)f:{`hello+x}
q).Q.trp[f;2;{2@"error: ",x,"\nbacktrace:\n",.Q.sbt y;-1}]
error: type
backtrace:
  [2]  f:{`hello+x}
                ^
  [1]  (.Q.trp)

  [0]  .Q.trp[f;2;{2@"error: ",x,"\nbacktrace:\n",.Q.sbt y;-1}]
       ^
-1
```

## Error Trap Modes

The internal error-trap mode governs `'` (Signal) behavior:

- **Mode 0**: Abort execution (set by `@` or `.` traps)
- **Mode 1**: Suspend and run debugger
- **Mode 2**: Collect stack trace and abort (set by `.Q.trp`)

"Mode 2 (dump stack trace) is now default for loading scripts non-interactively (e.g. with [`-q`](../cmdline/#-q-quiet-mode))."

Set the mode for async and HTTP callbacks with `\e`:

```q
q)\e 2
q)'type
  [2]  f@:{x*y}
            ^
  [1]  f:{{x*y}[x;3#x]}
          ^
  [0]  f `a
       ^
```

Debuggers nest automatically when errors occur at the debug prompt, indicated by additional parentheses (`q)))`, `q))))`, etc.).
