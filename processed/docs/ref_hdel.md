# hdel - Delete a File or Folder

## Syntax

```q
hdel x     hdel[x]
```

## Description

Deletes a file or folder specified by a file symbol atom `x`, and returns `x`.

## Parameters

- `x`: A file symbol atom representing the file or folder to delete

## Examples

Delete a file in the current working directory:

```q
q)hdel`:test.txt
`:test.txt
```

Attempting to delete a non-existent file generates an error:

```q
q)hdel`:test.txt
'test.txt: No such file or directory
```

## Important Constraints

"hdel can delete folders only if empty." To remove a folder with contents, use a recursive approach.

### Recursive Directory Deletion

```q
q)dir:{$[11h=type d:key x;raze x,.z.s each` sv/:x,/:d;d]}
q)nuke:hdel​ ​each​ ​​desc dir​@​
q)nuke`:mydir
```

### Visitor Pattern Approach

```q
q)visitNode:{if[11h=type d:key y;.z.s[x]each` sv/:y,/:d;];x y}
q)nuke:visitNode[hdel]
```

## Platform-Specific Notes

On Windows, memory-mapped files cannot be overwritten immediately after unmapping; a brief delay is required before deletion becomes possible.
