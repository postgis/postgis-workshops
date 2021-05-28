.. _validity:

Validity
========

In 90% of the cases the answer to the question, "why is my query giving me a 'TopologyException' error" is "one or more of the inputs are invalid".  Which begs the question: what does it mean to be invalid, and why should we care?

What is Validity
----------------

Validity is most important for polygons, which define bounded areas and require a good deal of structure. Lines are very simple and cannot be invalid, nor can points.

Some of the rules of polygon validity feel obvious, and others feel arbitrary (and in fact, are arbitrary).

* Polygon rings must close.
* Rings that define holes should be inside rings that define exterior boundaries.
* Rings may not self-intersect (they may neither touch nor cross themselves).
* Rings may not touch other rings, except at a point.
* Elements of multi-polygons may not touch each other.

The last three rules are in the arbitrary category. There are other ways to define polygons that are equally self-consistent but the rules above are the ones used by the :term:`OGC` :term:`SFSQL` standard that PostGIS conforms to.

The reason the rules are important is because algorithms for geometry calculations depend on consistent structure in the inputs. It is possible to build algorithms that have no structural assumptions, but those routines tend to be very slow, because the first step in any structure-free routine is to *analyze the inputs and build structure into them*.

Here's an example of why structure matters. This polygon is invalid:

::

  POLYGON((0 0, 0 1, 2 1, 2 2, 1 2, 1 0, 0 0));
  
You can see the invalidity a little more clearly in this diagram:

.. image:: ./validity/figure_eight.png

The outer ring is actually a figure-eight, with a self-intersection in the middle. Note that the graphic routines successfully render the polygon fill, so that visually it is appears to be an "area": two one-unit squares, so a total area of two units of area.

Let's see what the database thinks the area of our polygon is:

.. code-block:: sql

  SELECT ST_Area(ST_GeometryFromText(
           'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
         ));
  
::

    st_area 
   ---------
          0

What's going on here? The algorithm that calculates area assumes that rings do not self-intersect. A well-behaved ring will always have the area that is bounded (the interior) on one side of the bounding line (it doesn't matter which side, just that it is on *one* side). However, in our (poorly behaved) figure-eight, the bounded area is to the right of the line for one lobe and to the left for the other. This causes the areas calculated for each lobe to cancel out (one comes out as 1, the other as -1) hence the "zero area" result.


Detecting Validity
------------------

In the previous example we had one polygon that we **knew** was invalid. How do we detect invalidity in a table with millions of geometries? With the :command:`ST_IsValid(geometry)` function. Used against our figure-eight, we get a quick answer:

.. code-block:: sql

  SELECT ST_IsValid(ST_GeometryFromText(
           'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
         ));

:: 

  f

Now we know that the feature is invalid, but we don't know why. We can use the :command:`ST_IsValidReason(geometry)` function to find out the source of the invalidity:

.. code-block:: sql

  SELECT ST_IsValidReason(ST_GeometryFromText(
           'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
         ));

::

  Self-intersection[1 1]

Note that in addition to the reason (self-intersection) the location of the invalidity (coordinate (1 1)) is also returned.

We can use the :command:`ST_IsValid(geometry)` function to test our tables too:

.. code-block:: sql

  -- Find all the invalid polygons and what their problem is
  SELECT name, boroname, ST_IsValidReason(geom)
  FROM nyc_neighborhoods
  WHERE NOT ST_IsValid(geom);

::

           name           |   boroname    |          st_isvalidreason
 -------------------------+---------------+-----------------------------------------
  Howard Beach            | Queens        | Self-intersection[597264.08 4499924.54]
  Corona                  | Queens        | Self-intersection[595483.05 4513817.95]
  Steinway                | Queens        | Self-intersection[593545.57 4514735.20]
  Red Hook                | Brooklyn      | Self-intersection[584306.82 4502360.51]



Repairing Invalidity
--------------------

Repairing invalidity involves stripping a polygon down to its simplest structures (rings), ensuring the rings follow the rules of validity, then building up new polygons that follow the rules of ring enclosure. Frequently the results are intuitive, but in the case of extremely ill-behaved inputs, the valid outputs may not conform to your intuition of how they should look. Recent versions of PostGIS include different algorithms for geometry repair: read the `manual page <http://postgis.net/docs/ST_MakeValid.html>`_ carefully and choose the one you like best.

For example, here's a classic invalidity -- the "banana polygon" -- a single ring that encloses an area but bends around to touch itself, leaving a "hole" which is not actually a hole.

::

  POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))

.. image:: ./validity/banana.png
  :class: inline

Running `ST_MakeValid <http://postgis.net/docs/ST_MakeValid.html>`_ on the polygon returns a valid :term:`OGC` polygon, consisting of an outer and inner ring that touch at one point.

.. code-block:: sql

  SELECT ST_AsText(
           ST_Buffer(
             ST_GeometryFromText('POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))'),
             0.0
           )
         );

::

  POLYGON((0 0,0 4,4 4,4 0,2 0,0 0),(2 0,3 1,2 2,1 1,2 0))

.. note::

  The "banana polygon" (or "inverted shell") is a case where the :term:`OGC` topology model for valid geometry and the model used internally by ESRI differ. The ESRI model considers rings that touch to be invalid, and prefers the banana form for this kind of shape. The OGC model is the reverse. Neither is "correct", they are just different ways to model the same situation.


Bulk Validity Repair
--------------------

Here's an example of SQL to flag invalid geometries for review while adding a repaired version to the table.

.. code-block:: sql

  -- Column for old invalid form
  ALTER TABLE nyc_neighborhoods
    ADD COLUMN geom_invalid geometry
    DEFAULT NULL;

  -- Fix invalid and save the original
  UPDATE nyc_neighborhoods
    SET geom = ST_MakeValid(geom),
        invalid_geom = geom
    WHERE NOT ST_IsValid(geom);

  -- Review the invalid cases
  SELECT geom, ST_IsValidReason(geom_invalid)
    FROM nyc_neighborhoods
    WHERE geom_invalid IS NOT NULL;

A good tool for visually repairing invalid geometry is OpenJump (http://openjump.org) which includes a validation routine under **Tools->QA->Validate Selected Layers**.


Function List
-------------

`ST_IsValid(geometry A) <http://postgis.net/docs/ST_IsValid.html>`_: Returns a boolean indiciting whether the geometery is valid.

`ST_IsValidReason(geometry A) <http://postgis.net/docs/ST_IsValidReason.html>`_: Returns a text string with the reason for the invalidity and a coordinate of invalidity.

`ST_MakeValid(geometry A) <http://postgis.net/docs/ST_MakeValid.html>`_: Returns a geometry re-constructed to obey the validity rules.



