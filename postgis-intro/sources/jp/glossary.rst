.. _glossary:

付録 B: 用語解説
====================

.. glossary::

    CRS
        座標参照系のことで、測地座標系と投影座標系の組み合わせからなります。

    GDAL
        `Geospatial Data Abstraction Library <http://gdal.org>`_ の略で、オープンソースのラスタ・アクセス・ライブラリです。多様なフォーマットに対応し、オープンソース・ソフトウェアとプロプライエタリ・ソフトウェアの両方で幅広く利用されています。"GOO-duhl"と発音します。

    GeoJSON
        JavaScriptバーチャルマシン上で非常に高速に解析可能なテキストフォーマットのことで、"Javascript Object Notation"の略です。空間分野では `GeoJSON <http://geojson.org>`_ という拡張仕様が一般的に利用されています。
    
    GIS
        `Geographic information system <http://ja.wikipedia.org/wiki/%E5%9C%B0%E7%90%86%E6%83%85%E5%A0%B1%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0>`_ 、もしくは、"geographical information system"。位置情報に関連するデータの取込み、保存、分析、管理、表現するためのシステムです。
    
    GML
        `Geography Markup Language <http://www.opengeospatial.org/standards/gml>`_ の略です。GMLは:term:`OGC`スタンダードの規約で、空間的なフィーチャ情報を表現するためのXMLフォーマットです。

    JSON
        JavaScriptバーチャルマシン上で非常に高速にパース可能なテキストフォーマットのことで、"Javascript Object Notation"の略です。空間分野では `GeoJSON <http://geojson.org>`_ という拡張仕様が一般的に利用されています。

    JSTL
        :term:`JSP`用のタグライブラリである"JavaServer Page Template Library"の略で、JSPにおいてデータベースへの問い合わせや、繰り返し、条件文などをハンドリングする多数の標準関数を、簡潔な構文にカプセル化したものです。

    JSP
        "JavaServer Pages"の略で、マークアップとJavaの実行コードのインタリーブを可能にするJavaサーバ・アプリケーション用のスクリプト・システムのことです。

    KML
        "Keyhole Markup Language"の略で、Google Earthで利用されている空間的なXMLフォーマットのことです。Google Earthは当初は"Keyhole"という名前の会社によって開発されたため、その名称に名残が残っています。

    OGC
        `The Open Geospatial Consortium <http://opengeospatial.org/>`_ (OGC)は、地理空間サービスの仕様を整備する標準機関です。

    OSGeo
        `The Open Source Geospatial Foundation <http://osgeo.org>`_ (OSGeo)は、オープンソース地理空間システムの促進とサポートに献身する非営利財団です。

    SFSQL
        `Simple Features for SQL <http://www.opengeospatial.org/standards/sfs>`_ (SFSQL)は、標準的な空間データベースを構築するためのデータ型や関数を定義した:term:`OGC`の仕様です。

    SLD
        `Styled Layer Descriptor <http://www.opengeospatial.org/standards/sld>`_ (SLD)は、:term:`OGC`によって定義されている、ベクタ・フィーチャーを地図上にどのようにレンダリングするかを記述するフォーマットの仕様です。

    SRID
        "Spatial reference ID"の略で、特定の"空間参照系"にユニークな数値を割り当てたものです。PostGISの**spatial_ref_sys**テーブルには、よく利用される非常に多くSRIDと、テキスト表現された座標参照系が格納されています。

    SQL
        "Structured query language"の略で、リレーショナルデータベースへの問い合わせに利用される標準的な言語です。詳しくは http://ja.wikipedia.org/wiki/SQL を参照してください。

    SQL/MM
        `SQL Multimedia <http://www.fer.hr/_download/repository/SQLMM_Spatial-_The_Standard_to_Manage_Spatial_Data_in_Relational_Database_Systems.pdf>`_ の略です。拡張データ型の項もいくつか含まれており、さらにそこには相当数の空間データ型が記述されています。

    SVG
        "Scalable vector graphics"の略で、静的・動的（対話式やアニメーション）に2次元のベクターグラフィックを描画するための、XMLをベースとするフォーマット仕様です。詳しくは http://ja.wikipedia.org/wiki/Scalable_Vector_Graphics を参照してください。

    WFS
        `Web Feature Service <http://www.opengeospatial.org/standards/wfs>`_ (WFS) は :term:`OGC` によって定義されている、Webを通じた地理的フィーチャーの入出力インタフェース仕様のことです。

    WMS
        `Web Map Service <http://www.opengeospatial.org/standards/wms>`_ (WMS) は :term:`OGC` によって定義されている、レンダリングされた地図画像をWebを通してリクエストするためのインタフェース仕様です。

    WKB
        "Well-known binary"の略です。Simple Feature for SQL(:term:`SFSQL`)の、ジオメトリのバイナリ表現を参照してください。
        
    WKT
        "Well-known text"の略です。"POINT"、"LINESTRING"、"POLYGON"などで始まる、ジオメトリをテキスト表現した文字列のことか、"PROJCS"、"GEOGCS"などで始まる、:term:`CRS` をテキスト表現したものを指します。WKT表現は :term:`OGC` 標準ですが、固有の仕様書はありません。ジオメトリとCRSについてのWKTが最初に登場したのは、 :term:`SFSQL` のバージョン1.0の仕様書です。        
