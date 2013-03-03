---
layout: post
title: How to Re-enable all Constraints in SQL Server
---

How to Re-enable all Constraints in SQL Server
==============================================

``` sql
exec sp_msforeachtable 'ALTER TABLE ? WITH CHECK CHECK CONSTRAINT all'
```

[Source](http://stackoverflow.com/questions/1098554/sql-server-how-to-make-server-check-all-its-check-constraints)