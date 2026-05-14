# group

## Syntax

```q
group x     group[x]
```

## Description

The `group` function returns a dictionary where keys are distinct items from the input `x`, and values are the indexes where each distinct item occurs. Keys appear in the order they first occur in `x`.

## Examples

### Basic Usage

```q
q)group "mississippi"
m| ,0
i| 1 4 7 10
s| 2 3 5 6
p| 8 9
```

### Counting Occurrences

To determine how many times each distinct item appears:

```q
q)count each group "mississippi"
m| 1
i| 4
s| 4
p| 2
```

### First Occurrence Index

To find the index of the first occurrence of each distinct item:

```q
q)first each group "mississippi"
m| 0
i| 1
s| 2
p| 8
```

## Related Functions

- [`ungroup`](../ungroup/)
- [`xgroup`](../xgroup/)

## See Also

[Sorting](../../basics/by-topic/#sort)
