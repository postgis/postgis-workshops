.. _simple_sql_exercises:

第7章: シンプルなSQLの演習
===============================

``nyc_census_blocks`` を使って以下の問いに答えてください。（答えがわかっても声に出しちゃだめですよ。）

演習を始める前に、役に立つ情報を整理しておきましょう。
まずは`データについて<about_data>`の章の``nyc_census_blocks``テーブルの定義です。

.. list-table::
   :widths: 20 80

   * - **blkid**
     - すべての統計**block**を一意に特定するための15桁のコード 例：360050001009000
   * - **popn_total**
     - その統計ブロックの合計人口
   * - **popn_white**
     - その統計ブロックの「白人」と自己認識する人口
   * - **popn_black**
     - その統計ブロックの「黒人」と自己認識する人口
   * - **popn_nativ**
     - その統計ブロックの「アメリカ原住民」と自己認識する人口
   * - **popn_asian**
     - その統計ブロックの「アジア民族」と自己認識する人口
   * - **popn_other**
     - その統計ブロックのその他の民族と自己認識する人口
   * - **hous_total**
     - その統計ブロックの世帯数
   * - **hous_own**
     - その統計ブロックの持ち家数
   * - **hous_rent**
     - その統計ブロックの借家数
   * - **boroname**
     - ニューヨークの区名（Manhattan, The Bronx, Brooklyn, Staten Island, Queens）
   * - **the_geom**
     - その統計ブロックのポリゴン境界


もうひとつは、よく使われる便利な集約関数です。

 * avg() - レコードの値の平均値を返します
 * sum() - レコードの値の合計値を返します
 * count() - レコードの数を返します

さて問題です。

 * **"ニューヨーク市の人口は何人でしょうか？"**

   .. code-block:: sql
   
     SELECT Sum(popn_total) AS population
       FROM nyc_census_blocks;
     
   :: 
   
     8008278 
   
   .. note:: 
   
   この ``AS`` とは何でしょうか。これはテーブルやフィールド名の別名で、自由に付けることが出来ます。別名は、問い合わせをしやすくしたり、わかりやすくするために用いられます。この例では出力項目の名称を、 ``sum`` の代わりに、 **AS** 句を使って ``population`` というわかりやすい名称に置き換えています。
       
 * **"ブロンクス区（The Bronx）の人口は何人でしょうか？"**

   .. code-block:: sql
 
     SELECT Sum(popn_total) AS population
       FROM nyc_census_blocks
       WHERE boroname = 'The Bronx';
     
   :: 
   
     1332650 
   
 * **"ニューヨーク市の世帯当たりの家族数の平均は何人でしょうか？"**
 
   .. code-block:: sql

     SELECT Sum(popn_total)/Sum(hous_total) AS popn_per_house
       FROM nyc_census_blocks;

   :: 
   
     2.6503540522400804 
   
 * **"各区の白人の割合は何％でしょうか？"**

   .. code-block:: sql

     SELECT 
         boroname, 
         100 * Sum(popn_white)/Sum(popn_total) AS white_pct
       FROM nyc_census_blocks
       GROUP BY boroname;

   :: 
   
        boroname    |      white_pct      
     ---------------+---------------------
      Brooklyn      | 41.2005552206888663
      The Bronx     | 29.8655310846808990
      Manhattan     | 54.3594013771837665
      Queens        | 44.0806610271290794
      Staten Island | 77.5968611401579346
 
関数一覧
-------------

`avg(expression) <http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: PostgreSQL集約関数：数値フィールドの平均値を返します

`count(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: PostgreSQL集約関数：レコードの数を返します

`sum(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: PostgreSQL集約関数：レコードの値の合計値を返します
