.. _geometries:

Geometries
==========

Introduction
------------

In the previous :ref:`section <loading_data>`, we loaded a variety of data.  Before we start playing with our data lets have a look at some simpler examples.  In pgAdmin, once again select the **nyc** database and open the SQL query tool.  Paste this example SQL code into the pgAdmin SQL Editor window (removing any text that may be there by default) and then execute.

.. code-block:: sql

  CREATE TABLE geometries (name varchar, geom geometry);
  
  INSERT INTO geometries VALUES 
    ('Point', 'POINT(0 0)'),
    ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
    ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
    ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');
    
  SELECT name, ST_AsText(geom) FROM geometries;

.. image:: ./geometries/start01.png

The above example CREATEs a table (**geometries**) then INSERTs five geometries: a point, a line, a polygon, a polygon with a hole, and a collection. Finally, the inserted rows are SELECTed and displayed in the Output pane.

Metadata Tables
---------------

In conformance with the Simple Features for SQL (:term:`SFSQL`) specification, PostGIS provides two tables to track and report on the geometry types available in a given database. 

* The first table, ``spatial_ref_sys``, defines all the spatial reference systems known to the database and will be described in greater detail later.  
* The second table (actually, a view), ``geometry_columns``, provides a listing of all "features" (defined as an object with geometric attributes), and the basic details of those features.  

.. image:: ./geometries/table01.png
  :class: inline

Let's have a look at the ``geometry_columns`` table in our database.  Paste this command in the Query Tool as before:

.. code-block:: sql

  SELECT * FROM geometry_columns;

.. image:: ./geometries/start08.png

* ``f_table_catalog``, ``f_table_schema``, and ``f_table_name`` provide the fully qualified name of the feature table containing a given geometry.  Because PostgreSQL doesn't make use of catalogs, ``f_table_catalog`` will tend to be empty.  
* ``f_geometry_column`` is the name of the column that geometry containing column -- for feature tables with multiple geometry columns, there will be one record for each.  
* ``coord_dimension`` and ``srid`` define the the dimension of the geometry (2-, 3- or 4-dimensional) and the Spatial Reference system identifier that refers to the ``spatial_ref_sys`` table respectively.  
* The ``type`` column defines the type of geometry as described below; we've seen Point and Linestring types so far.  

By querying this table, GIS clients and libraries can determine what to expect when retrieving data and can perform any necessary projection, processing or rendering without needing to inspect each geometry.

.. note::

   Do some or all of your ``nyc`` tables not have an ``srid`` of 26918? It's easy to fix by updating the table::

   .. code-block:: sql
  
      SELECT UpdateGeometrySRID('nyc_neighborhoods','geom',26918);

Representing Real World Objects
-------------------------------

The Simple Features for SQL (:term:`SFSQL`) specification, the original guiding standard for PostGIS development, defines how a real world object is represented.  By taking a continuous shape and digitizing it at a fixed resolution we achieve a passable representation of the object.  SFSQL only handled 2-dimensional representations.  PostGIS has extended that to include 3- and 4-dimensional representations; more recently the SQL-Multimedia Part 3 (:term:`SQL/MM`) specification has officially defined their own representation.  

Our example table contains a mixture of different geometry types. We can collect general information about each object using functions that read the geometry metadata.

* :command:`ST_GeometryType(geometry)` returns the type of the geometry
* :command:`ST_NDims(geometry)` returns the number of dimensions of the geometry
* :command:`ST_SRID(geometry)` returns the spatial reference identifier number of the geometry

.. code-block:: sql

  SELECT name, ST_GeometryType(geom), ST_NDims(geom), ST_SRID(geom)
    FROM geometries;

::

       name       |    st_geometrytype    | st_ndims | st_srid 
 -----------------+-----------------------+----------+---------
  Point           | ST_Point              |        2 |       0
  Polygon         | ST_Polygon            |        2 |       0
  PolygonWithHole | ST_Polygon            |        2 |       0
  Collection      | ST_GeometryCollection |        2 |       0
  Linestring      | ST_LineString         |        2 |       0


