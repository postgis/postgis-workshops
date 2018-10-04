.. _spatial_relationships_exercises:

第11章: 空間演算 - 演習 -
===========================================

前節で紹介した関数のおさらいをしましょう。この演習ではこれらの関数を使用します。

* :command:`sum(expression)` はレコードの合計値を集計します。
* :command:`count(expression)` はレコードのサイズ(総数)を集計します。
* :command:`ST_Contains(geometry A, geometry B)` はジオメトリAがジオメトリBを含んでいるときにtrueを返します。
* :command:`ST_Crosses(geometry A, geometry B)` はジオメトリAがジオメトリBと交差するときにtrueを返します。
* :command:`ST_Disjoint(geometry A , geometry B)` はジオメトリ同士が空間的に交わりを持たないときにtrueを返します。
* :command:`ST_Distance(geometry A, geometry B)` はジオメトリAとジオメトリBの間の最短距離を返します。
* :command:`ST_DWithin(geometry A, geometry B, radius)` はジオメトリAがジオメトリBから特定の距離以内にあるときにtrueを返します。
* :command:`ST_Equals(geometry A, geometry B)` はジオメトリAとジオメトリBが等しいときにtrueを返します。
* :command:`ST_Intersects(geometry A, geometry B)` はジオメトリAとジオメトリBが交差するあるいは部分的に重なるときにtrueを返します。
* :command:`ST_Overlaps(geometry A, geometry B)` はジオメトリAとジオメトリBが空間を共有し、なおかつお互いがどちらかに完全に含まれることが無い場合にtrueを返します。
* :command:`ST_Touches(geometry A, geometry B)` はジオメトリAの境界部分がジオメトリBと接しているときにtureを返します。
* :command:`ST_Within(geometry A, geometry B)` はジオメトリAがジオメトリBの内部に存在するときにtureを返します。

また、各テーブルの構成は次のようになっていました:

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

 * **"'Atlantic Commons'という通りのジオメトリは何でしょうか？"**
 
   .. code-block:: sql

     SELECT the_geom
       FROM nyc_streets
       WHERE name = 'Atlantic Commons';

   ::
   
     01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141
     
 * **"Atlantic Commonsのある地域の名前と、そこの属する区は何でしょうか？"**
     
   .. code-block:: sql

     SELECT name, boroname 
     FROM nyc_neighborhoods 
     WHERE ST_Intersects(
       the_geom,
       '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141'
     );

   ::
     
          name    | boroname 
      ------------+----------
       Fort Green | Brooklyn
     

 * **"Atlantic Commonsと接する通りは何でしょうか？"** 
 
   .. code-block:: sql

     SELECT name 
     FROM nyc_streets 
     WHERE ST_Touches(
       the_geom, 
       '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141'
     );
    
   ::
  
          name      
     ---------------
      S Oxford St
      Cumberland St

   .. image:: ./spatial_relationships/atlantic_commons.jpg
  

 * **"Atlantic Commonsの近隣(50m以内の範囲)にはどれくらいの人口がいるでしょうか？"**
 
   .. code-block:: sql

     SELECT Sum(popn_total)
       FROM nyc_census_blocks
       WHERE ST_DWithin(
        the_geom,
        '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141',
        50
        );
        
   :: 
   
     1186 
   
