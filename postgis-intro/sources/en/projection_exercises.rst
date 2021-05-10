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

* http://spatialreference.org
* http://prj2epsg.org

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

* **"What is the length of all streets in New York, as measured in UTM 18?"**
 
  .. code-block:: sql

    SELECT Sum(ST_Length(geom))
      FROM nyc_streets;

  :: 
  
    10418904.7172
      
* **"What is the WKT definition of SRID 2831?"**   
    
  .. code-block:: sql

    SELECT srtext FROM spatial_ref_sys
    WHERE SRID = 2831;

  Or, via `prj2epsg <http://prj2epsg.org/epsg/2831>`_

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
  

* **"What is the length of all streets in New York, as measured in SRID 2831?"**
 
  .. code-block:: sql

    SELECT Sum(ST_Length(ST_Transform(geom,2831)))
      FROM nyc_streets;

  :: 
   
    10421993.706374
     
  .. note::
   
    The difference between the UTM 18 and the State Plane Long Island measurements is (10421993 - 10418904)/10418904, or 0.02%. Calculated on the spheroid using :ref:`geography` the total street length is 10421999, which is closer to the State Plane value. This is not surprising, since the State Plane Long Island projection is precisely calibrated for a very small area (New York City) while UTM 18 has to provide reasonable results for a large regional area.
     
* **"What is the KML representation of the point at 'Broad St' subway station?"**
 
  .. code-block:: sql
   
    SELECT ST_AsKML(geom) 
    FROM nyc_subway_stations
    WHERE name = 'Broad St';
     
  :: 
   
    <Point>
      <coordinates>
        -74.010671468873468,40.707104815584088
      </coordinates>
    </Point>
     
  Hey! The coordinates are in geographics even though we didn't call :command:`ST_Transform`, why? Because the KML standard dictates that all coordinates *must* be in geographics (ESPG:4326, in fact) so the :command:`ST_AsKML` function does the transformation automatically.
