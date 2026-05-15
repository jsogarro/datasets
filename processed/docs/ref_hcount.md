# hcount

## Overview

The `hcount` function returns the size of a file in bytes.

## Syntax

```q
hcount x
hcount[x]
```

## Parameters

`x` is a file symbol representing the target file.

## Returns

A long integer representing the file size in bytes.

## Description

When applied to a file symbol, `hcount` provides the size measurement of that file. According to the documentation, "On a compressed/encrypted file returns the size of the original uncompressed/unencrypted file."

## Example

```q
q)hcount`:c:/q/test.txt
42
```

This returns 42 bytes as the size of the file located at `c:/q/test.txt`.

## Related Topics

- [File system](../../basics/files/)
- [File compression](../../kb/file-compression/)
- [Data at rest encryption (DARE)](../../kb/dare/)
