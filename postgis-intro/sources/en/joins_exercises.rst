.. _joins_exercises:

Spatial Joins Exercises
=======================

Here's a reminder of some of the functions we have seen.  Hint: they should be useful for the exercises!

* :command:`sum(expression)`: aggregate to return a sum for a set of records
* :command:`count(expression)`: aggregate to return the size of a set of records
* :command:`ST_Area(geometry)` returns the area of the polygons
* :command:`ST_AsText(geometry)` returns WKT ``text``
* :command:`ST_Contains(geometry A, geometry B)` returns the true if geometry A contains geometry B 
* :command:`ST_Distance(geometry A, geometry B)` returns the minimum distance between geometry A and geometry B
* :command:`ST_DWithin(geometry A, geometry B, radius)` returns the true if geometry A is radius distance or less from geometry B
* :command:`ST_GeomFromText(text)` returns ``geometry``
* :command:`ST_Intersects(geometry A, geometry B)` returns the true if geometry A intersects geometry B
* :command:`ST_Length(linestring)` returns the length of the linestring
* :command:`ST_Touches(geometry A, geometry B)` returns the true if the boundary of geometry A touches geometry B
* :command:`ST_Within(geometry A, geometry B)` returns the true if geometry A is within geometry B
 
Also remember the tables we have available:

* ``nyc_census_blocks`` 
 
  * name, popn_total, boroname, geom
 
* ``nyc_streets``
 
  * name, type, geom
   
* ``nyc_subway_stations``
 
  * name, routes, geom
 
* ``nyc_neighborhoods``
 
  * name, boroname, geom

Exercises
---------

* **What subway station is in 'Little Italy'? What subway route is it on?**
 
  .. code-block:: sql
 
    SELECT s.name, s.routes 
    FROM nyc_subway_stations AS s
    JOIN nyc_neighborhoods AS n 
    ON ST_Contains(n.geom, s.geom)  
    WHERE n.name = 'Little Italy';

  :: 
  
      name    | routes 
   -----------+--------
    Spring St | 6
   
* **What are all the neighborhoods served by the 6-train?** (Hint: The ``routes`` column in the ``nyc_subway_stations`` table has values like 'B,D,6,V' and 'C,6')
 
  .. code-block:: sql
  
    SELECT DISTINCT n.name, n.boroname 
    FROM nyc_subway_stations AS s
    JOIN nyc_neighborhoods AS n 
    ON ST_Contains(n.geom, s.geom)  
    WHERE strpos(s.routes,'6') > 0;
    
  ::
  
            name        | boroname  
    --------------------+-----------
     Midtown            | Manhattan
     Hunts Point        | The Bronx
     Gramercy           | Manhattan
     Little Italy       | Manhattan
     Financial District | Manhattan
     South Bronx        | The Bronx
     Yorkville          | Manhattan
     Murray Hill        | Manhattan
     Mott Haven         | The Bronx
     Upper East Side    | Manhattan
     Chinatown          | Manhattan
     East Harlem        | Manhattan
     Greenwich Village  | Manhattan
     Parkchester        | The Bronx
     Soundview          | The Bronx

  .. note::
  
    We used the ``DISTINCT`` keyword to remove duplicate values from our result set where there were more than one subway station in a neighborhood.
    
* **After 9/11, the 'Battery Park' neighborhood was off limits for several days. How many people had to be evacuated?** (Hint: See Section 5.2 for detail on how to load relevant census data.)
 
  .. code-block:: sql
 
    SELECT Sum(popn_total)
    FROM nyc_neighborhoods AS n
    JOIN nyc_census_blocks_2000 AS c 
    ON ST_Intersects(n.geom, c.geom)  
    WHERE n.name = 'Battery Park';
   
  :: 

    9928
    
* **What neighborhood has the highest population density (persons/km2)?**
 
  .. code-block:: sql
   
    SELECT 
      n.name, 
      Sum(c.popn_total) / (ST_Area(n.geom) / 1000000.0) AS popn_per_sqkm
    FROM nyc_census_blocks AS c
    JOIN nyc_neighborhoods AS n
    ON ST_Intersects(c.geom, n.geom)
    GROUP BY n.name, n.geom
    ORDER BY 2 DESC;
     
  ::
   
          name         |  popn_per_sqkm   
    -------------------+------------------
     North Sutton Area | 68435.13283772678   

     
