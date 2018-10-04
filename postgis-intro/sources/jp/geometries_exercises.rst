.. _geometries_exercises:

第9章: ジオメトリ型の演習
=============================

今まで見てきた関数をここに並べました。これらの関数は演習をするのに役立ちますよ！

 * :command:`sum(expression)` レコードセットの総和を集計する
 * :command:`count(expression)` レコードセットの数を集計する
 * :command:`ST_GeometryType(geometry)` ジオメトリーの型を返す
 * :command:`ST_NDims(geometry)` ジオメトリーの次元数を返す
 * :command:`ST_SRID(geometry)` ジオメトリーの空間参照情報の番号を返す
 * :command:`ST_X(point)` X座標を返す
 * :command:`ST_Y(point)` Y座標を返す
 * :command:`ST_Length(linestring)` ラインストリングの長さを返す
 * :command:`ST_StartPoint(geometry)` ジオメトリーの最初の座標をポイントとして返す
 * :command:`ST_EndPoint(geometry)` ジオメトリーの最後の座標をポイントとして返す
 * :command:`ST_NPoints(geometry)` ラインストリング内の座標の数を返す
 * :command:`ST_Area(geometry)` ポリゴンの面積を返す
 * :command:`ST_NRings(geometry)` リングの数を返す（通常は1。穴があればそれ以上の数が返る）
 * :command:`ST_ExteriorRing(polygon)` 外側のリングをラインストリングとして返す
 * :command:`ST_InteriorRingN(polygon, integer)` ある内側のリングをラインストリングとして返す
 * :command:`ST_Perimeter(geometry)` 全てのリングの総延長を返す
 * :command:`ST_NumGeometries(multi/geomcollection)` ジオメトリーの集合体を構成する各部分の数を返す
 * :command:`ST_GeometryN(geometry, integer)` 集合体のある部分の図形を返す
 * :command:`ST_GeomFromText(text)` ジオメトリーを返す（ ``geometry`` 型）
 * :command:`ST_AsText(geometry)` WKTを返す（ ``text`` 型）
 * :command:`ST_AsEWKT(geometry)` EWKTを返す（ ``text`` 型）
 * :command:`ST_GeomFromWKB(bytea)` ジオメトリーを返す（ ``geometry`` 型）
 * :command:`ST_AsBinary(geometry)` WKBを返す（ ``bytea`` 型）
 * :command:`ST_AsEWKB(geometry)` EWKBを返す（ ``bytea`` 型）
 * :command:`ST_GeomFromGML(text)` ジオメトリーを返す（ ``geometry`` 型）
 * :command:`ST_AsGML(geometry)` GMLを返す（ ``text``型）
 * :command:`ST_GeomFromKML(text)` ジオメトリーを返す（ ``geometry`` 型）
 * :command:`ST_AsKML(geometry)` KMLを返す（ ``text`` 型）
 * :command:`ST_AsGeoJSON(geometry)` JSONを返す（ ``text`` 型）
 * :command:`ST_AsSVG(geometry)` SVGを返す（ ``text`` 型）

また、ここまでに利用可能なテーブルは以下のようになっています。

 * ``nyc_census_blocks`` 
 
   * name, popn_total, boroname, the_geom
 
 * ``nyc_streets``
 
   * name, type, the_geom
   
 * ``nyc_subway_stations``
 
   * name, the_geom
 
 * ``nyc_neighborhoods``
 
   * name, boroname, the_geom

