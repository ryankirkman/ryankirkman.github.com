---
layout: post
title: SQL Server CE - Changing INNER JOIN to LEFT JOIN is a 5x speedup
---

SQL Server CE - Changing INNER JOIN to LEFT JOIN is a 5x speedup
================================================================

I was recently working on a fairly complex query in SQL CE 3.5. It was around 8 seconds to complete on a Windows CE device. I had a very similar query in terms of complexity, joins and result set that would complete in around 1 second on the same device.

This led me to compare the queries via their execution plans. There are about 10 joins involved in each query.

The start of the first (slow) query results in an estimated query cost of `0.307772` and looks like:

``` sql
SELECT col1, col2, ...
FROM [Table]
INNER JOIN ...
... join other tables ...
```

The start of the second (fast) query results in an estimated query cost of `0.061847` and looks like:

``` sql
SELECT col1, col2, ...
FROM [Table]
LEFT JOIN ...
... join other tables ...
```

Notice the difference? The second query uses a `LEFT JOIN` instead of an `INNER JOIN`. The semantics of the query dictate an `INNER JOIN`, but it results in a **significantly** slower query plan. The only other source I've found mentioning such a problem says that [it could be a bug in the query optimizer](http://sqlserverselect.blogspot.com.au/2010/10/nested-loops-join-no-join-predicate.html):

  > Other behavior that I noticed was that if I changed the inner join to a left join, the optimizer came up with a different much more efficient plan. This appears to be a flaw in the optimizer but I would like to speak to someone at Microsoft before making that claim.

Note that this may only apply to a certain subset of queries under special circumstances. I haven't done an extensive investigation.