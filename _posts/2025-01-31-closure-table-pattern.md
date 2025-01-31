---
title: " 👀 [en] Optimizing a Hierarchical Data Model with the Closure Table Pattern. "

categories:
  - MySQL
  - Database
layout: single
classes: wide
last_modified_at: now
---

**Querying Hierarchical Data Without Recursion!**

When we design a database with parent-child relationships, we often use `parent_id` and `child_id` in each row.  
A common way to query hierarchical data is using a recursive CTE (Common Table Expression).  
However, recursive queries can become complex and hard to maintain as the service grows.

A more efficient alternative is the Closure Table Pattern.

### Closure Table

The closure table pattern stores all ancestor-descendant paths in a separate table.  
This makes queries simpler. We do not need recursion anymore.

Here is an example:

```sql
    A
   / \
  B   C
 /
D
```

**closure table** can look like this:

| ancestor | descendant | depth |
|----------|-----------|------|
| A        | A         | 0    |
| A        | B         | 1    |
| A        | C         | 1    |
| A        | D         | 2    |
| B        | B         | 0    |
| B        | D         | 1    |
| C        | C         | 0    |
| D        | D         | 0    |

The biggest advantage of the closure table is that it is easy to read.
- No recursion needed
- Only Simple SELECT queries


### Case Study

- Find All Descendants of `A`

```sql
SELECT descendant 
FROM closure
WHERE ancestor = 'A';
```

- Find Direct Children of `A`

```sql
select descendant 
from closure
where ancestoer = 'A' and dpeth = 1
```

- Find All Ancestors of `D`

```sql
select ancestor
from closure
where descendant = 'D'
```

- Find Direct Parent of `D`

```sql
select ancestor
from closure
where descendant = 'D' and depth = 1
```

While closure tables make SELECT queries fast and easy, INSERT and UPDATE become more complex.

But always remember: there is no silver bullet.

Choose the best approach based on your service needs.
