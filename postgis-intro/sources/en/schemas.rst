.. _schemas:

PostgreSQL Schemas
==================

Production databases inevitably have a large number of tables and views, and managing them all in one schema can become unwieldy quickly. Fortunately, PostgreSQL_ includes the concept of a "_Schema".

Schemas are like folders, and can hold tables, views, functions, sequences and other relations.  Every database starts out with one schema, the ``public`` schema.  

.. image:: ./screenshots/schemas.jpg

Inside that schema, the default install of PostGIS creates the ``geometry_columns``, ``geography_columns`` and ``spatial_ref_sys`` metadata relations, as well as all the types and functions used by PostGIS. So users of PostGIS always need access to the public schema.

In the public schema you can also see all the tables we have created so far in the workshop.


Why use Schemas?
----------------

There are two very good reasons for using schemas:

* Data that is managed in a schema is easier to apply bulk actions to. 

  * It's easier to back-up data that's in a separate schema: so volatile data can have a different back-up schedule from non-volatile data. 
  * It's easier to restore data that's in a separate schema: so application-oriented schemas can be separately restored and backed up for time travel and recovery.
  * It's easier to manage application differences when the application data is in a schema: so a new version of software can work off a table structure in a new schema, and cut-over involves a simple change to the schema name.

* Users can be restricted in their work to single schemas to allow isolation of analytical and test tables from production tables.

So for production purposes, keeping your application data separate in schemas improves management; and for user purposes, keeping your users in separate schemas keeps them from treading on each other.


Creating a Data Schema
----------------------

Let's create a new schema and move a table into it.  First, create a new schema in the database:

.. code-block:: sql

  CREATE SCHEMA census;

Next, we will move the ``nyc_census_blocks`` table to the ``census`` schema:

.. code-block:: sql

  ALTER TABLE nyc_census_blocks SET SCHEMA census;

If you're using the :command:`psql` command-line program, you'll notice that ``nyc_census_blocks`` has disappeared from your table listing now! If you're using PgAdmin, you might have to refresh your view to see the new schema and the table inside it.

You can access tables inside schemas in two ways: 

* by referencing them using ``schema.table`` notation
* by adding the schema to your ``search_path``

Explicit referencing is easy, but it gets tiring to type after a while:

.. code-block:: sql

  SELECT * FROM census.nyc_census_blocks LIMIT 1;

Manipulating the ``search_path`` is a nice way to provide access to tables in multiple schemas without lots of extra typing.

You can set the ``search_path`` at run time using the ``SET`` command:

.. code-block:: sql

  SET search_path = census, public;

This ensures that all references to relations and functions are searched in both the ``census`` and the ``public`` schemas. Remember that all the PostGIS functions and types are in ``public`` so we don't want to drop that from the search path.

Setting the search path every time you connect can get tiring too, but fortunately it's possible to permanently set the search path for a user:

.. code-block:: sql

  ALTER USER postgres SET search_path = census, public;

Now the postgres user will always have the ``census`` schema in their search path.


Creating a User Schema
----------------------

Users like to create tables, and PostGIS users do so particularly: analysis operations with SQL demand temporary tables for visualization or interim results, so spatial SQL tends to require that users have CREATE privileges more than ordinary database workloads.

By default, every role in Oracle is given a personal schema. This is a nice practice to use for PostgreSQL users too, and is easy to replicate using PostgreSQL roles, schemas, and search paths.

Create a new user with table creation privileges (see :ref:`security` for information about the ``postgis_writer`` role), then create a schema with that user as the authorization:

.. code-block:: sql

  CREATE USER myuser WITH ROLE postgis_writer;
  CREATE SCHEMA myuser AUTHORIZATION myuser;

If you log in as that user, you'll find the default ``search_path`` for PostgreSQL is actually this:

.. code-block:: sql

  show search_path;

:: 

    search_path   
  ----------------
   "$user",public
  
The first schema on the search path us the user's named schema! So now the following conditions exist:

* The user exists, with the ability to create spatial tables.
* The user's named schema exists, and the user owns it.
* The user's search path has the user schema first, so new tables are automatically created there, and queries automatically search there first.

That's all there is to it, the user's default work area is now nicely separated from any tables in other schemas.

.. _Schema: http://www.postgresql.org/docs/current/static/ddl-schemas.html
.. _PostgreSQL: http://www.postgresql.org/
