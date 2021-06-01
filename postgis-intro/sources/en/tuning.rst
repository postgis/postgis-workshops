.. _tuning:

Basic PostgreSQL Tuning
=======================

PostgreSQL is a very versatile database system, capable of running efficiently in very low-resource environments and environments shared with a variety of other applications.  In order to ensure it will run properly for many different environments, the default configuration is very conservative and not terribly appropriate for a high-performance production database.  Add the fact that geospatial databases have different usage patterns, and the data tend to consist of fewer, much larger records than non-geospatial databases, and you can see that the default configuration will not be totally appropriate for our purposes.  

All of these configuration parameters can edited in the `postgresql.conf` database configuration file. This is a regular text file and can be edited using any text editor.  The changes will not take effect until the server is restarted.

This section describes some of the configuration parameters that can be adjusted for a more production-ready geospatial database.

.. note:: These values are recommendations only; each environment will differ and testing is required to determine the optimal configuration.  But this section should get you off to a good start.


shared_buffers
--------------

Sets the amount of memory the database server uses for shared memory buffers.  These are shared amongst the back-end processes, as the name suggests.  The default values are typically woefully inadequate for production databases.

  *Default value*: typically 32MB

  *Recommended value*: about 75% of database memory up to a max of about 2GB


effective_cache_size
--------------------

In addition to the memory PostgreSQL sets aside for ``shared_buffers`` the query planner also takes into account how many disk blocks the operating system may have cached as part of its virtual file system. For systems with large amount of memory, this can be quite large. The ``effective_cache_size`` is approximately the amount of memory on the machine, less the ``shared_buffers``, less the ``work_mem`` times the expected number of connections, less any memory required for any other processes running on the machine, less about 1GB for other random operating system needs. The database will not **use** the extra cache directly, but it will compute plans expecting that the operating system has cached filesystem data in about that much memory.

  *Default value*: typically 4GB

  *Recommended value*: any amount of "free" memory expected to be around under ordinary operating conditions


work_mem
--------

Defines the amount of memory that internal sorting operations, indexing operations and hash tables can consume before the database switches to on-disk files.  This value defines the available memory for each operation; complex queries may have several sort or hash operations running in parallel, and each connected session may be executing a query.

As such you must consider how many connections and the complexity of expected queries before increasing this value.  The **benefit** to increasing is that the processing of more of these operations, including ORDER BY, and DISTINCT clauses, merge and hash joins, hash-based aggregation and hash-based processing of subqueries, can be accomplished without incurring disk writes. The **cost** of increasing is memory that will be used **per connection**, which can be quite high with production levels of connections.

  *Default value*: 1MB

  *Recommended value*: 32MB


maintenance_work_mem
--------------------

Defines the amount of memory used for maintenance operations, including vacuuming, index and foreign key creation.  As these operations are not terribly common, a higher value will only exact an occasional cost, and may substantially speed up maintenance activities  This parameter can alternately be increased for a single session before the execution of a number of :command:`CREATE INDEX` or :command:`VACUUM` calls as shown below.

  .. code-block:: sql

    SET maintenance_work_mem TO '128MB';
    VACUUM ANALYZE;
    SET maintenance_work_mem TO '16MB';

  *Default value*: 16MB

  *Recommended value*: 128MB


wal_buffers
-----------

Sets the amount of memory used for write-ahead log (WAL) data.  Write-ahead logs provide a high-performance mechanism for insuring data-integrity.  During each change command, the effects of the changes are written first to the WAL files and flushed to disk.  Only once the WAL files have been flushed will the changes be written to the data files themselves.  This allows the data files to be written to disk in an optimal and asynchronous manner while ensuring that, in the event of a crash, all data changes can be recovered from the WAL.  

The size of this buffer only needs to be large enough to hold WAL data for a single typical transaction.  While the default value is often sufficient for most data, geospatial data tends to be much larger.  Therefore, it is recommended to increase the size of this parameter.

  *Default value*: 64kB

  *Recommended value*: 1MB


checkpoint_segments
-------------------

This value sets the maximum number of log file segments (typically 16MB) that can be filled between automatic WAL checkpoints.  A WAL checkpoint is a point in the sequence of WAL transactions at which it is guaranteed that the data files have been updated with all information before the checkpoint.  At this time all dirty data pages are flushed to disk and a checkpoint record is written to the log file.  This allows the crash recovery process to find the latest checkpoint record and apply all following log segments to complete the data recovery.

Because the checkpoint process requires the flushing of all dirty data pages to disk, it creates a significant I/O load.  The same argument from above applies; geospatial data is large enough to unbalance non-geospatial optimizations.  Increasing this value will prevent excessive checkpoints, though it may cause the server to restart more slowly in the event of a crash.

  *Default value*: 3

  *Recommended value*: 6


random_page_cost
----------------

This is a unit-less value that represents the cost of a random page access from disk.  This value is relative to a number of other cost parameters including sequential page access, and CPU operation costs.  While there is no magic bullet for this value, the default is generally conservative and for databases running on spinning media. The random access cost for SSD should be set even lower.

This value can be set on a per-session basis using the ``SET random_page_cost TO 2.0`` command, which can be useful for testing how it effects query plans.

  *Default value*: 4.0

  *Recommended value*: 2.0 for spinning media, 1.0 for SSD


seq_page_cost
-------------

This is the parameter that controls the cost of a sequential page access.  This value does not generally require adjustment but the difference between this value and ``random_page_cost`` greatly affects the choices made by the query planner.  This value can also be set on a per-session basis.

  *Default value*: 1.0

  *Recommended value*: 1.0


Reload configuration
--------------------

After these changes are made, save changes and reload the configuration. The easiest way to do this is to restart the PostgreSQL service.

* In pgAdmin, right-click the server **PostGIS (localhost:5432)** and select *Disconnect*.
* In Windows Services (``services.msc``) right-click **PostgreSQL** and select *Restart*.
* Back in pgAdmin, click the server again select *Disconnect*.
