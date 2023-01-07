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

Although raster data can be created from scratch in PostGIS, a more common approach is to load raster data from various formats using the `raster2pgsql` command line tool packaged with PostGIS. Before all of that, you must enable raster support in your database by running the command:

.. code-block:: sql

  CREATE EXTENSION postgis_raster;

We'll start off by first creating raster data from vector data, and then move on to the more exciting approach of loading data from a raster source.
You will find that raster data is available in abundance and often free from various government sites.

We'll start by converting our earlier geometries into rasters using `ST_AsRaster <https://postgis.net/docs/RT_ST_AsRaster.html>`_ function as follows.

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
functions like `ST_AsPNG` or `ST_AsGDALRaster` listed in `PostGIS Raster output functions <https://postgis.net/docs/RT_reference.html#Raster_Outputs>`_. As your rasters get larger, you'll want to graduate to a tool
such as QGIS to view them in all their glory or the GDAL family of commandline tools such as gdal_translate to export them to other raster formats.  Remember though, postgis rasters are built for analysis,
not for generating pretty pictures for you to look at.

Run the below query and copy and paste the output into the address bar of your web browser.

.. code-block:: sql

 SELECT 'data:image/png;base64,' ||
    encode(ST_AsPNG(rast),'base64')
    FROM rasters
    WHERE name = 'Hello';

.. image:: ./rasters/hello.png

For the rasters created thus far, we didn't specify the number of bands nor did we even
define their relation to earth.  As such our rasters have an unknown spatial reference system (0).

You can think of a rasters exoskeletal as a geometry.
A matrix encased in a geometric envelop. In order to do useful analysis,
we need to georeference our rasters,
meaning we want each pixel (rectangle) to represent some meaningful plot of space.

The `ST_AsRaster` has many overloaded representations. The earlier example used the simplest such implementation
and accepted the default arguments which are 8BUI and 1 band, with no data being 0.
If you need to use the other variants, you should use the named arguments call syntax so that you don't accidentally
fall into the wrong variant of the function or get *function is not unique* errors.


If you start with a geometry that has a spatial reference system, you'll end up with a raster
with same spatial reference system.  In this next example, we'll plop our words in New York in
bright cheery colors. We will also use pixel scale instead of width and height so that
our raster pixel sizes represent 1 meter x 1 meter of space.

.. code-block:: sql

  INSERT INTO rasters(name, rast)
  SELECT f.word || ' in New York' ,
    ST_AsRaster(geom,
      scalex => 1.0, scaley => -1.0,
      pixeltype => ARRAY['8BUI', '8BUI', '8BUI'],
      value => ARRAY[ (random()*255)::integer,
         (random()*255)::integer, (random()*255)::integer ],
      nodataval => ARRAY[0,0,0], gridx => NULL, gridy => NULL
      ) AS rast
  FROM (
      VALUES ('Hello'), ('Raster') ) AS f(word)
    , ST_SetSRID(
        ST_Translate(ST_Letters(word),586467,4504725), 26918
      ) AS geom;

If we then look at this, we'll see a non-squashed colored geometry.
Your color may be different from the below since we used a random generate to choose the colors.

.. code-block:: sql

 SELECT 'data:image/png;base64,' ||
    encode(ST_AsPNG(rast),'base64')
    FROM rasters
    WHERE name = 'Hello in New York';

.. image:: ./rasters/hello-ny.png

What is more telling, if we rerun the

.. code-block:: sql

  SELECT name, ST_Count(rast) As num_pixels, md.*
    FROM rasters, ST_MetaData(rast) AS md;

.. code-block::

        name        | num_pixels | upperleftx |    upperlefty     | width | height |       scalex       |       scaley        | skewx | skewy | srid  | numbands
  --------------------+------------+------------+-------------------+-------+--------+--------------------+---------------------+-------+-------+-------+----------
  Hello              |      13926 |          0 | 77.10000000000001 |   150 |    150 |  1.226888888888889 | -0.5173333333333334 |     0 |     0 |     0 |        1
  Raster             |      11967 |          0 |              75.4 |   150 |    150 | 1.7226319023207244 | -0.5086666666666667 |     0 |     0 |     0 |        1
  Hello in New York  |       8786 |     586467 |         4504802.1 |   184 |     78 |                  1 |                  -1 |     0 |     0 | 26918 |        3
  Raster in New York |      10544 |     586467 |         4504800.4 |   258 |     76 |                  1 |                  -1 |     0 |     0 | 26918 |        3
  (4 rows)

Observe the metadata of the New York entries. They have the New York state plane meter spatial reference system.
They also have the same scale.  Since each unit is 1x1 meter, the width of the work 'Raster' is now wider than 'Hello'.
