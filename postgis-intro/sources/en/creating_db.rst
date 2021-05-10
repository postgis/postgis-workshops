.. _creating_db:

Creating a Spatial Database
===========================

PgAdmin
-------

PostgreSQL has a number of administrative front-ends.  The primary one is `psql <http://www.postgresql.org/docs/current/static/app-psql.html>`_, a command-line tool for entering SQL queries.  Another popular PostgreSQL front-end is the free and open source graphical tool `pgAdmin <http://www.pgadmin.org/>`_. All queries done in pgAdmin can also be done on the command line with ``psql``. 

#. Find pgAdmin and start it up.

   .. image:: ./screenshots/pgadmin_01.png
     :class: inline

#. If this is the first time you have run pgAdmin, you probably don't have any servers configured. Right click the ``Servers`` item in the Browser panel.
   
   We'll name our server **PostGIS**. In the Connection tab, enter the ``Host name/address``. If you're working with a local PostgreSQL install, you'll be able to use ``localhost``. If you're using a cloud service, you should be able to retrieve the host name from your account.

   Leave **Port** set at ``5432``, and both **Maintenance database** and **Username** as ``postgres``. The **Password** should be what you specified with a local install or with your cloud service.

   .. image:: ./screenshots/pgadmin_02a.png
      :class: inline

Creating a Database
-------------------

#. Open the Databases tree item and have a look at the available databases.  The ``postgres`` database is the user database for the default postgres user and is not too interesting to us.  

#. Right-click on the ``Databases`` item and select ``New Database``.

   .. image:: ./screenshots/pgadmin_02.png
     :class: inline

#. Fill in the ``Create Database`` form as shown below and click **OK**.  

   .. list-table::

     * - **Name**
       - ``nyc``
     * - **Owner**
       - ``postgres``


   .. image:: ./screenshots/pgadmin_03.png
     :class: inline

#. Select the new ``nyc`` database and open it up to display the tree of objects. You'll see the ``public`` schema.

   .. image:: ./screenshots/pgadmin_04.png

#. Click on the SQL query button indicated below (or go to *Tools > Query Tool*).

   .. image:: ./screenshots/pgadmin_05.png

#. Enter the following query into the query text field to load the PostGIS spatial extension:

   .. code-block:: sql

     CREATE EXTENSION postgis;
           
#. Click the **Play** button in the toolbar (or press **F5**) to "Execute the query." 

#. Now confirm that PostGIS is installed by running a PostGIS function:

   .. code-block:: sql

     SELECT postgis_full_version();

You have successfully created a PostGIS spatial database!!


Function List
-------------

`PostGIS_Full_Version <http://postgis.net/docs/PostGIS_Full_Version.html>`_: Reports full PostGIS version and build configuration info.