Points
~~~~~~

.. image:: ./introduction/points.png
  :align: center
  :class: inline

A spatial **point** represents a single location on the Earth.  This point is represented by a single coordinate (including either 2-, 3- or 4-dimensions).  Points are used to represent objects when the exact details, such as shape and size, are not important at the target scale.  For example, cities on a map of the world can be described as points, while a map of a single state might represent cities as polygons.  

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Point';

::

  POINT(0 0)

Some of the specific spatial functions for working with points are:

* :command:`ST_X(geometry)` returns the X ordinate
* :command:`ST_Y(geometry)` returns the Y ordinate

So, we can read the ordinates from a point like this:

.. code-block:: sql

  SELECT ST_X(geom), ST_Y(geom)
    FROM geometries
    WHERE name = 'Point';

The New York City subway stations (``nyc_subway_stations``) table is a data set represented as points. The following SQL query will return the geometry associated with one point (in the :command:`ST_AsText` column).

.. code-block:: sql

  SELECT name, ST_AsText(geom)
    FROM nyc_subway_stations
    LIMIT 1;


Linestrings
~~~~~~~~~~~

.. image:: ./introduction/lines.png
  :align: center
  :class: inline

A **linestring** is a path between locations.  It takes the form of an ordered series of two or more points.  Roads and rivers are typically represented as linestrings.  A linestring is said to be **closed** if it starts and ends on the same point.  It is said to be **simple** if it does not cross or touch itself (except at its endpoints if it is closed).  A linestring can be both **closed** and **simple**.

The street network for New York (``nyc_streets``) was loaded earlier in the workshop.  This dataset contains details such as name, and type.  A single real world street may consist of many linestrings, each representing a segment of road with different attributes.

The following SQL query will return the geometry associated with one linestring (in the :command:`ST_AsText` column).

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Linestring';
  
::

  LINESTRING(0 0, 1 1, 2 1, 2 2)

Some of the specific spatial functions for working with linestrings are:

* :command:`ST_Length(geometry)` returns the length of the linestring
* :command:`ST_StartPoint(geometry)` returns the first coordinate as a point
* :command:`ST_EndPoint(geometry)` returns the last coordinate as a point
* :command:`ST_NPoints(geometry)` returns the number of coordinates in the linestring

So, the length of our linestring is:

.. code-block:: sql

  SELECT ST_Length(geom) 
    FROM geometries
    WHERE name = 'Linestring';

::

  3.41421356237309


Polygons
~~~~~~~~

.. image:: ./introduction/polygons.png
  :align: center
  :class: inline

A polygon is a representation of an area.  The outer boundary of the polygon is represented by a ring.  This ring is a linestring that is both closed and simple as defined above.  Holes within the polygon are also represented by rings.

Polygons are used to represent objects whose size and shape are important.  City limits, parks, building footprints or bodies of water are all commonly represented as polygons when the scale is sufficiently high to see their area.  Roads and rivers can sometimes be represented as polygons.

The following SQL query will return the geometry associated with one linestring (in the :command:`ST_AsText` column).

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name LIKE 'Polygon%';

.. note::

   Rather than using an ``=`` sign in our ``WHERE`` clause, we are using the ``LIKE`` operator to carry out a string matching operation. **You may be used to the ``*`` symbol as a "glob" for pattern matching, but in SQL the ``%`` symbol is used**, along with the ``LIKE`` operator to tell the system to do globbing.

::

 POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))
 POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))

The first polygon has only one ring. The second one has an interior "hole". Most graphics systems include the concept of a "polygon", but GIS systems are relatively unique in allowing polygons to explicitly have holes.

.. image:: ./screenshots/polygons.png

Some of the specific spatial functions for working with polygons are:

* :command:`ST_Area(geometry)` returns the area of the polygons
* :command:`ST_NRings(geometry)` returns the number of rings (usually 1, more of there are holes)
* :command:`ST_ExteriorRing(geometry)` returns the outer ring as a linestring
* :command:`ST_InteriorRingN(geometry,n)` returns a specified interior ring as a linestring
* :command:`ST_Perimeter(geometry)` returns the length of all the rings

