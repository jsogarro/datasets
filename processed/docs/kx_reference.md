# Kx Reference

# Reference card¶

Note

Want a quick and handy q reference resource? Download the q Cheat Sheet (PDF download).

## Keywords¶

abscorejgtimelikeminsprevscovsystemwavg

acoscosemahcloselj ljfmmaxpriorsdevtableswhere

aj aj0countenlisthcountloadmminrandselecttanwhile

ajf ajf0covevalhdellogmmuranksettilwithin

allcrossexcepthopenlowermodratiossetenvtrimwj wj1

andcsvexechsymlsqmsumrazeshowtypewsum

anycutexitiascltimenegread0signumuj ujfxasc

ascdeleteexpidescltrimnextread1sinungroupxbar

asindeltasfbyifmavgnotreciprocalsqrtunionxcol

asofdescfillsij ijfmaxnullrevalssupdatexcols

atandevfirstinmaxsorreversessrupperxdesc

attrdifferfkeysinsertmcountoverrloadstringupsertxexp

avgdistinctflipintermd5parserotatesublistvaluexgroup

avgsdivfloorinvmdevpeachrsavesumvarxkey

bin binrdogetkeymedpjrtrimsumsviewxlog

ceilingdsavegetenvkeysmetaprdsavesvviewsxprev

colseachgrouplastminprdsscansvarvsxrank

### By category¶

controldo, exit, if, while

envgetenv, gtime, ltime, setenv

interpreteval, parse, reval, show, system, value

iodsave, get, hclose, hcount, hdel, hopen, hsym, load, read0, read1, rload, rsave, save, set

iterateeach, over, peach, prior, scan

joinaj aj0, ajf ajf0, asof, ej, ij ijf, lj ljf, pj, uj ujf, wj wj1

listcount, cross, cut, enlist, except, fills, first, flip, group, in, inter, last, mcount, next, prev, raze, reverse, rotate, sublist, sv, til, union, vs, where, xprev

logicall, and, any, not, or

mathabs, acos, asin, atan, avg, avgs, ceiling, cor, cos, cov, deltas, dev, div, ema, exp, floor, inv, log, lsq, mavg, max, maxs, mdev, med, min, mins, mmax, mmin, mmu, mod, msum, neg, prd, prds, rand, ratios, reciprocal, scov, sdev, signum, sin, sqrt, sum, sums, svar, tan, var, wavg, within, wsum, xexp, xlog

metaattr, null, tables, type, view, views

querydelete, exec, fby, select, update

sortasc, bin binr, desc, differ, distinct, iasc, idesc, rank, xbar, xrank

tablecols, csv, fkeys, insert, key, keys, meta, ungroup, upsert, xasc, xcol, xcols, xdesc, xgroup, xkey

textlike, lower, ltrim, md5, rtrim, ss, ssr, string, trim, upper

.Q.id (sanitize),
.Q.res (reserved words)

`.Q.id``.Q.res`## Operators¶

`.`Apply, Index, Trap, Amend

`@`Apply At, Index At, Trap At, Amend At

`$`Cast, Tok, Enumerate, Pad, mmu

`mmu``!`Dict, Enkey, Unkey, Enumeration, Flip Splayed, Display, internal, Update, Delete, lsq

`lsq``?`Find, Roll, Deal, Enum Extend, Select, Exec, Simple Exec, Vector Conditional

+ - * %Add, Subtract, Multiply, Divide

`+ - * %`= <> ~Equals, Not Equals, Match

`= <> ~``< <= >= >`Less Than, Up To, At Least, Greater Than

| &Greater (OR), Lesser, AND

`| &``#`Take, Set Attribute

`_`Cut, Drop

`:`Assign

`^`Fill, Coalesce

`,`Join

`'`Compose

0: 1: 2:File Text, File Binary, Dynamic Load

`0: 1: 2:`0 ±1 ±2 ±nwrite to console, stdout, stderr, handle n

`0 ±1 ±2 ±n`.: @: $: !: ?: +: -: *: %: =: ~: <: >: |: &: #: _: ^: ,:Assign through operator

`.: @: $: !: ?: +: -: *: %: =: ~: <: >: |: &: #: _: ^: ,:`Overloaded glyphs

## Iterators¶

maps accumulators
' Each, each, Case /: Each Right / Over, over
': Each Parallel, peach \: Each Left \ Scan, scan
': Each Prior, prior

`'``each``/:``/``over``':``peach``\:``\``scan``':``prior`## Execution control¶

.[f;x;e] Trap : Return do exit $[x;y;z] Cond
@[f;x;e] Trap-At ' Signal if while :[v;p1;r1;...] Pattern conditional

Debugging

## Other¶

