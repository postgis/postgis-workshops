.. _projection:

第15章: データの投影
===========================

地表は平面ではないため、平面である紙の地図（またはコンピュータの画面）に表現することは単純ではありません。それぞれ長所と短所を含みつつも、人々は巧みな方法を考えだしてきました。地図上のすべてのものの大きさの比が保存され、面積が正しく表現される投影法、メルカトル図法のように角度（正角）が正しく表現される投影法があります。また、ある一定の範囲内で歪みが小さくなるように、投影法をうまく組み合わせる投影法もあります。球体の世界を平面直角座標系に変換していることはすべての投影法に共通であるため、データの用途に応じてどの投影法が適切か選択する必要があります。

:ref:`nyc（ニューヨーク市）データの読み込み <loading_data>` の際に、すでに投影法を扱っています。（SRID 26918があったことを確認してください）。空間参照システム間で投影変換を行い、データを投影し直すことが必要になることがあります。PostGISでは、 :command:`ST_Transform(geometry, srid)` 関数が用意されていて、データの投影法を変更する機能があります。ジオメトリの空間参照IDを管理するために、PostGISでは、 :command:`ST_SRID(geometry)` および  :command:`ST_SetSRID(geometry, srid)` 関数が用意されています。

:command:`ST_SRID` コマンドでデータのSRIDを確認することができます。

.. code-block:: sql

  SELECT ST_SRID(the_geom) FROM nyc_streets LIMIT 1;
  
::

  26918

「26918」とは何の定義でしょうか。":ref:`データのローディング <loading_data>`" 章を参照すると、 ``spatial_ref_sys`` テーブルにその定義の記載があります。実は、 **２種類** の定義が記載されています。ひとつは「well-known text(:term:`WKT`)」による定義で、``srtext`` フィールドにあります。もうひとつは ``proj4text`` フィールドにある "proj.4" 形式による定義です。

.. code-block:: sql

   SELECT * FROM spatial_ref_sys WHERE srid = 26918;

実は、PostGISによる再投影計算の内部処理では、 ``proj4text`` 列での定義が使用されています。26918の投影法のproj.4テキストは次のSQL文で表示することができます。

.. code-block:: sql

  SELECT proj4text FROM spatial_ref_sys WHERE srid = 26918;

::

  +proj=utm +zone=18 +ellps=GRS80 +datum=NAD83 +units=m +no_defs 

実際のところ、 ``srtext`` 列と ``proj4text`` フィールドは両者とも重要なフィールドです。 ``srtext`` フィールドは、 `GeoServer <http://geoserver.org>`_ や `uDig <udig.refractions.net>`_、 `FME <http://www.safe.com/>`_ といった外部プログラムが使用します。 ``proj4text`` フィールドは内部で使用します。

データの比較
--------------

座標系とSRIDを組み合わせると、地球上での位置を表現することができます。SRIDがなければ、座標系は単なる抽象的な概念を表現するものになります。平面 “直角”座標系は、地球の表面に置かれた“平面”の座標系として定義されます。PostGISの関数は、このような平面を想定しているため、比較演算においては、両者が同一のSRIDで表現されたジオメトリである必要があります。

異なるSRIDで表現されたジオメトリで演算しようとすると、エラーが返されます。

.. code-block:: sql
  SELECT ST_Equals(
           ST_GeomFromText('POINT(0 0)', 4326),
           ST_GeomFromText('POINT(0 0)', 26918)
           );

::

  ERROR:  Operation on two geometries with different SRIDs
  CONTEXT:  SQL function "st_equals" statement 1

.. note::
   オン・ザ・フライ変換における :command:`ST_Transform` コマンドに頼りすぎるのは禁物です。空間インデックスはメモリに記憶されているジオメトリのSRIDを使用して構築されます。異なるSRIDで変換が行われると、多くの場合で空間インデックスは使用されません。データベースの中のすべてのテーブルを **ひとつのSRID** に統一しておくことをお勧めします。外部アプリケーションでデータを読み書きする場合のみ変換する関数を使用するとよいでしょう。


データ変換
-----------------

SRID 26918のproj4上の定義を戻り値として受け取れれば、それがUTM（ユニバーサル横メルカトル）ゾーン18（単位：メートル）であることがわかります。

::

   +proj=utm +zone=18 +ellps=GRS80 +datum=NAD83 +units=m +no_defs 

手元にあるデータを現在の座標系から"緯度経度"で表現される地理座標系に変換してみましょう。

あるSRIDから別のSRIDにデータを変換するためには、ジオメトリが有効なSRIDで定義されていることを確認してください。SRIDが有効であることが既に確認している場合は、次に変換する先の座標系SRIDが必要です。別の言い方をすると、地理座標系のSRIDは何かということです。

一般的には地理座標系のSRIDは"WGS84楕円体における緯度経度"に対応する4326を指します。定義は「spatialreference.org」のサイトでみることができます。

  http://spatialreference.org/ref/epsg/4326/

``spatial_ref_sys`` テーブルから定義を引き出すこともできます。

.. code-block:: sql

  SELECT srtext FROM spatial_ref_sys WHERE srid = 4326;

::

  GEOGCS["WGS 84",
    DATUM["WGS_1984",
      SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],
      AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]

地下鉄「Broad通り」駅の座標を地理座標系に変換しましょう。

.. code-block:: sql

  SELECT ST_AsText(ST_Transform(the_geom,4326)) 
  FROM nyc_subway_stations 
  WHERE name = 'Broad St';

::

  POINT(-74.0106714688735 40.7071048155841)

SRIDを指定せずにデータを読み込む、あるいは、ジオメトリを作成するためには、SRIDの値を「-1」にします。 :ref:`geometries` において、 ``geoemetries`` テーブルを作成した際にSRIDを指定しなかったことを確認してください。データベースにクエリを発行する際に、``geometries``テーブルのSRIDの初期値が「-1」であるため、名称に ``nyc_`` を含むすべてのテーブルはSRIDが26918として処理されます。

テーブルのSRIDの割り当てを参照するには、データベースの ``geometry_columns`` テーブルにクエリを発行します。

.. code-block:: sql

  SELECT f_table_name AS name, srid 
  FROM geometry_columns;

::

          name         | srid  
  ---------------------+-------
   nyc_census_blocks   | 26918
   nyc_neighborhoods   | 26918
   nyc_streets         | 26918
   nyc_subway_stations | 26918
   geometries          |    -1

  
しかしながら、座標系のSRIDが既知である場合は、ジオメトリに対して :command:`ST_SetSRID` コマンドを使用して、SRIDを後で設定することができます。ジオメトリを別の座標系に変換することができるようになります。

.. code-block:: sql
   SELECT ST_AsText(
    ST_Transform(
      ST_SetSRID(geom,26918),
    4326)
   )
   FROM geometries;

関数一覧
-------------
`ST_AsText <http://postgis.net/docs/manual-2.0/ST_AsText.html>`_:  ジオメトリまたはジオグラフィをWell-Known Text (WKT)表記で返します。SRIDメタデータは表示されません。

`ST_SetSRID(geometry, srid) <http://postgis.net/docs/manual-2.0/ST_SetSRID.html>`_:ジオメトリのSRIDを指定された整数値で設定します。

`ST_SRID(geometry) <http://postgis.net/docs/manual-2.0/ST_SRID.html>`_: spatial_ref_sysテーブルで定義されたST_Geometryの空間参照IDを返します。

`ST_Transform(geometry, srid) <http://postgis.net/docs/manual-2.0/ST_Transform.html>`_: 指定された整数値をSRIDとして座標系を変換し、新しいジオメトリを返します。