We can calculate the area of our polygons using the area function:

.. code-block:: sql

  SELECT name, ST_Area(geom) 
    FROM geometries
    WHERE name LIKE 'Polygon%';

::

  Polygon            1
  PolygonWithHole    99

Note that the polygon with a hole has an area that is the area of the outer shell (a 10x10 square) minus the area of the hole (a 1x1 square).

Collections
~~~~~~~~~~~

There are four collection types, which group multiple simple geometries into sets.  

* **MultiPoint**, a collection of points
* **MultiLineString**, a collection of linestrings
* **MultiPolygon**, a collection of polygons
* **GeometryCollection**, a heterogeneous collection of any geometry (including other collections)

Collections are another concept that shows up in GIS software more than in generic graphics software. They are useful for directly modeling real world objects as spatial objects. For example, how to model a lot that is split by a right-of-way? As a **MultiPolygon**, with a part on either side of the right-of-way.

.. image:: ./screenshots/collection2.png

Our example collection contains a polygon and a point:

.. code-block:: sql

  SELECT name, ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Collection';

::

  GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))

.. image:: ./screenshots/collection.png

Some of the specific spatial functions for working with collections are:

* :command:`ST_NumGeometries(geometry)` returns the number of parts in the collection
* :command:`ST_GeometryN(geometry,n)` returns the specified part
* :command:`ST_Area(geometry)` returns the total area of all polygonal parts
* :command:`ST_Length(geometry)` returns the total length of all linear parts



Geometry Input and Output
-------------------------

Within the database, geometries are stored on disk in a format only used by the PostGIS program. In order for external programs to insert and retrieve useful geometries, they need to be converted into a format that other applications can understand. Fortunately, PostGIS supports emitting and consuming geometries in a large number of formats:

* Well-known text (:term:`WKT`)
 
  * :command:`ST_GeomFromText(text, srid)` returns ``geometry``
  * :command:`ST_AsText(geometry)` returns ``text``
  * :command:`ST_AsEWKT(geometry)` returns ``text``
   
* Well-known binary (:term:`WKB`)
 
  * :command:`ST_GeomFromWKB(bytea)` returns ``geometry``
  * :command:`ST_AsBinary(geometry)` returns ``bytea``
  * :command:`ST_AsEWKB(geometry)` returns ``bytea``
   
* Geographic Mark-up Language (:term:`GML`)
 
  * :command:`ST_GeomFromGML(text)` returns ``geometry``
  * :command:`ST_AsGML(geometry)` returns ``text``
   
* Keyhole Mark-up Language (:term:`KML`)
 
  * :command:`ST_GeomFromKML(text)` returns ``geometry``
  * :command:`ST_AsKML(geometry)` returns ``text``
   
* :term:`GeoJSON`
 
  * :command:`ST_AsGeoJSON(geometry)` returns ``text``
   
* Scalable Vector Graphics (:term:`SVG`)
 
  * :command:`ST_AsSVG(geometry)` returns ``text``
 
The most common use of a constructor is to turn a text representation of a geometry into an internal representation:

.. code-block::sql

  SELECT ST_GeomFromText('POINT(583571 4506714)',26918);
 
Note that in addition to a text parameter with a geometry representation, we also have a numeric parameter providing the :term:`SRID` of the geometry.
 
The following SQL query shows an example of :term:`WKB` representation (the call to :command:`encode()` is required to convert the binary output into an ASCII form for printing):

.. code-block:: sql

  SELECT encode(
    ST_AsBinary(ST_GeometryFromText('LINESTRING(0 0,1 0)')), 
    'hex');

::

  01020000000200000000000000000000000000000000000000000000000000f03f0000000000000000
  
For the purposes of this workshop we will continue to use WKT to ensure you can read and understand the geometries we're viewing.  However, most actual processes, such as viewing data in a GIS application, transferring data to a web service, or processing data remotely, WKB is the format of choice.  

