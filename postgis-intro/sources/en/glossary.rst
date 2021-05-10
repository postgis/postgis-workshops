.. _glossary:

Appendix B: Glossary
====================

.. glossary::

    CRS
        A "coordinate reference system". The combination of a geographic coordinate system and a projected coordinate system.

    GDAL
        `Geospatial Data Abstraction Library <http://gdal.org>`_, pronounced "GOO-duhl", an open source raster access library with support for a large number of formats, used widely in both open source and proprietary software.

    GeoJSON
        "Javascript Object Notation", a text format that is very fast to parse in Javascript virtual machines. In spatial, the extended specification for `GeoJSON <http://geojson.org>`_ is commonly used.
    
    GIS
        `Geographic information system <http://en.wikipedia.org/wiki/Geographic_information_system>`_ or geographical information system captures, stores, analyzes, manages, and presents data that is linked to location.
    
    GML
        `Geography Markup Language <http://www.opengeospatial.org/standards/gml>`_.  GML is the :term:`OGC` standard XML format for representing spatial feature information.

    JSON
        "`Javascript Object Notation <http://en.wikipedia.org/wiki/JSON>`_", a text format that is very fast to parse in Javascript virtual machines. In spatial, the extended specification for `GeoJSON <http://geojson.org>`_ is commonly used.

    JSTL
        "JavaServer Page Template Library", a tag library for :term:`JSP` that encapsulates many of the standard functions handled in JSP (database queries, iteration, conditionals) into a terse syntax.

    JSP
        "JavaServer Pages" a scripting system for Java server applications that allows the interleaving of markup and Java procedural code.

    KML
        "Keyhole Markup Language", the spatial XML format used by Google Earth. Google Earth was originally written by a company named "Keyhole", hence the (now obscure) reference in the name.

    OGC
        The `Open Geospatial Consortium <http://opengeospatial.org/>`_ (OGC) is a standards organization that develops specifications for geospatial services.

    OSGeo
         The `Open Source Geospatial Foundation <http://osgeo.org>`_ (OSGeo) is a non-profit foundation dedicated to the promotion and support of open source geospatial software.

    SFSQL
        The `Simple Features for SQL <http://www.opengeospatial.org/standards/sfs>`_ (SFSQL) specification from the :term:`OGC` defines the types and functions that make up a standard spatial database.

    SLD
        The `Styled Layer Descriptor <http://www.opengeospatial.org/standards/sld>`_ (SLD) specification from the :term:`OGC` defines an format for describing cartographic rendering of vector features.

    SRID
        "Spatial reference ID" a unique number assigned to a particular "coordinate reference system". The PostGIS table **spatial_ref_sys** contains a large collection of well-known srid values and text representations of the coordinate reference systems.

    SQL
        "`Structured query language <http://en.wikipedia.org/wiki/SQL>`_" is the standard means for querying relational databases.

    SQL/MM
        `SQL Multimedia <http://www.fer.hr/_download/repository/SQLMM_Spatial-_The_Standard_to_Manage_Spatial_Data_in_Relational_Database_Systems.pdf>`_; includes several sections on extended types, including a substantial section on spatial types.

    SVG
        "`Scalable vector graphics <http://en.wikipedia.org/wiki/Scalable_Vector_Graphics>`_" is a family of specifications of an XML-based file format for describing two-dimensional vector graphics, both static and dynamic (i.e. interactive or animated).

    WFS
        The `Web Feature Service <http://www.opengeospatial.org/standards/wfs>`_ (WFS) specification from the :term:`OGC` defines an interface for reading and writing geographic features across the web.

    WMS
        The `Web Map Service <http://www.opengeospatial.org/standards/wms>`_ (WMS) specification from the :term:`OGC` defines an interface for requesting rendered map images across the web.

    WKB
        "Well-known binary". Refers to the binary representation of geometries described in the Simple Features for SQL specification (:term:`SFSQL`).
        
    WKT
        "`Well-known text <http://en.wikipedia.org/wiki/Well-known_text>`_". Can refer either to the text representation of geometries, with strings starting "POINT", "LINESTRING", "POLYGON", etc. Or can refer to the text representation of a :term:`CRS`, with strings starting "PROJCS", "GEOGCS", etc.  Well-known text representations are :term:`OGC` standards, but do not have their own specification documents. The first descriptions of WKT (for geometries and for CRS) appeared in the :term:`SFSQL` 1.0 specification.
        

  