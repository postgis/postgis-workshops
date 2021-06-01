.. _upgrades:

Software Upgrades
=================

Because PostGIS resides within PostgreSQL every PostGIS installation actually consists of two versions of software: the PostgreSQL version and the PostGIS version.  The OpenGeo Suite only ships one combination at a time, so the number of potential combinations is reduced, but as a general principle, each version of PostGIS can be theoretically run within a number of versions of PostgreSQL, and vice versa.

So, upgrades need to be considered in terms of upgrading each component.


Upgrading PostgreSQL
--------------------

There are two kinds of PostgreSQL upgrade scenarios:

* A "minor upgrade" when the software version increases at the "patch" level. For example, from 8.4.3 to 8.4.4, or from 9.0.1 to 9.0.3. Increases of more than one patch version are just fine. Minor upgrades fix bugs but do not add any new features or change behaviour.
* A "major upgrade" when the "major" or "minor" versions increase. For example, from 8.4.5 to 9.0.0, or from 9.0.5 to 9.1.1. Major upgrades add new features and change behavior.

Minor PostgreSQL Upgrades
~~~~~~~~~~~~~~~~~~~~~~~~~

For "minor upgrades", no special process is necessary. Simply install the new software, and re-start the server. 

Major PostgreSQL Upgrades
~~~~~~~~~~~~~~~~~~~~~~~~~

For "major upgrades" there are two ways to carry out the upgrade.

Dump/Restore
************

Dumping and restoring involves converting all the data to a platform neutral format (text representations) on dump, and back to native representations on restore, so it can be time consuming and CPU intensive. However, if you are migrating to a new architecture or operating system, it's a required process. It's also a time-tested and well-understood upgrade path, so if your database is not too big, there's no reason not to stick with it.

* Dump your data ``pg_dumpall`` from the old database.
* Install the new version of PostgreSQL and the same version of PostGIS you are using in your old database. You need to match the PostGIS version so that the dump file function definitions reference an expected version of the PostGIS library.
* Initialize the new data area using the ``initdb`` program from the new software.
* Start the new server on the new data area.
* Restore the dump file using ``pg_restore``.

pg_upgrade
**********

The pg_upgrade_ utility allows PostgreSQL data directories to be upgraded without the requirement for a dump/restore step. The utility cannot handle changes to the data files themselves, but handles the more common and frequent changes to system tables that occur in PostgreSQL major upgrades.

.. note:: 

  The full instructions for running the upgrade process are in the pg_upgrade_ web page at the PostgreSQL site.

The pg_upgrade_ program expects to have access to both versions of PostgreSQL it is working with, the old and the new version, so you will have to install them both. 

* Install the new version of PostgreSQL you will be using.
* Install the same version of PostGIS you are using in the old PostgreSQL into the new PostgreSQL.
* Initialize the new PostgreSQL data area with the new copy of ``initdb``.
* Ensure both the old and new PostgreSQL servers are turned off.
* Run pg_upgrade_, making sure to use the binary from the new software installation.

  ::
      
    pg_upgrade 
      --old-datadir "/var/lib/postgres/12/data"
      --new-datadir "/var/lib/postgres/13/data"
      --old-bindir "/usr/pgsql/12/bin"
      --new-bindir "/usr/pgsql/13/bin"

* If pg_upgrade_ generated any ``.sql`` files, run them now.
* Start the new server.


Upgrading PostGIS
-----------------

PostGIS deals with minor and upgrades through the ``EXTENSION`` mechanism. If you spatially-enabled your database using ``CREATE EXTENSION postgis``, you can update your database using the same functionality.

First, install the new software so it is available to the database.

Then, run the SQL to upgrade your PostGIS extension.

.. code-block:: sql

  -- If you are upgrading from PostGIS 2.5 or later
  -- and want the latest installed version
  SELECT postgis_extensions_upgrade();

  -- If you are upgrading from an earlier version
  -- you have to specifically turn on the version you want
  ALTER EXTENSION postgis UPDATE TO '2.5.5';


.. _pg_upgrade: http://www.postgresql.org/docs/current/static/pgupgrade.html
