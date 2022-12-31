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
  SELECT name, ST_AsRaster(geom, width=>150, height=>150)
  FROM geometries;

The above example CREATEs a table (**rasters**) from geometries created earlier. You can see some useful metadata of your rasters
with the following query:

.. code-block:: sql

 SELECT name, ST_Count(rast) As num_pixels, md.*
    FROM rasters, ST_MetaData(rast) AS md;


.. code-block::

      name       | num_pixels | upperleftx | upperlefty | width | height |        scalex        |        scaley         | skewx | skewy | srid | numbands
  -----------------+------------+------------+------------+-------+--------+----------------------+-----------------------+-------+-------+------+----------
  Point           |          1 |          0 |          0 |   150 |    150 |                    1 |                    -1 |     0 |     0 |    0 |        1
  Linestring      |        149 |          0 |          2 |   150 |    150 | 0.013333333333333334 | -0.013333333333333334 |     0 |     0 |    0 |        1
  Polygon         |      22500 |          0 |          1 |   150 |    150 | 0.006666666666666667 | -0.006666666666666667 |     0 |     0 |    0 |        1
  PolygonWithHole |      22275 |          0 |         10 |   150 |    150 |  0.06666666666666667 |  -0.06666666666666667 |     0 |     0 |    0 |        1
  Collection      |      11250 |          0 |          1 |   150 |    150 | 0.013333333333333334 | -0.006666666666666667 |     0 |     0 |    0 |        1
  (5 rows)


Note how all the rasters have a 150x150 dimension.  This is not ideal. This means that in order to force that,
our rasters, are squished in all sorts of ways.  If only we could see the ugliness of the rasters before us.

Although pgAdmin has no mechanism yet to view postgis rasters, we have a couple of options. The easiest is
to output to a web-friendly raster format such as PNG using batteries included postgis raster
functions like `ST_AsPNG` or `ST_AsGDALRaster`. As your rasters get larger, you'll want to graduate to a tool
such as QGIS to view them in all their glory.  Remember though, postgis rasters are built for analysis,
not for generating big pretty pictures.


.. code-block:: sql

 SELECT 'data:image/jpeg;base64,' ||
    encode(ST_AsJPEG(rast),'base64')
    FROM rasters
    WHERE name = 'PolygonWithHole';

.. image:: ./rasters/polygonwithhole.png