演習
---------

 * **"West Village地区の面積はいくらでしょう？"**

   .. code-block:: sql

     SELECT ST_Area(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';
       
   :: 

     1044614.53027344

   .. note::

      面積は平方メートルの単位で与えられます.ヘクタール単位で面積を計算する場合、10000で割ってください。エーカー単位で計算する場合は4047で割ります。

 * **"マンハッタンの面積は、エーカーで計算するといくらでしょう？"** (ヒント: ``nyc_census_blocks`` と ``nyc_neighbourhoods`` テーブルは、 ``boroname`` フィールドを持っています。)
 
   .. code-block:: sql

     SELECT Sum(ST_Area(the_geom)) / 4047
       FROM nyc_neighborhoods
       WHERE boroname = 'Manhattan';

   :: 
   
     13965.3201224118

   あるいは・・・

   .. code-block:: sql

     SELECT Sum(ST_Area(the_geom)) / 4047
       FROM nyc_census_blocks
       WHERE boroname = 'Manhattan';

   :: 
   
     14572.1575543757


 * **"ニューヨーク市内で、穴の開いているセンサス・ブロックはいくつあるでしょう？"**

   .. code-block:: sql

     SELECT Count(*) 
       FROM nyc_census_blocks
       WHERE ST_NRings(the_geom) > 1;

   :: 
   
     66 
   
 * **"ニューヨーク市内の通りの総延長はいくらでしょう？（キロメートルで）"** (ヒント: 空間データの長さをの単位はメートルです。1キロメートル=1000メートルです。)

  
    .. code-block:: sql

     SELECT Sum(ST_Length(the_geom)) / 1000
       FROM nyc_streets;

   :: 
   
     10418.9047172

 * **"'Columbus Cir' (Columbus Circle)の長さはいくらでしょう？**
 
     .. code-block:: sql
 
      SELECT ST_Length(the_geom)
        FROM nyc_streets
        WHERE name = 'Columbus Cir';

     :: 
   
       308.34199

 * **"West Villageの境界をJSON形式で表現するとどのようになるでしょう？"**
  
   .. code-block:: sql

     SELECT ST_AsGeoJSON(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';

   ::
     
      {"type":"MultiPolygon","coordinates":
       [[[[583263.2776595836,4509242.6260239873],
          [583276.81990686338,4509378.825446927], ...
          [583263.2776595836,4509242.6260239873]]]]}

ジオメトリーの型は"MultiPolygon"になっています。面白いですね！
   
      
 * **"マルチポリゴンであるWest Villageは、いくつのポリゴンで構成されているでしょう？"**
 
   .. code-block:: sql

     SELECT ST_NumGeometries(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';

   ::

      1
       
   .. note::
   
      空間テーブル内では、1つの要素からなるマルチポリゴンは異例なことではありません。マルチポリゴンを使うと、複数のgeometry型を使わずに1つのgeometry型で1つの、あるいは複数の図形を格納することができます。
       
 * **"ニューヨーク市内の通りの長さを、その種別で分類して求めてください。"**
 
   .. code-block:: sql

      SELECT type, Sum(ST_Length(the_geom)) AS length
       FROM nyc_streets
       GROUP BY type
       ORDER BY length DESC;

   ::
   
                            type                       |      length      
     --------------------------------------------------+------------------
      residential                                      | 8629870.33786606
      motorway                                         | 403622.478126363
      tertiary                                         | 360394.879051303
      motorway_link                                    | 294261.419479668
      secondary                                        | 276264.303897926
      unclassified                                     | 166936.371604458
      primary                                          | 135034.233017947
      footway                                          | 71798.4878378096
      service                                          |  28337.635038596
      trunk                                            | 20353.5819826076
      cycleway                                         | 8863.75144825929
      pedestrian                                       | 4867.05032825026
      construction                                     | 4803.08162103562
      residential; motorway_link                       | 3661.57506293745
      trunk_link                                       | 3202.18981240201
      primary_link                                     | 2492.57457083536
      living_street                                    | 1894.63905457332
      primary; residential; motorway_link; residential | 1367.76576941335
      undefined                                        |  380.53861910346
      steps                                            | 282.745221342127
      motorway_link; residential                       |  215.07778911517

    
   .. note::

      ``ORDER BY length DESC`` 句は、lengthを降順に並び替えます。最も値の大きい種別がリストの最初に出てきます。
 
 
 
        