.. _tuning:

Tuning PostgreSQL for Spatial
=============================

PostgreSQL is a very versatile database system, capable of running efficiently in very low-resource environments and environments shared with a variety of other applications.  In order to ensure it will run properly for many different environments, the default configuration is very conservative and not terribly appropriate for a high-performance production database.  Add the fact that geospatial databases have different usage patterns, and the data tend to consist of fewer, much larger records than non-geospatial databases, and you can see that the default configuration will not be totally appropriate for our purposes.  

All of these configuration parameters can edited in the `postgresql.conf` database configuration file. This is a regular text file and can be edited using Notepad or any other text editor.  The changes will not take effect until the server is restarted.

.. image:: ./tuning/conf01.png

An easier way of editing this configuration is by using the built-in "Backend Configuration Editor".  In pgAdmin, go to *File > Open postgresql.conf...*.  It will ask for the location of the file, and navigate to :file:`postgresql.conf`.

.. image:: ./tuning/conf02.png

.. image:: ./tuning/conf03.png

This section describes some of the configuration parameters that should be adjusted for a production-ready geospatial database.  For each section, find the appropriate item in the list, double-click on the line to edit the configuration.  Change the *Value* to the recommended value as described, make sure the item is *Enabled*, then click **OK**.

.. note:: These values are recommendations only; each environment will differ and testing is required to determine the optimal configuration.  But this section should get you off to a good start.

shared_buffers
--------------

Sets the amount of memory the database server uses for shared memory buffers.  These are shared amongst the back-end processes, as the name suggests.  The default values are typically woefully inadequate for production databases.

  *Default value*: typically 32MB

  *Recommended value*: 75% of database memory (500MB)

.. image:: ./tuning/conf04.png

work_mem
--------

Defines the amount of memory that internal sorting operations and hash tables can consume before the database switches to on-disk files.  This value defines the available memory for each operation; complex queries may have several sort or hash operations running in parallel, and each connected session may be executing a query.

As such you must consider how many connections and the complexity of expected queries before increasing this value.  The benefit to increasing is that the processing of more of these operations, including ORDER BY, and DISTINCT clauses, merge and hash joins, hash-based aggregation and hash-based processing of subqueries, can be accomplished without incurring disk writes.

  *Default value*: 1MB

  *Recommended value*: 16MB

.. image:: ./tuning/conf05.png

maintenance_work_mem
--------------------

Defines the amount of memory used for maintenance operations, including vacuuming, index and foreign key creation.  As these operations are not terribly common, the default value may be acceptable.  This parameter can alternately be increased for a single session before the execution of a number of :command:`CREATE INDEX` or :command:`VACUUM` calls as shown below.

  .. code-block:: sql

    SET maintenance_work_mem TO '128MB';
    VACUUM ANALYZE;
    SET maintenance_work_mem TO '16MB';

  *Default value*: 16MB

  *Recommended value*: 128MB

.. image:: ./tuning/conf06.png

wal_buffers
-----------

Sets the amount of memory used for write-ahead log (WAL) data.  Write-ahead logs provide a high-performance mechanism for insuring data-integrity.  During each change command, the effects of the changes are written first to the WAL files and flushed to disk.  Only once the WAL files have been flushed will the changes be written to the data files themselves.  This allows the data files to be written to disk in an optimal and asynchronous manner while ensuring that, in the event of a crash, all data changes can be recovered from the WAL.  

The size of this buffer only needs to be large enough to hold WAL data for a single typical transaction.  While the default value is often sufficient for most data, geospatial data tends to be much larger.  Therefore, it is recommended to increase the size of this parameter.

  *Default value*: 64kB

  *Recommended value*: 1MB

.. image:: ./tuning/conf07.png

checkpoint_segments
-------------------

This value sets the maximum number of log file segments (typically 16MB) that can be filled between automatic WAL checkpoints.  A WAL checkpoint is a point in the sequence of WAL transactions at which it is guaranteed that the data files have been updated with all information before the checkpoint.  At this time all dirty data pages are flushed to disk and a checkpoint record is written to the log file.  This allows the crash recovery process to find the latest checkpoint record and apply all following log segments to complete the data recovery.

Because the checkpoint process requires the flushing of all dirty data pages to disk, it creates a significant I/O load.  The same argument from above applies; geospatial data is large enough to unbalance non-geospatial optimizations.  Increasing this value will prevent excessive checkpoints, though it may cause the server to restart more slowly in the event of a crash.

  *Default value*: 3

  *Recommended value*: 6

.. image:: ./tuning/conf08.png

random_page_cost
----------------

This is a unit-less value that represents the cost of a random page access from disk.  This value is relative to a number of other cost parameters including sequential page access, and CPU operation costs.  While there is no magic bullet for this value, the default is generally conservative.  This value can be set on a per-session basis using the ``SET random_page_cost TO 2.0`` command.

  *Default value*: 4.0

  *Recommended value*: 2.0

.. image:: ./tuning/conf09.png

seq_page_cost
-------------

This is the parameter that controls the cost of a sequential page access.  This value does not generally require adjustment but the difference between this value and ``random_page_cost`` greatly affects the choices made by the query planner.  This value can also be set on a per-session basis.

  *Default value*: 1.0

  *Recommended value*: 1.0

.. image:: ./tuning/conf10.png

Reload configuration
--------------------

After these changes are made, save changes and reload the configuration. The easiest way to do this is to restart the PostgreSQL service.

* In pgAdmin, right-click the server **PostGIS (localhost:5432)** and select *Disconnect*.
* In Windows Services (``services.msc``) right-click **PostgreSQL** and select *Restart*.
* Back in pgAdmin, click the server again select *Disconnect*.