Since WKT and WKB were defined in the  :term:`SFSQL` specification, they do not handle 3- or 4-dimensional geometries.  For these cases PostGIS has defined the Extended Well Known Text (EWKT) and Extended Well Known Binary (EWKB) formats.  These provide the same formatting capabilities of WKT and WKB with the added dimensionality.

Here is an example of a 3D linestring in WKT:

.. code-block:: sql

  SELECT ST_AsText(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)'));

::

  LINESTRING Z (0 0 0,1 0 0,1 1 2)

Note that the text representation changes! This is because the text input routine for PostGIS is liberal in what it consumes. It will consume 

* hex-encoded EWKB, 
* extended well-known text, and 
* ISO standard well-known text.

On the output side, the :command:`ST_AsText` function is conservative, and only emits ISO standard well-known text.

In addition to the :command:`ST_GeometryFromText` function, there are many other ways to create geometries from well-known text or similar formatted inputs:

.. code-block:: sql

  -- Using ST_GeomFromText with the SRID parameter
  SELECT ST_GeomFromText('POINT(2 2)',4326);

  -- Using ST_GeomFromText without the SRID parameter
  SELECT ST_SetSRID(ST_GeomFromText('POINT(2 2)'),4326);
  
  -- Using a ST_Make* function
  SELECT ST_SetSRID(ST_MakePoint(2, 2), 4326);
  
  -- Using PostgreSQL casting syntax and ISO WKT
  SELECT ST_SetSRID('POINT(2 2)'::geometry, 4326);
  
  -- Using PostgreSQL casting syntax and extended WKT
  SELECT 'SRID=4326;POINT(2 2)'::geometry;

  
In addition to emitters for the various forms (WKT, WKB, GML, KML, JSON, SVG), PostGIS also has consumers for four (WKT, WKB, GML, KML). Most applications use the WKT or WKB geometry creation functions, but the others work too. Here's an example that consumes GML and output JSON:

.. code-block:: sql

  SELECT ST_AsGeoJSON(ST_GeomFromGML('<gml:Point><gml:coordinates>1,1</gml:coordinates></gml:Point>'));

.. image:: ./geometries/represent-07.png


Casting from Text
-----------------

The :term:`WKT` strings we've see so far have been of type 'text' and we have been converting them to type 'geometry' using PostGIS functions like :command:`ST_GeomFromText()`. 

PostgreSQL includes a short form syntax that allows data to be converted from one type to another, the casting syntax, `oldata::newtype`. So for example, this SQL converts a double into a text string.

.. code-block:: sql

  SELECT 0.9::text;

Less trivially, this SQL converts a :term:`WKT` string into a geometry:

.. code-block:: sql

  SELECT 'POINT(0 0)'::geometry;

One thing to note about using casting to create geometries: unless you specify the SRID, you will get a geometry with an unknown SRID. You can specify the SRID using the "extended" well-known text form, which includes an SRID block at the front:

.. code-block:: sql

  SELECT 'SRID=4326;POINT(0 0)'::geometry;

It's very common to use the casting notation when working with :term:`WKT`, as well as `geometry` and `geography` columns (see :ref:`geography`).


Function List
-------------

`ST_Area <http://postgis.net/docs/ST_Area.html>`_: Returns the area of the surface if it is a polygon or multi-polygon. For "geometry" type area is in SRID units. For "geography" area is in square meters.

`ST_AsText <http://postgis.net/docs/ST_AsText.html>`_: Returns the Well-Known Text (WKT) representation of the geometry/geography without SRID metadata.

`ST_AsBinary <http://postgis.net/docs/ST_AsBinary.html>`_: Returns the Well-Known Binary (WKB) representation of the geometry/geography without SRID meta data.

`ST_EndPoint <http://postgis.net/docs/ST_EndPoint.html>`_: Returns the last point of a LINESTRING geometry as a POINT.

`ST_AsEWKB <http://postgis.net/docs/ST_AsEWKB.html>`_: Returns the Well-Known Binary (WKB) representation of the geometry with SRID meta data.

