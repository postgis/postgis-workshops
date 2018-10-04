.. _joins:

第12章: 空間結合
=========================

空間結合と空間データベースの関係は、パンとバターのようなものと言えます。空間情報を結合キーとした空間結合を行うことで、複数の異なるテーブル間の情報を連結させることができます。私たちが考える「標準的なGIS解析」の大部分は、この空間結合によって実現することができるのです。

前節では、空間的な関連性を明らかにするのに2段階のステップを用いました。まず最初にBroad駅のポイントを抽出し、続いてそのポイントデータをもとにBroad駅の近所には何があるのかを調べました。

しかし、空間結合を使えば、そのような操作を1ステップで済ませることができます。つまり、地下鉄の駅の情報とその周辺の情報とを一度に取得することができるのです。

.. code-block:: sql

  SELECT 
    subways.name AS subway_name, 
    neighborhoods.name AS neighborhood_name, 
    neighborhoods.boroname AS borough
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_subway_stations AS subways
  ON ST_Contains(neighborhoods.the_geom, subways.the_geom)
  WHERE subways.name = 'Broad St';

:: 

   subway_name | neighborhood_name  |  borough  
  -------------+--------------------+-----------
   Broad St    | Financial District | Manhattan


すべての地下鉄駅に対して各々の近傍の情報を結びつけることはできますが、今回のケースに限ればBroad駅の情報のみで十分でした。このように、空間結合を行う関数は多数用意されていますが、主立って使用されるものは限られてきます。使用頻度の高い関数として、 :command:`ST_Intersects` 、 :command:`ST_Contains` 、 :command:`ST_DWithin` が挙げられます。

結合と要約
------------------

``JOIN`` と ``GROUP BY`` を組み合わせることで、GISシステムで行われる一般的な処理は実現できます。

例として次のような問題を考えてみましょう。 **「Manhattan地区の人口分布および人種比率はどのようになっているか？」** この問題は、国勢調査から得られる人口データからManhattan地区のものだけを抽出して考える必要があります。

.. code-block:: sql

  SELECT 
    neighborhoods.name AS neighborhood_name, 
    Sum(census.popn_total) AS population,
    Round(100.0 * Sum(census.popn_white) / Sum(census.popn_total),1) AS white_pct,
    Round(100.0 * Sum(census.popn_black) / Sum(census.popn_total),1) AS black_pct
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_census_blocks AS census
  ON ST_Intersects(neighborhoods.the_geom, census.the_geom)
  WHERE neighborhoods.boroname = 'Manhattan'
  GROUP BY neighborhoods.name
  ORDER BY white_pct DESC;

::

   neighborhood_name  | population | white_pct | black_pct 
 ---------------------+------------+-----------+-----------
  Carnegie Hill       |      19909 |      91.6 |       1.5
  North Sutton Area   |      21413 |      90.3 |       1.2
  West Village        |      27141 |      88.1 |       2.7
  Upper East Side     |     201301 |      87.8 |       2.5
  Greenwich Village   |      57047 |      84.1 |       3.3
  Soho                |      15371 |      84.1 |       3.3
  Murray Hill         |      27669 |      79.2 |       2.3
  Gramercy            |      97264 |      77.8 |       5.6
  Central Park        |      49284 |      77.8 |      10.4
  Tribeca             |      13601 |      77.2 |       5.5
  Midtown             |      70412 |      75.9 |       5.1
  Chelsea             |      51773 |      74.7 |       7.4
  Battery Park        |       9928 |      74.1 |       4.9
  Upper West Side     |     212499 |      73.3 |      10.4
  Financial District  |      17279 |      71.3 |       5.3
  Clinton             |      26347 |      64.6 |      10.3
  East Village        |      77448 |      61.4 |       9.7
  Garment District    |       6900 |      51.1 |       8.6
  Morningside Heights |      41499 |      50.2 |      24.8
  Little Italy        |      14178 |      39.4 |       1.2
  Yorkville           |      57800 |      31.2 |      33.3
  Inwood              |      50922 |      29.3 |      14.9
  Lower East Side     |     104690 |      28.3 |       9.0
  Washington Heights  |     187198 |      26.9 |      16.3
  East Harlem         |      62279 |      20.2 |      46.2
  Hamilton Heights    |      71133 |      14.6 |      41.1
  Chinatown           |      18195 |      10.3 |       4.2
  Harlem              |     125501 |       5.7 |      80.5


