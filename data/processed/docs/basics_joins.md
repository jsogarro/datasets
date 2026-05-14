# Joins in kdb+ and q

## Overview

Joins combine data from two tables or a table and dictionary. The documentation describes three primary categories: keyed joins, as-of joins, and implicit joins.

## Keyed Joins

These joins match columns in the first argument against key columns in the second argument:

- **Coalesce (`^`)**: "merges keyed tables ignoring nulls"
- **Equi join (`ej`)**: "Similar to `ij`, where the columns to be matched are given as a parameter"
- **Inner join (`ij`, `ijf`)**: "Joins on the key columns of the second table. The result has one row for each row of the first table that matches"
- **Left join (`lj`, `ljf`)**: "Outer join on the key columns of the second table. The result has one row for each row of the first table"
- **Plus join (`pj`)**: "For each matching row, values from the second table are added to the first table, instead of replacing values"
- **Union join (`uj`, `ujf`)**: "Uses all rows from both tables"
- **Upsert**: Can join tables with matching columns and update or append records

## As-of Joins

These use time columns to specify intervals:

- **Window join (`wj`, `wj1`)**: "The most general forms of as-of join. Function parameters aggregate values in the time intervals"
- **As-of join (`aj`, `aj0`, `ajf`, `ajf0`)**: "Simpler window joins where only the last value in each interval is used"
- **Simple as-of (`asof`)**: "A simpler `aj` where all columns of the second argument are used"

## Implicit Joins

Foreign keys enable automatic joining through enumeration over keyed table columns.
