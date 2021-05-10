.. _joins:

Spatial Joins
=============

Spatial joins are the bread-and-butter of spatial databases.  They allow you to combine information from different tables by using spatial relationships as the join key.  Much of what we think of as "standard GIS analysis" can be expressed as spatial joins.

In the previous section, we explored spatial relationships using a two-step process: first we extracted a subway station point for 'Broad St'; then, we used that point to ask further questions such as "what neighborhood is the 'Broad St' station in?"

Using a spatial join, we can answer the question in one step, retrieving information about the subway station and the neighborhood that contains it:

.. code-block:: sql

  SELECT 
    subways.name AS subway_name, 
    neighborhoods.name AS neighborhood_name, 
    neighborhoods.boroname AS borough
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_subway_stations AS subways
  ON ST_Contains(neighborhoods.geom, subways.geom)
  WHERE subways.name = 'Broad St';

:: 

   subway_name | neighborhood_name  |  borough  
  -------------+--------------------+-----------
   Broad St    | Financial District | Manhattan

We could have joined every subway station to its containing neighborhood, but in this case we wanted information about just one.  Any function that provides a true/false relationship between two tables can be used to drive a spatial join, but the most commonly used ones are: :command:`ST_Intersects`, :command:`ST_Contains`, and :command:`ST_DWithin`.

Join and Summarize
------------------

The combination of a ``JOIN`` with a ``GROUP BY`` provides the kind of analysis that is usually done in a GIS system.

For example: **"What is the population and racial make-up of the neighborhoods of Manhattan?"** Here we have a question that combines information from about population from the census with the boundaries of neighborhoods, with a restriction to just one borough of Manhattan.

.. code-block:: sql

  SELECT 
    neighborhoods.name AS neighborhood_name, 
    Sum(census.popn_total) AS population,
    100.0 * Sum(census.popn_white) / Sum(census.popn_total) AS white_pct,
    100.0 * Sum(census.popn_black) / Sum(census.popn_total) AS black_pct
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_census_blocks AS census
  ON ST_Intersects(neighborhoods.geom, census.geom)
  WHERE neighborhoods.boroname = 'Manhattan'
  GROUP BY neighborhoods.name
  ORDER BY white_pct DESC;

::

    neighborhood_name  | population | white_pct | black_pct 
  ---------------------+------------+-----------+-----------
   Carnegie Hill       |      18763 |      90.1 |       1.4
   North Sutton Area   |      22460 |      87.6 |       1.6
   West Village        |      26718 |      87.6 |       2.2
   Upper East Side     |     203741 |      85.0 |       2.7
   Soho                |      15436 |      84.6 |       2.2
   Greenwich Village   |      57224 |      82.0 |       2.4
   Central Park        |      46600 |      79.5 |       8.0
   Tribeca             |      20908 |      79.1 |       3.5
   Gramercy            |     104876 |      75.5 |       4.7
   Murray Hill         |      29655 |      75.0 |       2.5
   Chelsea             |      61340 |      74.8 |       6.4
   Upper West Side     |     214761 |      74.6 |       9.2
   Midtown             |      76840 |      72.6 |       5.2
   Battery Park        |      17153 |      71.8 |       3.4
   Financial District  |      34807 |      69.9 |       3.8
   Clinton             |      32201 |      65.3 |       7.9
   East Village        |      82266 |      63.3 |       8.8
   Garment District    |      10539 |      55.2 |       7.1
   Morningside Heights |      42844 |      52.7 |      19.4
   Little Italy        |      12568 |      49.0 |       1.8
   Yorkville           |      58450 |      35.6 |      29.7
   Inwood              |      50047 |      35.2 |      16.8
   Washington Heights  |     169013 |      34.9 |      16.8
   Lower East Side     |      96156 |      33.5 |       9.1
   East Harlem         |      60576 |      26.4 |      40.4
   Hamilton Heights    |      67432 |      23.9 |      35.8
   Chinatown           |      16209 |      15.2 |       3.8
   Harlem              |     134955 |      15.1 |      67.1




What's going on here? Notionally (the actual evaluation order is optimized under the covers by the database) this is what happens:

#. The ``JOIN`` clause creates a virtual table that includes columns from both the neighborhoods and census tables. 
#. The ``WHERE`` clause filters our virtual table to just rows in Manhattan. 
#. The remaining rows are grouped by the neighborhood name and fed through the aggregation function to :command:`Sum()` the population values.
#. After a little arithmetic and formatting (e.g., ``GROUP BY``, ``ORDER BY``) on the final numbers, our query spits out the percentages.

.. note:: 

   The ``JOIN`` clause combines two ``FROM`` items.  By default, we are using an ``INNER JOIN``, but there are four other types of joins. For further information see the `join_type <http://www.postgresql.org/docs/9.1/interactive/sql-select.html#SQL-FROM>`_ definition in the PostgreSQL documentation.

We can also use distance tests as a join key, to create summarized "all items within a radius" queries. Let's explore the racial geography of New York using distance queries.

First, let's get the baseline racial make-up of the city.

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks;

:: 

      white_pct     |    black_pct     | popn_total 
  ------------------+------------------+------------
   44.0039500762811 | 25.5465789002416 |    8175032


So, of the 8M people in New York, about 44% are recorded as "white" and 26% are recorded as "black". 

Duke Ellington once sang that "You / must take the A-train / To / go to Sugar Hill way up in Harlem." As we saw earlier, Harlem has far and away the highest African-American population in Manhattan (80.5%). Is the same true of Duke's A-train?