`ST_AsEWKT <http://postgis.net/docs/ST_AsEWKT.html>`_: Returns the Well-Known Text (WKT) representation of the geometry with SRID meta data.

`ST_AsGeoJSON <http://postgis.net/docs/ST_AsGeoJSON.html>`_: Returns the geometry as a GeoJSON element.

`ST_AsGML <http://postgis.net/docs/ST_AsGML.html>`_: Returns the geometry as a GML version 2 or 3 element.

`ST_AsKML <http://postgis.net/docs/ST_AsKML.html>`_: Returns the geometry as a KML element. Several variants. Default version=2, default precision=15.

`ST_AsSVG <http://postgis.net/docs/ST_AsSVG.html>`_: Returns a Geometry in SVG path data given a geometry or geography object.

`ST_ExteriorRing <http://postgis.net/docs/ST_ExteriorRing.html>`_: Returns a line string representing the exterior ring of the POLYGON geometry. Return NULL if the geometry is not a polygon. Will not work with MULTIPOLYGON

`ST_GeometryN <http://postgis.net/docs/ST_GeometryN.html>`_: Returns the 1-based Nth geometry if the geometry is a GEOMETRYCOLLECTION, MULTIPOINT, MULTILINESTRING, MULTICURVE or MULTIPOLYGON. Otherwise, return NULL.

`ST_GeomFromGML <http://postgis.net/docs/ST_GeomFromGML.html>`_: Takes as input GML representation of geometry and outputs a PostGIS geometry object.

`ST_GeomFromKML <http://postgis.net/docs/ST_GeomFromKML.html>`_: Takes as input KML representation of geometry and outputs a PostGIS geometry object

`ST_GeomFromText <http://postgis.net/docs/ST_GeomFromText.html>`_: Returns a specified ST_Geometry value from Well-Known Text representation (WKT).

`ST_GeomFromWKB <http://postgis.net/docs/ST_GeomFromWKB.html>`_: Creates a geometry instance from a Well-Known Binary geometry representation (WKB) and optional SRID.

`ST_GeometryType <http://postgis.net/docs/ST_GeometryType.html>`_: Returns the geometry type of the ST_Geometry value.

`ST_InteriorRingN <http://postgis.net/docs/ST_InteriorRingN.html>`_: Returns the Nth interior linestring ring of the polygon geometry. Return NULL if the geometry is not a polygon or the given N is out of range.

`ST_Length <http://postgis.net/docs/ST_Length.html>`_: Returns the 2d length of the geometry if it is a linestring or multilinestring. geometry are in units of spatial reference and geography are in meters (default spheroid)

`ST_NDims <http://postgis.net/docs/ST_NDims.html>`_: Returns coordinate dimension of the geometry as a small int. Values are: 2,3 or 4.

`ST_NPoints <http://postgis.net/docs/ST_NPoints.html>`_: Returns the number of points (vertexes) in a geometry.

`ST_NRings <http://postgis.net/docs/ST_NRings.html>`_: If the geometry is a polygon or multi-polygon returns the number of rings.

`ST_NumGeometries <http://postgis.net/docs/ST_NumGeometries.html>`_: If geometry is a GEOMETRYCOLLECTION (or MULTI*) returns the number of geometries, otherwise return NULL.

`ST_Perimeter <http://postgis.net/docs/ST_Perimeter.html>`_: Returns the length measurement of the boundary of an ST_Surface or ST_MultiSurface value. (Polygon, Multipolygon)

`ST_SRID <http://postgis.net/docs/ST_SRID.html>`_: Returns the spatial reference identifier for the ST_Geometry as defined in spatial_ref_sys table.

`ST_StartPoint <http://postgis.net/docs/ST_StartPoint.html>`_: Returns the first point of a LINESTRING geometry as a POINT.

`ST_X <http://postgis.net/docs/ST_X.html>`_: Returns the X coordinate of the point, or NULL if not available. Input must be a point.

`ST_Y <http://postgis.net/docs/ST_Y.html>`_: Returns the Y coordinate of the point, or NULL if not available. Input must be a point.


