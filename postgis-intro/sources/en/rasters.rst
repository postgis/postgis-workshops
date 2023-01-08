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

Creating Rasters From Geometries
--------------------------------
We'll start off by first creating raster data from vector data, and then move on to the more exciting approach of loading data from a raster source.
You will find that raster data is available in abundance and often free from various government sites.

We'll start by converting our earlier geometries into rasters using `ST_AsRaster <https://postgis.net/docs/RT_ST_AsRaster.html>`_ function as follows.

.. code-block:: sql

  CREATE TABLE rasters (name varchar, rast raster);

  INSERT INTO rasters(name, rast)
  SELECT f.word, ST_AsRaster(geom, width=>150, height=>150)
  FROM (VALUES ('Hello'), ('Raster') ) AS f(word)
    , ST_Letters(word) AS geom;

  CREATE INDEX ix_rasters_rast
    ON rasters USING gist(ST_ConvexHull(rast));

The above example CREATEs a table (**rasters**) from geometries formed from letters using the PostGIS 3.2+ `ST_Letters <https://postgis.net/docs/ST_Letters.html>`_ function. Rasters similar to geometries, can take advantage of spatial indexes. The spatial index used for raster
is a functional index that indexes the geometry envelope (or convexhull) of the raster.

You can see some useful metadata of your rasters
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

Viewing Rasters in Browser
------------------------------
Although pgAdmin and psql have no mechanism yet to view postgis rasters, we have a couple of options. For smallish rasters
the easiest is to output to a web-friendly raster format such as PNG using batteries included postgis raster
functions like `ST_AsPNG` or `ST_AsGDALRaster` listed in `PostGIS Raster output functions <https://postgis.net/docs/RT_reference.html#Raster_Outputs>`_.
As your rasters get larger, you'll want to graduate to a tool
such as QGIS to view them in all their glory or the GDAL family of commandline tools such as gdal_translate to export them to other raster formats.  Remember though, postgis rasters are built for analysis,
not for generating pretty pictures for you to look at.

One caveat, by default all different raster types outputs are disabled. In order to utilize these,
you'll need to enable drivers, all or a subset as detailed
in `PostGIS Raster output functions <https://postgis.net/docs/postgis_gdal_enabled_drivers.html>` _

.. code-block:: sql

  SET postgis.gdal_enabled_drivers = 'ENABLE_ALL';

If you don't want to have to do this for each connection, you can set at the database level using:

.. code-block:: sql

  ALTER DATABASE nyc SET postgis.gdal_enabled_drivers = 'ENABLE_ALL';

Each new connection to the database will use that setting.

Run the below query and copy and paste the output into the address bar of your web browser.

.. code-block:: sql

 SET postgis.gdal_drivers
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

Observe the metadata of the New York entries. They have the New York state plane meter spatial reference system.
They also have the same scale.  Since each unit is 1x1 meter,
the width of the word **Raster** is now wider than **Hello**.

.. code-block::

        name         | num_pixels | upperleftx |    upperlefty     | width | height |       scalex       |       scaley        | skewx | skewy | srid  | numbands
  -------------------+------------+------------+-------------------+-------+--------+--------------------+---------------------+-------+-------+-------+----------
  Hello              |      13926 |          0 | 77.10000000000001 |   150 |    150 |  1.226888888888889 | -0.5173333333333334 |     0 |     0 |     0 |        1
  Raster             |      11967 |          0 |              75.4 |   150 |    150 | 1.7226319023207244 | -0.5086666666666667 |     0 |     0 |     0 |        1
  Hello in New York  |       8786 |     586467 |         4504802.1 |   184 |     78 |                  1 |                  -1 |     0 |     0 | 26918 |        3
  Raster in New York |      10544 |     586467 |         4504800.4 |   258 |     76 |                  1 |                  -1 |     0 |     0 | 26918 |        3
  (4 rows)

Exploring Raster Functions
===========================
The postgis_raster extension has over 100 functions to choose from.  We'll focus on the ones you will commonly use.
PostGIS raster functionality was patterned after the PostGIS geometry support.  As such you'll
find an overlap of functions between raster and geometry where it makes sense.
Common ones you'll use are :command:`ST_Intersects`, :command:`ST_Union`, :command:`ST_Intersection`, and :command:`ST_Transform`.
In addition to those overlapping functions, it offers many functions that work in conjunction with geometry
or are very specific to rasters.

Unioning Rasters
--------------------------
The `ST_Union <https://postgis.net/docs/RT_ST_Union.html>`_ function for raster,
just as the geometry equivalent :command:`ST_Union`, aggregates a set of rasters together
into a single raster.  However, just as with geometry,
not all rasters can be combined together,
but the rules for raster unioning are more complicated than geometry rules.
In the case of geometries, all you need is to have the same spatial reference system,
but for rasters that is not sufficient.

If you were to attempt, the following

.. code-block:: sql

 SELECT ST_Union(rast)
    FROM rasters;

You'd be summarily punished with an error:

**ERROR:  rt_raster_from_two_rasters: The two rasters provided do not have the same alignment
SQL state: XX000**

What is this same alignment thing, that is preventing you from unioning your precious rasters?

In order for rasters to be combined, they need to be on the same grid so to speak. Meaning
they must have same pixel sizes, same orientation (the skew), same spatial reference system,
and their pixels must not cut into each other, meaning they share the same worldly pixel grid.

