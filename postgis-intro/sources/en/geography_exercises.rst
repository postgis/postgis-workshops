.. _geography_exercises:

Geography Exercises
===================

Here's a reminder of all the functions we have seen so far. They should be useful for the exercises!

* :command:`Sum(number)` adds up all the numbers in the result set
* :command:`ST_GeogFromText(text)` returns a geography
* :command:`ST_Distance(geography, geography)` returns the distance between geographies
* :command:`ST_Transform(geometry, srid)` returns geometry, in the new projection
* :command:`ST_Length(geography)` returns the length of the line
* :command:`ST_Intersects(geometry, geometry)` returns true if the objects are not disjoint in planar space
* :command:`ST_Intersects(geography, geography)` returns true if the objects are not disjoint in spheroidal space

Also remember the tables we have available:

* ``nyc_streets``
 
  * name, type, geom
   
* ``nyc_neighborhoods``
 
  * name, boroname, geom


Exercises
---------

* **"What is the total length of all streets in New York, calculated on the spheroid?"**
 
  .. code-block:: sql

    SELECT Sum(
      ST_Length(Geography(
        ST_Transform(geom,4326)
      )))
    FROM nyc_streets;

  :: 

    10421999.666

  .. note::

    The length calculated in the planar "UTM Zone 18" projection is 10418904.717, 0.02% different. UTM is good at preserving area and distance, within the zone boundaries.

* **"Does ‘POINT(1 2.0001)’ intersect with ‘POLYGON((0 0, 0 2, 2 2, 2 0, 0 0))’ in geography? In geometry? Why the difference?"**
 
  .. code-block:: sql

    SELECT ST_Intersects(
      'POINT(1 2.0001)'::geography,
      'POLYGON((0 0,0 2,2 2,2 0,0 0))'::geography
    );

    SELECT ST_Intersects(
      'POINT(1 2.0001)'::geometry,
      'POLYGON((0 0,0 2,2 2,2 0,0 0))'::geometry
    );

  :: 
   
    true and false

  .. note::

    The upper edge of the square is a straight line in geometry, and passes **below** the point, so the square does not contain the point. The upper edge of the square is a great circle in geography, and passes **above** the point, so the square does contain the point.

