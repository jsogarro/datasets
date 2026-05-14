# differ

## Overview

The `differ` function identifies positions where consecutive items in a list change value, returning a boolean list.

## Syntax

```q
differ x
differ[x]
```

## Description

`differ` returns a boolean list showing where consecutive pairs of items in `x` differ. It works with all data types and is a uniform function.

The logic for each position is:
- First item: always `1b`
- Other positions: `1b` if current item differs from previous item, `0b` if they match

## Examples

```q
q)differ`IBM`IBM`MSFT`CSCO`CSCO
10110b

q)differ 1 3 3 4 5 6 6
1101110b

q)differ (7;`a;`a;09:34)
1101b
```

## Practical Usage

Combine `differ` with Cut (`_`) to split tables by changing values:

```q
q)d:2009.10.01+asc 100?30
q)s:100?`IBM`MSFT`CSCO
q)t:([]date:d;sym:s;price:100?100f;size:100?1000)
q)i:where differ t[`date]
q)tlist:i _ t

q)tlist 0
date       sym  price    size
-----------------------------
2009.10.01 IBM  37.95179 710
2009.10.01 CSCO 52.908   594
2009.10.01 MSFT 32.87258 250
2009.10.01 CSCO 75.15704 592

q)tlist 1
date       sym  price   size
----------------------------
2009.10.02 MSFT 18.9035 26
2009.10.02 CSCO 12.7531 760
```

## Technical Details

- **Domain:** B G X H I J E F C S P M D Z N U V T
- **Range:** B B B B B B B B B B B B B B B B B B
- Multithreaded primitive
- Binary application is deprecated as of V3.6; use Match Each Prior (`~:'`) instead