First, note that the contents of the ``nyc_subway_stations`` table ``routes`` field is what we are interested in to find the A-train. The values in there are a little complex.

.. code-block:: sql

  SELECT DISTINCT routes FROM nyc_subway_stations;
  
:: 

 A,C,G
 4,5
 D,F,N,Q
 5
 E,F
 E,J,Z
 R,W

.. note::

   The ``DISTINCT`` keyword eliminates duplicate rows from the result.  Without the ``DISTINCT`` keyword, the query above identifies 491 results instead of 73.
   
So to find the A-train, we will want any row in ``routes`` that has an 'A' in it. We can do this a number of ways, but today we will use the fact that :command:`strpos(routes,'A')` will return a non-zero number only if 'A' is in the ``routes`` field.

.. code-block:: sql

   SELECT DISTINCT routes 
   FROM nyc_subway_stations AS subways 
   WHERE strpos(subways.routes,'A') > 0;
   
::

  A,B,C
  A,C
  A
  A,C,G
  A,C,E,L
  A,S
  A,C,F
  A,B,C,D
  A,C,E
  
Let's summarize the racial make-up of within 200 meters of the A-train line.

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks AS census
  JOIN nyc_subway_stations AS subways
  ON ST_DWithin(census.geom, subways.geom, 200)
  WHERE strpos(subways.routes,'A') > 0;

::

      white_pct     |    black_pct     | popn_total 
  ------------------+------------------+------------
   45.5901255900202 | 22.0936235670937 |     189824

So the racial make-up along the A-train isn't radically different from the make-up of New York City as a whole. 

Advanced Join
-------------

In the last section we saw that the A-train didn't serve a population that differed much from the racial make-up of the rest of the city. Are there any trains that have a non-average racial make-up?

To answer that question, we'll add another join to our query, so that we can simultaneously calculate the make-up of many subway lines at once. To do that, we'll need to create a new table that enumerates all the lines we want to summarize.

.. code-block:: sql

    CREATE TABLE subway_lines ( route char(1) );
    INSERT INTO subway_lines (route) VALUES 
      ('A'),('B'),('C'),('D'),('E'),('F'),('G'),
      ('J'),('L'),('M'),('N'),('Q'),('R'),('S'),
      ('Z'),('1'),('2'),('3'),('4'),('5'),('6'),
      ('7');

Now we can join the table of subway lines onto our original query.

.. code-block:: sql

    SELECT 
      lines.route,
      100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
      100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
      Sum(popn_total) AS popn_total
    FROM nyc_census_blocks AS census
    JOIN nyc_subway_stations AS subways
    ON ST_DWithin(census.geom, subways.geom, 200)
    JOIN subway_lines AS lines
    ON strpos(subways.routes, lines.route) > 0
    GROUP BY lines.route
    ORDER BY black_pct DESC;

::

     route | white_pct | black_pct | popn_total 
    -------+-----------+-----------+------------
     S     |      39.8 |      46.5 |      33301
     3     |      42.7 |      42.1 |     223047
     5     |      33.8 |      41.4 |     218919
     2     |      39.3 |      38.4 |     291661
     C     |      46.9 |      30.6 |     224411
     4     |      37.6 |      27.4 |     174998
     B     |      40.0 |      26.9 |     256583
     A     |      45.6 |      22.1 |     189824
     J     |      37.6 |      21.6 |     132861
     Q     |      56.9 |      20.6 |     127112
     Z     |      38.4 |      20.2 |      87131
     D     |      39.5 |      19.4 |     234931
     L     |      57.6 |      16.8 |     110118
     G     |      49.6 |      16.1 |     135012
     6     |      52.3 |      15.7 |     260240
     1     |      59.1 |      11.3 |     327742
     F     |      60.9 |       7.5 |     229439
     M     |      56.5 |       6.4 |     174196
     E     |      66.8 |       4.7 |      90958
     R     |      58.5 |       4.0 |     196999
     N     |      59.7 |       3.5 |     147792
     7     |      35.7 |       3.5 |     102401


As before, the joins create a virtual table of all the possible combinations available within the constraints of the ``JOIN ON`` restrictions, and those rows are then fed into a ``GROUP`` summary. The spatial magic is in the ``ST_DWithin`` function, that ensures only census blocks close to the appropriate subway stations are included in the calculation.

Function List
-------------

`ST_Contains(geometry A, geometry B) <http://postgis.net/docs/ST_Contains.html>`_: Returns true if and only if no points of B lie in the exterior of A, and at least one point of the interior of B lies in the interior of A.

`ST_DWithin(geometry A, geometry B, radius) <http://postgis.net/docs/ST_DWithin.html>`_: Returns true if the geometries are within the specified distance of one another. 

`ST_Intersects(geometry A, geometry B) <http://postgis.net/docs/ST_Intersects.html>`_: Returns TRUE if the Geometries/Geography "spatially intersect" - (share any portion of space) and FALSE if they don't (they are Disjoint). 

`round(v numeric, s integer) <http://www.postgresql.org/docs/current/interactive/functions-math.html>`_: PostgreSQL math function that rounds to s decimal places

`strpos(string, substring) <http://www.postgresql.org/docs/current/static/functions-string.html>`_: PostgreSQL string function that returns an integer location of a specified substring.

`sum(expression) <http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: PostgreSQL aggregate function that returns the sum of records in a set of records.

.. rubric:: Footnotes

.. [#PostGIS_Doco] http://postgis.net/docs/

