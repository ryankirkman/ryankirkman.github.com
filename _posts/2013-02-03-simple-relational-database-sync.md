---
layout: post
title: Simple Relational Database Synchronization
---

Simple Relational Database Sync
===============================

I recently designed a system that facilitates simple synchronization (sync) between databases. The techniques used are general enough to be used in any relational database system.

The sync system was originally designed for relatively low transaction slave databases and an aggregation master database. An example would be syncing user preferences, browsing history and bookmarks between computers in Google Chrome.


Architecture
------------

The system is designed to sync individual tables, rather than entire databases at a time. This allows simple, flexible control over what is synced.

The architecture required for this system is:

* A master database
* One or more slave databases
* An interface to query the master database
  * e.g. A HTTP service that speaks JSON
* A messaging queue (or publish / subscribe system)
  * Slaves send changes to the queue
  * e.g. [Azure Service Bus](http://www.windowsazure.com/en-us/home/features/messaging/), [Amazon SQS](http://aws.amazon.com/sqs/)
* A message consumer
  * Processes messages from slaves and updates the master database as necessary


Common Table Structure
----------------------

Each table in the database to be synced has a SequenceId column of integer type and an IsDeleted column of boolean type.

SequenceId is not a timestamp, but a greater SequenceId indicates a record has been modified more recently. It is analogous to the [rowversion](http://msdn.microsoft.com/en-us/library/ms182776.aspx) data type in SQL Server. Every time a row is Inserted, Updated or logically deleted (via the IsDeleted column), SequenceId is set to:

``` sql
MAX(SequenceId) + 1
```

The IsDeleted column allows logical deletion of rows. This is a core requirement of the sync strategy. It allows us to sync Deletes which would otherwise be lost if we simply deleted the row. Logical Deletes provide a number of other benefits of their own accord:

* Slaves can choose to physically delete a logically Deleted row if storage space is at a premium
* Seeing which rows were deleted can be part of a built-in audit system
* Rows can be undeleted


General Overview of the Sync Strategy
-------------------------------------

__For sending changes from slaves -> master__

* When a slave Inserts, Updates or Deletes a row it sends a message to the messaging queue
  * The message contains the operation (e.g. INSERT, UPDATE, DELETE) and a copy of the modified row
* A service consumes messages in the queue, performing the appropriate operation with the row on the master database (e.g. INSERT row, UPDATE row, logically DETLE row)



__For pulling changes from master -> slave__

The following operation is best encapsulated in a "Master" HTTP sync service for maximum flexibility and modularization:

``` sql
SELECT *
FROM [Table]
WHERE SequenceId > Current_Slave_SequenceId
```


Syncing Changes to Slave Clients
--------------------------------

The slave client will receive a collection of rows in whatever format has been requested from the master HTTP sync service.

Because we don't know which of the rows in the new collection the slave already has, we have a few choices:

* Attempt to INSERT the record
  * If a primary key violation occurs, attempt to UPDATE the record
* Check first to see if a row exists with the primary key of the "new" row
  * If the primary key already exists, attempt to UPDATE rather than INSERT


Potential Performance Optimizations
-----------------------------------
Because a service sits between the slaves and the master database, high traffic tables could be moved onto their own shard without changing the interface of the HTTP sync service. These changes would be transparent to slave clients.