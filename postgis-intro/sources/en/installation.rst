.. _installation:

Installation
============

To explore the PostgreSQL/PostGIS database, and learn about writing spatial queries in SQL, we will need some software, either installed locally or available remotely on the cloud.

* There are instructions below on how to access PostgreSQL for installation on Windows or MacOS. PostgreSQL for Windows and MacOS either include PostGIS or have an easy way to add it on.
* There are instructions below on how to install `PgAdmin <https://www.pgadmin.org/>`_. PgAdmin is a graphical database explorer and SQL editor which provides a "user facing" interface to the database engine that does all the work.

For always up-to-date directions on installing PostgreSQL, go to the `PostgreSQL download page  <https://www.postgresql.org/download/>`_ and select the operating system you are using.


PostgreSQL for Microsoft Windows
--------------------------------

For a Windows install:

#. Go to the `Windows PostgreSQL download page <https://www.enterprisedb.com/downloads/postgres-postgresql-downloads>`_.

#. Select the latest version of PostgreSQL and save the installer to disk.

#. Run the installer and accept the defaults.

#. Find and run the "StackBuilder" program that was installed with the database.

#. Select the "Spatial Extensions" section and choose latest "PostGIS ..Bundle" option.

   .. image:: ./screenshots/install_windows_01.png
     :class: inline

#. Accept the defaults and install.


PostgreSQL for Apple MacOS
--------------------------

For a MacOS install:

#. Go to the `Postgres.app <https://postgresapp.com/>`_ site, and download the latest release.

#. Open the disk image, and drag the **Postgres** icon into the **Applications** folder.

   .. image:: ./screenshots/install_macos_01.png
     :class: inline

#. In the **Applications** folder, double-click the **Postgres** icon to start the server.

#. Click the **Initialize** button to create a new blank database instance.

   .. image:: ./screenshots/install_macos_02.png
     :class: inline, border

#. In the **Applications** folder, go to the **Utilities** folder and open **Terminal**.

#. Add the command-line utilities to your `PATH` for convenience.

  ::

    sudo mkdir -p /etc/paths.d
    echo /Applications/Postgres.app/Contents/Versions/latest/bin | sudo tee /etc/paths.d/postgresapp


PgAdmin for Windows and MacOS
-----------------------------

PgAdmin is available for multiple platforms, at https://www.pgadmin.org/download/.

#. Download and install the latest version for your platform.

#. Start PgAdmin!

   .. image:: ./screenshots/install_pgadmin_01.png
     :class: inline




