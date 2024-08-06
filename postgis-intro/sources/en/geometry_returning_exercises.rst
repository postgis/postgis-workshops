.. _geometry_returning_exercises:

Geometry Constructing Exercises
===============================

Here's a reminder of some of the functions we have seen.  Hint: they should be useful for the exercises!

* :command:`sum(expression)` aggregate to return a sum for a set of records
* :command:`ST_Area(geometry)` returns the area of the geometry
* :command:`ST_Centroid(geometry)` returns the ``geometry`` centroid
* :command:`ST_Transform(geometry, srid)` converts ``geometries`` into different spatial reference systems
* :command:`ST_Buffer(geometry, radius)` returns an expanded ``geometry`` shape
* :command:`ST_Contains(geometry1, geometry2)` returns true if geometry1 contains geometry2
* :command:`ST_Union(geometry[])` returns the aggregate union of all geometries in the group
* :command:`ST_GeometryType(geometry)` returns the type of the geometry
* :command:`ST_NumGeometries(geometry)` returns the number of geometries in a collection or 1 for simple geometries
* :command:`ST_Intersection(geometry, geometry)` returns the area that the two input geometries share in common


Remember the tables we have available:

* ``nyc_census_blocks``

  * name, popn_total, boroname, geom

* ``nyc_streets``

  * name, type, geom

* ``nyc_subway_stations``

  * name, geom

* ``nyc_neighborhoods``

  * name, boroname, geom

Exercises
---------

* **How many census blocks don’t contain their own centroid?**

  .. code-block:: sql

    SELECT Count(*)
      FROM nyc_census_blocks
      WHERE NOT
        ST_Contains(
          geom,
          ST_Centroid(geom)
        );

  ::

    481

* **Union all the census blocks into a single output. What kind of geometry is it? How many parts does it have?**

  .. code-block:: sql

    CREATE TABLE nyc_census_blocks_merge AS
      SELECT ST_Union(geom) AS geom
      FROM nyc_census_blocks;

    SELECT ST_GeometryType(geom)
      FROM nyc_census_blocks_merge;

  ::

    ST_MultiPolygon

  .. code-block:: sql

    SELECT ST_NumGeometries(geom)
      FROM nyc_census_blocks_merge;

  ::

    63


* **What is the area of a one unit buffer around the origin? How different is it from what you would expect? Why?**

  .. code-block:: sql

    SELECT ST_Area(ST_Buffer('POINT(0 0)', 1));

  ::

    3.121445152258052

  .. note::

    A unit circle (circle with radius of one) should have an area of pi, 3.1415926... The difference is due to the linear stroking of the edges of the buffer. The buffer has a finite number of edges. Increasing the number of edges in the buffer will get the value closer to pi, but it will always be smaller due to the linearization.

* **The Brooklyn neighborhoods of ‘Park Slope’ and ‘Carroll Gardens’ are going to war! Construct a polygon delineating a 100 meter wide DMZ on the border between the neighborhoods. What is the area of the DMZ?**

  .. code-block:: sql

    CREATE TABLE brooklyn_dmz AS
      SELECT
        ST_Intersection(
          ST_Buffer(ps.geom, 50),
          ST_Buffer(cg.geom, 50))
        AS geom
      FROM
        nyc_neighborhoods ps,
        nyc_neighborhoods cg
      WHERE ps.name = 'Park Slope'
      AND cg.name = 'Carroll Gardens';

    SELECT ST_Area(geom) FROM brooklyn_dmz;

  .. note::

    It is easy to buffer both the neighborhoods of interest, but to get the intersection requires a self-join of the table, creating one relation (``ps``) with just the "Park Slope" record and another (``cg``) with just the "Carroll Gardens" record. Note that the area of the intersection is in square meters because we are still working in UTM 18 (EPSG:26918).

  ::

    180990.964207547