`   pop stack :: identity \x  system cmd x
. push stack generic null \    abort
global amend     \\   quit q
                          set view         /    comment

()     precedence    [;]  expn block       {}  lambda       `   symbol
(;)    list argt list        ;   separator    `:  filepath
([]..) table

## Attributes¶

g grouped     p parted     s sorted     u unique

Set Attribute

## Command-line options and system commands¶

file

\atables\rrename

`\a``\r`-bblocked-s \ssecondary processes

`-b``-s``\s`\b \Bviews-S \Srandom seed

`\b``\B``-S``\S`-c \cconsole size-t \ttimer ticks

`-c``\c``-t``\t`-C \CHTTP size\tstime and space

`-C``\C``\ts`\cdchange directory-T \Ttimeout

`\cd``-T``\T`\ddirectory-u -U \uusr-pwd

`\d``-u``-U``\u`-e \eerror traps-udisable syscmds

`-e``\e``-u`-E \ETLS server mode\vvariables

`-E``\E``\v`\ffunctions-w \wmemory

`\f``-w``\w`-g \ggarbage collection-W \Wweek offset

`-g``\g``-W``\W`\lload file or directory\xexpunge

`\l``\x`-l -Llog sync-z \zdate format

`-l``-L``-z``\z`-o \oUTC offset\1 \2redirect

`-o``\o``\1``\2`-p \plistening port\_hide q code

`-p``\p``\_`-P \Pdisplay precision\terminate

`-P``\P``\`-qquiet mode\toggle q/k

`-q``\`-r \rreplicate\\quit

`-r``\r``\\`system

`system`Command-line options,
System commands,
OS commands

## Datatypes¶

Basic datatypes
n   c   name      sz  literal            null inf SQL       Java      .Net
------------------------------------------------------------------------------------
0   *   list
1   b   boolean   1   0b                                    Boolean   boolean
2   g   guid      16                     0Ng                UUID      GUID
4   x   byte      1   0x00                                  Byte      byte
5   h   short     2   0h                 0Nh  0Wh smallint  Short     int16
6   i   int       4   0i                 0Ni  0Wi int       Integer   int32
7   j   long      8   0j                 0Nj  0Wj bigint    Long      int64
                      0                  0N   0W
8   e   real      4   0e                 0Ne  0We real      Float     single
9   f   float     8   0.0                0n   0w  float     Double    double
                      0f                 0Nf
10  c   char      1   " "                " "                Character char
11  s   symbol        `                  `        varchar
12  p   timestamp 8   dateDtimespan      0Np  0Wp           Timestamp DateTime (RW)
13  m   month     4   2000.01m           0Nm
14  d   date      4   2000.01.01         0Nd  0Wd date      Date
15  z   datetime  8   dateTtime          0Nz  0wz timestamp Timestamp DateTime (RO)
16  n   timespan  8   00:00:00.000000000 0Nn  0Wn           Timespan  TimeSpan
17  u   minute    4   00:00              0Nu  0Wu
18  v   second    4   00:00:00           0Nv  0Wv
19  t   time      4   00:00:00.000       0Nt  0Wt time      Time      TimeSpan

Columns:
n    short int returned by type and used for Cast, e.g. 9h$3
c    character used lower-case for Cast and upper-case for Tok and Load CSV
sz   size in bytes
inf  infinity (no math on temporal types); 0Wh is 32767h

`type``9h$3``0Wh``32767h`RO: read only; RW: read-write

Other datatypes
20-76   enums
77      anymap                                      104  projection
78-96   77+t – mapped list of lists of type t       105  composition
97      nested sym enum                             106  f'
98      table                                       107  f/
99      dictionary                                  108  f\
100     lambda                                      109  f':
101     unary primitive                             110  f/:
102     operator                                    111  f\:
103     iterator                                    112  dynamic load

Above, f is an applicable value.

`f`Nested types are 77+t (e.g. 78 is boolean. 96 is time.)

Cast $: where char is from the c column above char$data:CHAR$string

`$``char``c``char$data:CHAR$string````q
dict:`a`b!…
table:([]x:…;y:…)
date.(year month week mm dd)
time.(minute second mm ss)
milliseconds: time mod 1000
```

## Namespaces¶

### .h (markup)¶

`.h`HTTP, markup and data conversion.

### .j (JSON)¶

`.j`De/serialize as JSON.

### .m (modules)¶

`.m`The currently loaded modules. In kdb+ 4.x, .m is reserved for memory domain 1 objects. In kdb+ 5.0 and later, .m is used for modules.

`.m``.m`### .Q (utils)¶

`.Q`Utilities: general, environment, IPC, datatype, database, partitioned database state, segmented database state, file I/O, debugging, profiling.

### .z (environment, callbacks)¶

`.z`Environment, callbacks

