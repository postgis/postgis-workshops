.. _projection_exercises:

Projection Exercises
====================

Here's a reminder of some of the functions we have seen.  Hint: they should be useful for the exercises!

* :command:`sum(expression)` aggregate to return a sum for a set of records
* :command:`ST_Length(linestring)` returns the length of the linestring
* :command:`ST_SRID(geometry)` returns the SRID of the geometry
* :command:`ST_Transform(geometry, srid)` converts geometries into different spatial reference systems
* :command:`ST_GeomFromText(text)` returns ``geometry``
* :command:`ST_AsText(geometry)` returns WKT ``text``
* :command:`ST_AsGML(geometry)` returns GML ``text``

Remember the online resources that are available to you:

* https://epsg.io/

Also remember the tables we have available:

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

* **What is the length of all streets in New York, as measured in UTM 18?**

  .. code-block:: sql

    SELECT Sum(ST_Length(geom))
      FROM nyc_streets;

  ::

    10418904.7172

* **What is the WKT definition of SRID 2831?**

  .. code-block:: sql

    SELECT srtext FROM spatial_ref_sys
    WHERE SRID = 2831;

  Or, via https://epsg.io/2831

  ::

    PROJCS["NAD83(HARN) / New York Long Island",
      GEOGCS["NAD83(HARN)",
        DATUM["NAD83 (High Accuracy Regional Network)",
          SPHEROID["GRS 1980", 6378137.0, 298.257222101,
            AUTHORITY["EPSG","7019"]],
          TOWGS84[-0.991, 1.9072, 0.5129, 0.0257899075194932, -0.009650098960270402, -0.011659943232342112, 0.0],
          AUTHORITY["EPSG","6152"]],
        PRIMEM["Greenwich", 0.0,
          AUTHORITY["EPSG","8901"]],
        UNIT["degree", 0.017453292519943295],
        AXIS["Geodetic longitude", EAST],
        AXIS["Geodetic latitude", NORTH],
        AUTHORITY["EPSG","4152"]],
      PROJECTION["Lambert Conic Conformal (2SP)",
        AUTHORITY["EPSG","9802"]],
      PARAMETER["central_meridian", -74.0],
      PARAMETER["latitude_of_origin", 40.166666666666664],
      PARAMETER["standard_parallel_1", 41.03333333333333],
      PARAMETER["false_easting", 300000.0],
      PARAMETER["false_northing", 0.0],
      PARAMETER["scale_factor", 1.0],
      PARAMETER["standard_parallel_2", 40.666666666666664],
      UNIT["m", 1.0],
      AXIS["Easting", EAST],
      AXIS["Northing", NORTH],
      AUTHORITY["EPSG","2831"]]


* **What is the length of all streets in New York, as measured in SRID 2831?**

  .. code-block:: sql

    SELECT Sum(ST_Length(ST_Transform(geom,2831)))
      FROM nyc_streets;

  ::

    10421993.706374

  .. note::

    The difference between the UTM 18 and the State Plane Long Island measurements is (10421993 - 10418904)/10418904, or 0.02%. Calculated on the spheroid using :ref:`geography` the total street length is 10421999, which is closer to the State Plane value. This is not surprising, since the State Plane Long Island projection is precisely calibrated for a very small area (New York City) while UTM 18 has to provide reasonable results for a large regional area.


* **How many streets cross the 74th meridian?**

  .. code-block:: sql

    SELECT Count(*)
    FROM nyc_streets
    WHERE ST_Intersects(
      ST_Transform(geom, 4326),
      'SRID=4326;LINESTRING(-74 20, -74 60)'
      );

  ::

    223

  The "74th meridian" is a fancy way of saying "a vertical line in geographics where the X value is -74". We can construct such a line and then compare it to the streets, projected into geographics also. Projecting the line into UTM and comparing it there will return a slightly different answer. To get the same answer, you need to "segmentize" it, so it has more points, before transforming.

  .. code-block:: sql

    SELECT Count(*)
    FROM nyc_streets
    WHERE ST_Intersects(
      geom,
      ST_Transform(ST_Segmentize('SRID=4326;LINESTRING(-74 20, -74 60)'::geometry,0.001), 26918)
      );