If you try the same query, but just with words we carefully placed in New York.

Again, the same error. These are the same spatial ref system, the same pixel sizes,
and yet it's still not good enough.
Because their grids are off.

We can fix this by shifting the upper left y coordinates ever so slightly and then trying again.
If our grids start at integer level since our pixel sizes are whole integer,
then the pixels won't cut into each other.

.. code-block:: sql

  UPDATE rasters SET rast = ST_SetUpperLeft(rast,
    ST_UpperLeftX(rast)::integer,
    ST_UpperLeftY(rast)::integer)
  WHERE name LIKE '%New York';

  SELECT ST_Union(rast)
    FROM rasters
    WHERE name LIKE '%New York%';

Voila it worked, and if we were to view, we'd see something like this:

.. image:: ./rasters/hello-raster-ny.png

If ever you are unclear why your rasters don't have the same alignment, you can use the function
`ST_SameAlignment <https://postgis.net/docs/RT_ST_SameAlignment.html>`_, which will compare 2 rasters
or a set of rasters and tell you if they have the same alignment.  If you have notices enabled, the
NOTICE will tell you what is off with the rasters in question. The
`ST_NotSameAlignmentReason <https://postgis.net/docs/RT_ST_NotSameAlignmentReason.html>`_, instead of just a notice
will output the reason. It however only works with two rasters at a time.

One major way in which the `ST_Union <https://postgis.net/docs/RT_ST_Union.html>`_ raster function deviates
from the `ST_Union <https://postgis.net/docs/ST_Union.html>`_ geometry function is that
it allows for an argument called *uniontype*.  This argument by default is set to `LAST` if you don't specify it,
which means, take the **LAST** raster pixel values in occasions where the rasters overlap.

But on occassion, **LAST** may not be the right operation.
Let's suppose our rasters represented two different sets of
observations from two different devices. These devices measure the same,
thing, and we aren't sure which is right when they cross paths,
so we'd instead like to take the `MEAN` of the results.  We'd do this:

.. code-block:: sql

  SELECT ST_Union(rast, 'MEAN')
    FROM rasters
    WHERE name LIKE '%New York%';

Voila it worked, and if we were to view, we'd see something like this:

.. image:: ./rasters/hello-raster-ny-mean.png

So instead of **Raster** trumping **Hello**, we'd see a blending of the two forces.
Note that for geometries
since geometries are vector and thus have no values besides there or not there,
there really isn't any ambiguity on how to combine two vectors when they intersect.

Another feature of the raster :command:`ST_Union` we glossed over,
is this idea of if you should return all bands or just some bands.
When you don't specify what bands to union, :command:`ST_Union` will combine
same banded numbers and use the `LAST` unioning
strategy.  If you have multiple bands, this may not be what you want to do.
Perhaps you only want to union, the second band.
In this case, the Green Band and you want the count of pixel values.

.. code-block:: sql

  SELECT ST_BandPixelType(ST_Union(rast, 2, 'COUNT'))
    FROM rasters
    WHERE name LIKE '%New York%';

Note in the case of the **COUNT** union type, which counts the number of pixels filled in and returns that value,
The result is always a **32BUI** similar to how when you do a :command:`COUNT` in sql, the result is always a bigint,
to accommodate large counts.

In other cases, the band pixel type does not change and is set to the max value or rounded
if the amounts exceed the bounds of the type.
Why would anyone ever want to count pixels that intersect at a location.
Well suppose each of your rasters
represent police squadrons and incidents of arrests in the areas.
Each value, might represent a different kind
of arrest reason. You are doing stats on how many arrests in each region,
therefore you only care about the count of arrests.

Or perhaps, you want to do all bands, but you want different strategies.

.. code-block:: sql

  SELECT ST_Union(rast, ARRAY[(1, 'MAX'),
    (2, 'MEAN'),
    (3, 'RANGE')]::unionarg[])
    FROM rasters
    WHERE name LIKE '%New York%';

Using the *unionarg[]* variant of the :command:`ST_Union` function, also allows you to shuffle the order of the bands.

Clipping Rasters
-----------------
The `ST_Clip <https://postgis.net/docs/RT_ST_Clip.html>`_ function is one of the most widely used functions
for PostGIS rasters.  The main reason is the more pixels you need to inspect or do operations on, the slower your processing.
**ST_Clip** clips your raster to just the area of interest, so you can isolate your operations to just that area.

This function is also special in that it utilizes the power of geometry to help raster analysis.
To reduce the number of pixels, :command:`ST_Union` has to handle, each raster is clipped first to the area we are interested in.

.. code-block:: sql

  SELECT ST_Union( ST_Clip(r.rast, g.geom) )
    FROM rasters AS r
        INNER JOIN
          ST_Buffer(ST_Point(586598, 4504816, 26918), 100 ) AS g(geom)
            ON ST_Intersects(r.rast, g.geom)
    WHERE r.name LIKE '%New York%';

This example showcases several functions working in unison.  The :command:`ST_Intersects` function employed
is the one packaged with **postgis_raster** and can intersect 2 rasters or a raster and a geometry.
Similar to the geometry :command:`ST_Intersects` the raster :command:`ST_Intersects`
can take advantage of spatial indexes on the raster or geometry tables.
