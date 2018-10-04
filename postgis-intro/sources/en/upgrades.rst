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
      --old-datadir "C:/Program Files/PostgreSQL/8.4/data"
      --new-datadir "C:/Program Files/PostgreSQL/9.0/data"
      --old-bindir "C:/Program Files/PostgreSQL/8.4/bin"
      --new-bindir "C:/Program Files/PostgreSQL/9.0/bin"

* If pg_upgrade_ generated any ``.sql`` files, run them now.
* Start the new server.


Upgrading PostGIS
-----------------

There are two upgrade scenarios for PostGIS too, but they are slightly different from the PostgreSQL scheme:

There are two kinds of PostGIS upgrade scenarios:

* A "minor or patch upgrade" is when the software version increases at the "patch" or "minor" level. For example, from 2.0.1 to 2.0.2, or from 2.0.2 to 2.1.0. Patch upgrades fix bugs only and do not add new features. Minor upgrades fix add new features or change behaviour.
* A "major upgrade" is when the "major" version increases. This is extremely rare. For example, from 0.9.4 to 1.0.0, or from 1.5.4 to 2.0.0. Major upgrades change the on-disk storage format for geometries and require a full database dump and restore.


Minor/Patch PostGIS Upgrades
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PostGIS deals with minor and upgrades through the ``EXTENSION`` mechanism. If you spatially-enabled your database using ``CREATE EXTENSION postgis``, you can update your database using the same functionality.

First, install the new software so it is available to the database.

Then, run the SQL to upgrade your PostGIS extension.

.. code-block:: sql

  -- To upgrade to 2.1.2
  ALTER EXTENSION postgis UPDATE TO '2.1.2';



Major PostGIS Upgrades
~~~~~~~~~~~~~~~~~~~~~~

Major upgrades involve changes to the actual data format for the on-disk storage of geometry and geography data. As such, the data tables need to be re-written. The only way to achieve this is to dump (creating a neutral text-based output) and restore (writing the new table format to disk).

To upgrade, you will have to dump your data first, as discussed in :ref:`backup`.

With Data in Schemas
********************

* Dump your data by schema.

  ::
 
    pg_dump 
       --port=54321
       --type=compressed 
       --file=yourschema.backup 
       --schema=yourschema 
       yourdatabase

* Install the new version of the PostGIS software.
* Create a new blank database, and enable PostGIS in it.
* Load your data using pg_restore.

  ::

    pg_restore
      --port=54321
      --type=compressed 
      --dbname=yournewdatabase
      yourschema.backup

Without Data in Schemas
***********************

In this case you have to dump the whole database, which means the dump file will contain PostGIS function and type signatures, and old ones at that. Before loading that file back into the new database, we strip out all the PostGIS-specific bits using a magic script from the PostGIS distribution.

* Dump your whole database, using the "compressed" backup format.

  ::

    pg_dump 
       --port=54321 
       --type=compressed 
       --file=yourdatabase.backup yourdatabase

* Install the new version of the PostGIS software.
* Filter your database backup using the ./utils/postgis_restore.pl script from the new version of PostGIS.

  ::

    postgis_restore.pl yourdatabase.backup > yourdatabase.sql

* Create a new blank database, and enable PostGIS in it.

  .. code-block:: sql

    -- New in PostGIS 2+ / PgSQL 9.1+
    -- Formal extensions replace hand loading sql files!
    CREATE EXTENSION postgis;

* Load the filtered data back into the new databaes

  ::

    psql 
       --port=54321 
       --file=yourdatabase.sql 
       --dbname=yournewdatabase


You should now have an upgraded database ready to use.


PostGIS 2.0 Upgrade Issues
--------------------------

In addition to being a major upgrade, and therefore requiring a dump and restore, PostGIS 2.0 made some major changes to behaviour and functionality, some of which are user facing. If you are upgrading from 1.X to 2.X you'll want to remain aware of these.

Function Signatures
~~~~~~~~~~~~~~~~~~~

Not only are there new functions, to support new features, but PostGIS 2.0 **removed a large number of old function signatures**. In particular, most of the function variants that are not prefixed by "ST\_" have been removed.

For example, ST_Intersects() exists, but Intersects() no longer exists in PostGIS 2.0.

For some client applications, upgrading the software to no longer use the old signatures is not an option. For those users, **it is possible to restore the old signatures**, by loading the ``legacy.sql`` file into your database. 

Loading ``legacy.sql`` reestablishes all the old function signatures as aliases to the new signatures.

Default WKT and WKB
~~~~~~~~~~~~~~~~~~~

Prior to PostGIS 2.0, the default forms of the ST_AsBinary() and ST_AsText() functions were the :term:`OGC` :term:`SFSQL` defined versions, which only supported two dimensions. Running ST_AsBinary() and ST_AsText() on 3-D and 4-D features just caused the extra dimensions to be stripped, and the returns were 2-D.

For PostGIS 2.0, the ISO SQL/MM definitions of ST_AsBinary() and ST_AsText() are used. For 2-D features, the representations are the same, so no changes will be noticed. For 3-D and 4-D features, however, legal representations in ISO SQL/MM exist, so the dimensions will no longer be stripped, and ISO text and binary results will be returned.

For well-known text, that means that the type string will include dimensionality information, and there will be extra ordinates, eg:

::

  POINT Z (0 0 0)
  LINESTRING ZM (0 0 0 0, 1 1 1 1)

For well-known binaries, that means that the type number will be promoted by a multiple of 1000 to indicate the dimensionality.

* 0 implies 2D
* 1000 implied 3D with a Z
* 2000 implies 3D with an M
* 3000 implies 4D

So, for example

* A POINT has type number 1, a POINT ZM has type number 3001
* A LINESTRING has type number 2, LINESTRING M has type number 2002.

Default SRID
~~~~~~~~~~~~

For PostGIS 0.X and 1.X, the SRID assigned to geometries created without specifying an SRID was -1.

For PostGIS 2.X, the SRID assigned to geometries created without specifying an SRID is 0.

This is only important to client applications calling the ST_SRID() function and testing the result.

SRID Range Limits
~~~~~~~~~~~~~~~~~

In order to fit the SRID number into a limited address range in the PostgreSQL system tables, the range of values PostGIS 2.X supports for SRID numbers is actually smaller than the range supported in 1.X. 

Legal user-defined SRIDs in PostGIS 2.X are from 1 to 998999. The top 10000 SRIDs are retained by PostGIS for internal use.


.. _pg_upgrade: http://www.postgresql.org/docs/current/static/pgupgrade.html
