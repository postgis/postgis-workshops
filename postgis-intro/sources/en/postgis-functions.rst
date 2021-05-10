.. _postgis-functions:

Appendix A: PostGIS Functions
=============================

Constructors
------------

:command:`ST_MakePoint(Longitude, Latitude)` 
  Returns a new point. Note the order of the coordinates (longitude then latitude).

:command:`ST_GeomFromText(WellKnownText, srid)`
  Returns a new geometry from a standard WKT string and srid.

:command:`ST_SetSRID(geometry, srid)`
  Updates the srid on a geometry.  Returns the same geometry.  This does not alter the coordinates of the geometry, it just updates the srid. This function is useful for conditioning geometries created without an srid.

:command:`ST_Expand(geometry, Radius)`
  Returns a new geometry that is an expanded bounding box of the input geometry.  This function is useful for creating envelopes for use in indexed searches.

Outputs
-------

:command:`ST_AsText(geometry)`
  Returns a geometry in a human-readable text format.

:command:`ST_AsGML(geometry)`
  Returns a geometry in standard OGC :term:`GML` format.

:command:`ST_AsGeoJSON(geometry)`
  Returns a geometry to a standard `GeoJSON <http://geojson.org>`_ format.

Measurements
------------

:command:`ST_Area(geometry)`
  Returns the area of the geometry in the units of the spatial reference system.

:command:`ST_Length(geometry)`
  Returns the length of the geometry in the units of the spatial reference system.

:command:`ST_Perimeter(geometry)`
  Returns the perimeter of the geometry in the units of the spatial reference system.

:command:`ST_NumPoints(linestring)`
  Returns the number of vertices in a linestring.

:command:`ST_NumRings(polygon)`
  Returns the number of rings in a polygon.

:command:`ST_NumGeometries(geometry)` 
  Returns the number of geometries in a geometry collection.

Relationships
-------------

:command:`ST_Distance(geometry, geometry)`
  Returns the distance between two geometries in the units of the spatial reference system.

:command:`ST_DWithin(geometry, geometry, radius)` 
  Returns true if the geometries are within the radius distance of one another, otherwise false.

:command:`ST_Intersects(geometry, geometry)`
  Returns true if the geometries are not disjoint, otherwise false.

:command:`ST_Contains(geometry, geometry)`
  Returns true if the first geometry fully contains the second geometry, otherwise false.

:command:`ST_Crosses(geometry, geometry)`
  Returns true if a line or polygon boundary crosses another line or polygon boundary, otherwise false.
