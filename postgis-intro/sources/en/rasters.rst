.. _raster:

Raster
=========

PostGIS supports another kind of spatial data type called a *raster*.
Raster data, much like geometry data, uses **Cartesian coordinates** and a spatial reference system.
However instead of vector data, raster data is represented as an n-dimensional matrix consisting of pixels and bands.
The bands defines the number of matrices you have. Each pixel stores a value corresponding to each band.
So a 3-banded raster such as an RGB image, would have 3 values for each pixel corresponding to the Red-Green-Blue bands.

Although pretty pictures such as those you see on your TV screen are rasters, rasters may not be that exciting to look at.
In a nutshell, a raster is a matrix, pinned on a coordinate system, that has values that can represent anything you want them to represent.

Since rasters leave in cartesian space, rasters can interact with geometries.  PostGIS offers many functions that take as input both rasters and geometries.
Also many operations applied to rasters will result in geometries. Common ones are the `ST_Polygon`, `ST_Envelope`, `ST_ConvexHull`, and `ST_MinConvexHull`
as shown below.

.. image:: ./rasters/postgis_raster.jpg

The raster format is commonly used to store elevation data, temperature data, satellite data, and thematic data representing things like environmental contamination, population density, and environmental hazard occurrences.

Although raster data can be created from scratch in PostGIS,  a more common approch is to load raster data from various formats using the `raster2pgsql` command line tool packaged with PostGIS. Before all of that, you must enable raster support in your database by running the command:

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
