.. _projection_exercises:

第16章：投影法についての演習
================================

これまで見てきた関数のおさらいをしておきます。ヒント：本実践で役に立つものばかりです。

* :command:`sum(expression)`: レコードセットの合計値を返します。
* :command:`ST_Length(linestring)`: ラインストリングの長さを返します。
* :command:`ST_SRID(geometry, srid)`: ジオメトリのSRIDを返します。
* :command:`ST_Transform(geometry, srid)`: 異なる空間参照系へジオメトリを変換します。
* :command:`ST_GeomFromText(text)`: ジオメトリを返します。
* :command:`ST_AsText(geometry)`: WKTテキストを返します。
* :command:`ST_AsGML(geometry)`: GMLテキストを返します。

オンラインで下記の情報を参照することができます。

* http://spatialreference.org
* http://prj2epsg.org


下記のテーブルが利用できることを確認してください。

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

 * **UTM ゾーン18で計測した場合のニューヨーク市内にある道路の合計距離は？**

   .. code-block:: sql

     SELECT Sum(ST_Length(the_geom))
       FROM nyc_streets;

   :: 
   
     10418904.7172

 * **SRID 2831のWKTでの定義は？**

   .. code-block:: sql

     SELECT srtext FROM spatial_ref_sys
     WHERE SRID = 2831;


※ `prj2epsg <http://prj2epsg.org/epsg/2831>`_ でも知ることができます。


 ::

  PROJCS["NAD83(HARN) / New York Long Island", 
  GEOGCS["NAD83(HARN)", 
    DATUM["NAD83 (High Accuracy Regional Network)", 
      SPHEROID["GRS 1980", 6378137.0, 298.257222101, AUTHORITY["EPSG","7019"]], 
      TOWGS84[-0.991, 1.9072, 0.5129, 0.0257899075194932, -0.009650098960270402, -0.011659943232342112, 0.0], 
      AUTHORITY["EPSG","6152"]], 
    PRIMEM["Greenwich", 0.0, AUTHORITY["EPSG","8901"]], 
    UNIT["degree", 0.017453292519943295], 
    AXIS["Geodetic longitude", EAST], 
    AXIS["Geodetic latitude", NORTH], 
    AUTHORITY["EPSG","4152"]], 
  PROJECTION["Lambert Conic Conformal (2SP)", AUTHORITY["EPSG","9802"]], 
  PARAMETER["central_meridian", -74.0], 
  PARAMETER["latitude_of_origin", 40.166666666666664], 
  PARAMETER["standard_parallel_1", 41.03333333333333], 
  PARAMETER["false_easting", 300000.0], 
  PARAMETER["false_northing", 0.0], 
  PARAMETER["scale_factor", 1.0], 
  PARAMETER["standard_parallel_2", 40.666666666666664], 
  UNIT["m", 1.0], 
  AXIS["Easting", EAST], 
  AXIS["Northing", NORTH], 
  AUTHORITY["EPSG","2831"]]


 * **SRID 2831で計測した場合のニューヨーク市内にある道路の合計距離は？**


   .. code-block:: sql

     SELECT Sum(ST_Length(ST_Transform(the_geom,2831)))
       FROM nyc_streets;

   :: 
   
     10421993.706374

   .. note::

     UTM 18とState Plane Long islandとの計測値差異は(10,421,993 - 10,418,904) / 10,418,904であり、0.02%です。セクション17のジオグラフィで回転楕円体上で計算したところ、道路の総距離は10,421,999で、State Planeでの値に近い結果を得ました。これは珍しいことではなく、Stateplane Long island投影法はニューヨーク市というとても狭い地域に対して正確に計測できるようになっているからです。一方でUTM ゾーン18はより広い地域において妥当な結果が得られるように定義されています。

 * **地下鉄「Broad通り」駅の地点をKMLで表現するとどうなりますか？**

   .. code-block:: sql
   
     SELECT ST_AsKML(the_geom) 
     FROM nyc_subway_stations
     WHERE name = 'Broad St';
     
   :: 
   
     <Point><coordinates>-74.010671468873468,40.707104815584088</coordinates></Point>

:command:`ST_Transform` を呼び出していないにもかかわらず、座標値が地理座標系で表示されています。なぜでしょうか？
KML標準においては、すべての座標は地理座標系（EPSG 4326）の範囲内にある必要があり、そのため、 :command:`ST_AsKML` 関数が自動的に変換することができるのです。
