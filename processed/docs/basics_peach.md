# Parallel Processing in kdb+

## Overview

The [Each Parallel](../../ref/maps/#each-parallel) operator `':` (or keyword `peach`) enables parallel execution by delegating work to secondary tasks. This approach proves valuable for computationally intensive functions or simultaneous multi-drive access from a single CPU.

Starting kdb+ with multiple secondary processes requires the [`-s` command-line option](../cmdline/#-s-secondary-threads) and the [`\s` system command](../syscmds/#s-number-of-secondary-threads) (available since V3.5).

Each Parallel processes a unary function by distributing the argument list items across secondary processes. The outcome of `m':[x]` matches exactly with `m'[x]`; performance remains identical if no secondary tasks exist.

**Syntax:** `(f':) x`, `f':[x]`, `f peach x`

where `f` represents a unary value and list `x` items fall within its domain.

### Example Performance Comparison

```q
f:{sum exp x?1.0}
\t f each 2#1000000
132
\t f peach 2#1000000     / with 2 CPUs
70
```

## Threads

### Globals

The main kdb+ thread exclusively updates global variables. Functions executed via `peach` restrict updates to local variables only:

```q
{`a set x} peach enlist 0    / works - single item executes on main thread
{`a set x} peach 0 1         / fails - signals noupdate error
```

When no secondary threads exist at startup, `peach` defaults to `each`, executing on the main thread only. This makes the second example functional in single-threaded mode.

Symbol grouping algorithms differ between secondary threads and the main thread. The main thread uses optimizations unavailable to secondary threads, affecting performance metrics.

### Number of Cores/Secondary Threads

With uniform job completion times, total execution quantizes by `#jobs mod #cores`. For instance, using 4 cores, 12 jobs execute similarly to 9 jobs (assuming sufficient secondary processes).

### Sockets and Handles

"A handle must not be used concurrently between threads as there is no locking around a socket descriptor, and the bytes being read/written from/to the socket will be garbage (due to message interleaving) and most likely result in a crash."

Starting with V3.0, sockets function exclusively from the main thread or via one-shot sync requests like:

```q
`:localhost:5000 "2+2"
```

`peach` underpins multithreaded HDB implementations. This example illustrates parallel date-based query execution:

```q
{select max price by date,sym from trade where date=d} peach date
```

### Memory Usage

Each secondary thread maintains its own heap with a 64MB minimum. Since V2.7 2011.09.21, [`.Q.gc[]`](../../ref/dotq/#gc-garbage-collect) in the main thread also collects garbage in secondary threads.

Automatic garbage collection (triggered by `wsfull` or the [`-w` heap limit](../cmdline/#-w-workspace)) executes per-thread only, not globally.

Symbols internalize from a shared memory area across all threads.

## Processes (Distributed Each)

Since V3.1, `peach` supports multiprocess execution via negative integers in the [`-s` option](../cmdline/#-s-secondary-threads), such as `-s -4`.

Unlike multithreading, workload distribution occurs dynamically as secondary processes complete items, rather than precalculated. All function-required data must exist on secondary processes or pass as arguments; minimize argument sizes due to IPC overhead.

The primary use case involves multiprocess HDBs with uncompressed data and [`.Q.MAP[]`](../../ref/dotq/#map-maps-partitions).

Secondary processes require explicit startup with [`.z.pd`](../../ref/dotz/#zpd-peach-handles) configured as either a handle vector or function returning one. These handles serve exclusively for `peach` communication; other messages cause closure:

```q
.z.pd:{n:abs system"s";$[n=count handles;handles;[hclose each handles;:handles::`u#hopen each 20000+til n]]}
.z.pc:{handles::`u#handles except x;}
handles:`u#`int$();
```

---

**Related Resources:** [`.Q.fc` (parallel on cut)](../../ref/dotq/#fc-parallel-on-cut), _Q for Mortals_ §A.68 `peach`
