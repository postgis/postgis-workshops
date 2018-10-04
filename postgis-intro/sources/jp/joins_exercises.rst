.. _joins_exercises:

第13章: 空間結合 - 演習 -
===================================

以下は今までに見てきた関数の一部です。この演習ではこれらの関数を使用します。

* :command:`sum(expression)` はレコードの合計値を集計します。
* :command:`count(expression)` はレコードのサイズ(総数)を集計します。
* :command:`ST_Area(geometry)` はポリゴンのエリアを返します。
* :command:`ST_AsText(geometry)` はWKT形式の``テキスト``を返します。
* :command:`ST_Contains(geometry A, geometry B)` はジオメトリAがジオメトリBを含んでいるときにtrueを返します。
* :command:`ST_Distance(geometry A, geometry B)` はジオメトリAとジオメトリBの間の最短距離を返します。
* :command:`ST_DWithin(geometry A, geometry B, radius)` はジオメトリAがジオメトリBから特定の距離以内にあるときにtrueを返します。
* :command:`ST_GeomFromText(text)`は``ジオメトリ`` を返します。
* :command:`ST_Intersects(geometry A, geometry B)` はジオメトリAとジオメトリBが交差するあるいは部分的に重なるときにtrueを返します。
* :command:`ST_Length(linestring)` はラインの長さを返します。
* :command:`ST_Touches(geometry A, geometry B)` はジオメトリAの境界部分がジオメトリBと接しているときにtureを返します。
* :command:`ST_Within(geometry A, geometry B)` はジオメトリAがジオメトリBの内部に存在するときにtureを返します。
 
また、各テーブルの構成は次のようになっています:

 * ``nyc_census_blocks`` 
 
   * name, popn_total, boroname, the_geom
 
 * ``nyc_streets``
 
   * name, type, the_geom
   
 * ``nyc_subway_stations``
 
   * name, routes, the_geom
 
 * ``nyc_neighborhoods``
 
   * name, boroname, the_geom

演習
---------

 * **"'Little Italy'にある地下鉄駅はどこでしょう？またその駅はどこの路線でしょう？"**
 
   .. code-block:: sql
 
     SELECT s.name, s.routes 
     FROM nyc_subway_stations AS s
     JOIN nyc_neighborhoods AS n 
     ON ST_Contains(n.the_geom, s.the_geom)  
     WHERE n.name = 'Little Italy';

   :: 
  
       name    | routes 
    -----------+--------
     Spring St | 6
     
 * **"地下鉄の6番線沿いにある地域の名称をすべて求めなさい。"** (ヒント: ``nyc_subway_stations`` の ``routes`` 列の値は'B,D,6,V'や'C,6'のようになっています)
 
   .. code-block:: sql
  
    SELECT DISTINCT n.name, n.boroname 
    FROM nyc_subway_stations AS s
    JOIN nyc_neighborhoods AS n 
    ON ST_Contains(n.the_geom, s.the_geom)  
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

   .. 補足::
  
     地域によっては同じ路線の駅を複数持つこともあるので、 ``DISTINCT`` キーワードを使って重複するレコードは取り除いています。
    
 * **"9/11以降、'Battery Park'地域は数日の間立ち入り禁止区域に指定されました。これにより移動を余儀なくされた人々はどれくらいいたでしょうか？"**
 
   .. code-block:: sql
 
     SELECT Sum(popn_total)
     FROM nyc_neighborhoods AS n
     JOIN nyc_census_blocks AS c 
     ON ST_Intersects(n.the_geom, c.the_geom)  
     WHERE n.name = 'Battery Park';
   
   :: 

     9928
    
 * **"'Upper West Side'と'Upper East Side'の人口密度(人/km^2)はどれくらいでしょうか？"** (ヒント: 1 km^2 = 1000000 m^2)
 
   .. code-block:: sql
   
     SELECT 
       n.name, 
       Sum(c.popn_total) / (ST_Area(n.the_geom) / 1000000.0) AS popn_per_sqkm
     FROM nyc_census_blocks AS c
     JOIN nyc_neighborhoods AS n
     ON ST_Intersects(c.the_geom, n.the_geom)
     WHERE n.name = 'Upper West Side'
     OR n.name = 'Upper East Side'
     GROUP BY n.name, n.the_geom;
     
   ::
   
           name       |  popn_per_sqkm   
     -----------------+------------------
      Upper East Side | 47943.3590089405
      Upper West Side | 39729.5779474286

     