ここでは何が行われているのでしょう？簡単に説明すると、以下のようになります。（実際の演算結果はデータベースにより最適化されています）:

#. ``JOIN`` 句により、近傍情報と調査内容の両方のデータを含む仮想的なテーブルを作成する。
#. ``WHERE`` 句により、Manhattan地区のレコードのみを抽出する。
#. 抽出したレコードを'neighborhood_name'単位でグルーピングし、 :command:`Sum()` で得られる人口の総数を出力する。
#. データの出力フォーマットを整えたり (例えば、 ``GROUP BY`` や ``ORDER BY`` などを使って)、演算加工を施して、最終的に目的とする値をパーセンテージで出力する。 

.. 補足:: 

   ``JOIN``句は、``FROM``に指定された2つの項目を連結する役割を持ちます。通常我々は``INNER JOIN``を用いますが、JOIN句には4種類のタイプが存在します。より詳細な情報は、PostgreSQLドキュメントに書かれた`join_type <http://www.postgresql.org/docs/8.1/interactive/sql-select.html>`を参照してください。

距離を測定する関数を ``JOIN`` 句のキーに使うこともできます。これにより、"ある半径以内に含まれるアイテム"を取得することが可能です。距離関数を使って、New Yorkの人種分布を探ってみましょう。

手始めに、New Yorkの人種比率を算出してみます。

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks;

:: 

        white_pct      |      black_pct      | popn_total 
  ---------------------+---------------------+------------
   44.6586020115685295 | 26.5945063345703034 |    8008278


New Yorkにいる800万人の住民のうち、約44%が"白人"で、26%が"黒人"であることが分かります。

黒人アーティストのの Duke Ellington はかつてこのように歌っていました。"ハーレムのシュガー・ヒル方面に行くのなら、北方面行きA列車にお乗りなさい"。 確かに、先の結果も示しているように、Harlem は Manhattan の中でも最も高い黒人人口(80.5%)を持ちます。この傾向は、Duke の歌ったA列車の全路線に当てはまるのでしょうか？

最初に、nyc_subway_stationsテーブルのroutes列にA列車の情報が含まれていることに注目してください。この列の値は若干複雑です。

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

.. 補足::

   ``DISTINCT`` キーワードは、重複したレコードを結果から除外します。 ``DISTINCT`` キーワードを使わなかった場合、上記のクエリから返るレコード数は73から491に増加します。
   
A列車を探すために、 ``routes`` 列の中で'A'という文字を含んだレコードを抽出します。これには様々な方法がありますが、今回は :command:`strpos(routes,'A')` を利用します。文字'A'が含まれている場合、コマンドの結果は0より大きい値を返します。

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
  
A列車の路線から200m以内の範囲で、人種比率を算出してみましょう。

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks AS census
  JOIN nyc_subway_stations AS subways
  ON ST_DWithin(census.the_geom, subways.the_geom, 200)
  WHERE strpos(subways.routes,'A') > 0;

::

        white_pct      |      black_pct      | popn_total 
  ---------------------+---------------------+------------
   42.0805466940877366 | 23.0936148851067964 |     185259

この結果から、A列車沿いの人種比率はNew Yorkと大差ないことが分かります。

より高度な結合
-------------

A列車が都市の人種比率に影響を与えていないことを見てきました。他の路線だとどうでしょうか？比率が大きく異なるものはあるのでしょうか？

この問いに答えるために、先のクエリにもう一つのJOIN句を追加し、複数の路線沿いの人種比率を一度に求めます。これには、すべての路線をリストアップするあらたなテーブルを作る必要があります。

