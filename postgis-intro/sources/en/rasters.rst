.. _rasters:

Rasters
=========

PostGIS supports another kind of spatial data type called a *raster*.
Raster data, much like geometry data, uses **Cartesian coordinates** and a spatial reference system.
However instead of vector data, raster data is represented as an n-dimensional matrix consisting of pixels and bands.
The bands defines the number of matrices you have. Each pixel stores a value corresponding to each band.
So a 3-banded raster such as an RGB image, would have 3 values for each pixel corresponding to the Red-Green-Blue bands.

Although pretty pictures such as those you see on your TV screen are rasters, rasters may not be that exciting to look at.
In a nutshell, a raster is a matrix, pinned on a coordinate system, that has values that can represent anything you want them to represent.

Since rasters live in cartesian space, rasters can interact with geometries.  PostGIS offers many functions that take as input both rasters and geometries.
Also many operations applied to rasters will result in geometries. Common ones are the `ST_Polygon`, `ST_Envelope`, `ST_ConvexHull`, and `ST_MinConvexHull`
as shown below.

.. image:: ./rasters/postgis_raster.jpg

The raster format is commonly used to store elevation data, temperature data, satellite data, and thematic data representing things like environmental contamination, population density, and environmental hazard occurrences.

Although raster data can be created from scratch in PostGIS, a more common approch is to load raster data from various formats using the `raster2pgsql` command line tool packaged with PostGIS. Before all of that, you must enable raster support in your database by running the command:

.. code-block:: sql

  CREATE EXTENSION postgis_raster;

We'll start off by first creating raster data from vector data, and then move on to the more exciting approach of loading data from a raster source.
You will find that raster data is available in abundance and often free from various government sites.

We'll start by converting our earlier geometries into rasters using `ST_AsRaster` function as follows.

.. code-block:: sql

  CREATE TABLE rasters (name varchar, rast raster);

  INSERT INTO rasters(name, rast)
  SELECT f.word, ST_AsRaster(geom, width=>150, height=>150)
  FROM (VALUES ('Hello'), ('Raster') ) AS f(word)
    , ST_Letters(word) AS geom;

The above example CREATEs a table (**rasters**) from geometries formed from letters using the PostGIS 3.2+ `ST_Letters <https://postgis.net/docs/ST_Letters.html>`_ function. You can see some useful metadata of your rasters
with the following query which utilizes the postgis raster functions `ST_Count <https://postgis.net/docs/RT_ST_Count.html>`_ function to count the number of pixels and the `ST_MetaData <https://postgis.net/docs/RT_ST_MetaData.html>`_ function to provide all sorts of useful background info for our rasters.

.. code-block:: sql

 SELECT name, ST_Count(rast) As num_pixels, md.*
    FROM rasters, ST_MetaData(rast) AS md;


.. code-block::

  name  | num_pixels | upperleftx |    upperlefty     | width | height |       scalex       |       scaley        | skewx | skewy | srid | numbands
  --------+------------+------------+-------------------+-------+--------+--------------------+---------------------+-------+-------+------+----------
  Hello  |      13926 |          0 | 77.10000000000001 |   150 |    150 |  1.226888888888889 | -0.5173333333333334 |     0 |     0 |    0 |        1
  Raster |      11967 |          0 |              75.4 |   150 |    150 | 1.7226319023207244 | -0.5086666666666667 |     0 |     0 |    0 |        1
  (2 rows)


Note how all the rasters have a 150x150 dimension.  This is not ideal. This means that in order to force that,
our rasters, are squished in all sorts of ways.  If only we could see the ugliness of the rasters before us.

Although pgAdmin and psql have no mechanism yet to view postgis rasters, we have a couple of options. For smallish rasters
the easiest is to output to a web-friendly raster format such as PNG using batteries included postgis raster
functions like `ST_AsPNG` or `ST_AsGDALRaster` listed in `PostGIS Raster output functions <https://postgis.net/docs/RT_reference.html#Raster_Output>`_. As your rasters get larger, you'll want to graduate to a tool
such as QGIS to view them in all their glory or the GDAL family of commandline tools such as gdal_translate to export them to other raster formats.  Remember though, postgis rasters are built for analysis,
not for generating pretty pictures for you to look at.

Run the below query and copy and paste the output into the address bar of your web browser.

.. code-block:: sql

 SELECT 'data:image/png;base64,' ||
    encode(ST_AsPNG(rast),'base64')
    FROM rasters
    WHERE name = 'Hello';

.. image:: ./rasters/hello.png