.. code-block:: sql

    CREATE TABLE subway_lines ( route char(1) );
    INSERT INTO subway_lines (route) VALUES 
      ('A'),('B'),('C'),('D'),('E'),('F'),('G'),
      ('J'),('L'),('M'),('N'),('Q'),('R'),('S'),
      ('Z'),('1'),('2'),('3'),('4'),('5'),('6'),
      ('7');

ここで、このテーブルを、先で使用したクエリに連結します。

.. code-block:: sql

    SELECT 
      lines.route,
      Round(100.0 * Sum(popn_white) / Sum(popn_total), 1) AS white_pct, 
      Round(100.0 * Sum(popn_black) / Sum(popn_total), 1) AS black_pct, 
      Sum(popn_total) AS popn_total
    FROM nyc_census_blocks AS census
    JOIN nyc_subway_stations AS subways
    ON ST_DWithin(census.the_geom, subways.the_geom, 200)
    JOIN subway_lines AS lines
    ON strpos(subways.routes, lines.route) > 0
    GROUP BY lines.route
    ORDER BY black_pct DESC;

::

     route | white_pct | black_pct | popn_total 
    -------+-----------+-----------+------------
     S     |      30.1 |      59.5 |      32730
     3     |      34.3 |      51.8 |     201888
     2     |      33.6 |      45.5 |     535414
     5     |      32.1 |      45.1 |     407324
     C     |      41.3 |      35.9 |     430194
     4     |      34.7 |      30.9 |     328292
     B     |      36.1 |      30.6 |     261186
     Q     |      52.9 |      26.3 |     259820
     J     |      29.5 |      23.6 |     126764
     A     |      42.1 |      23.1 |     370518
     Z     |      29.5 |      21.5 |      81493
     D     |      39.8 |      20.9 |     233855
     G     |      44.8 |      20.0 |     138602
     L     |      53.9 |      17.1 |     104140
     6     |      52.7 |      16.3 |     257769
     1     |      54.8 |      12.6 |     659028
     F     |      60.0 |       8.6 |     438212
     M     |      50.0 |       7.8 |     166721
     E     |      69.4 |       5.3 |      86118
     R     |      57.7 |       4.8 |     389124
     7     |      42.4 |       3.8 |     107543


既に見てきたように、JOIN句は ``JOIN ON`` で指定した条件に合致するすべての組み合わせで仮想的なテーブルを作成し、それらのレコードをグループ単位でまとめて出力します。これを絞り込むコツは ``ST_DWithin`` を使うことです。 ``ST_DWithin`` により、地下鉄駅から一定の距離以内にあるデータだけが計算対象となっています。

関数一覧
-------------

`ST_Contains(geometry A, geometry B) <http://postgis.net/docs/manual-2.0/ST_Contains.html>`_: Bのどの部分もAの外に属しておらず、かつBの中の1点だけでもAの中に含まれていれば true を返します。

`ST_DWithin(geometry A, geometry B, radius) <http://postgis.net/docs/manual-2.0/ST_DWithin.html>`_: 両方のジオメトリーが指定した距離以内であれば true を返します。

`ST_Intersects(geometry A, geometry B) <http://postgis.net/docs/manual-2.0/ST_Intersects.html>`_: ジオメトリーが空間的に交わっている（すなわち、ジオメトリーの一部を共有している）場合に true を返し、まったく交わりがない場合に false を返します。

`round(v numeric, s integer) <http://www.postgresql.org/docs/7.4/interactive/functions-math.html>`_: 四捨五入して整数にした値を返すPostgreSQLの数学関数です。

`strpos(string, substring) <http://www.postgresql.org/docs/current/static/functions-string.html>`_: 指定した文字列を検索し、それが含まれる位置を整数で返すPostgreSQLの文字列関数です。

`sum(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: レコードの合計値を返すPostgreSQLの関数です。

.. rubric:: 脚注

.. [#PostGIS_Doco] http://postgis.net/docs/manual-2.0/

