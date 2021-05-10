.. _postgis-functions:

付録 A: PostGISの関数
=============================

コンストラクタ
------------

:command:`ST_MakePoint(Longitude, Latitude)` 
  新しいジオメトリポイントを生成します。座標 (経度, 緯度)の順序に注意して下さい。

:command:`ST_GeomFromText(WellKnownText, srid)`
 Well-Known Text表現(WKT)とSRIDから新しいST_Geometryを返します。

:command:`ST_SetSRID(geometry, srid)`
  ジオメトリのSRIDを更新します。同じジオメトリを返します。これは、ジオメトリは変更せず、ジオメトリのSRIDだけを更新します。

:command:`ST_Expand(geometry, Radius)`
  入力ジオメトリのバウンディングボックスから全ての方向に拡張されたバウンディングボックスを返します。インデックス検索で使用するために範囲を作成する場合に便利です。

出力
-------

:command:`ST_AsText(geometry)`
  ジオメトリ/ジオグラフィのSRIDメタデータのないWell-Known Text(WKT)表現を返します。

:command:`ST_AsGML(geometry)`
  OGC標準のGeography Markup Language (GML)要素としてジオメトリを返します。

:command:`ST_AsGeoJSON(geometry)`
  OGC標準のGeometry Javascript Object Notation (GeoJSON)要素としてジオメトリを返します。 `GeoJSON <http://geojson.org>`_ フォーマット。

測定
------------

:command:`ST_Area(geometry)`
  空間参照系の単位でジオメトリの面積を返します。

:command:`ST_Length(geometry)`
  空間参照系の単位でジオメトリの長さを返します。

:command:`ST_Perimeter(geometry)`
  空間参照系の単位でジオメトリの境界の長さの計測値を返します。

:command:`ST_NumPoints(linestring)`
  ラインストリング内のポイント数を返します。

:command:`ST_NumRings(polygon)`
  ポリゴンのリング数を返します。

:command:`ST_NumGeometries(geometry)` 
  ジオメトリコレクション内のジオメトリの数を返します。

関係
-------------

:command:`ST_Distance(geometry, geometry)`
  空間参照系の単位の2つのジオメトリ間の距離を返します。

:command:`ST_DWithin(geometry, geometry, radius)` 
  ジオメトリが、指定したジオメトリから指定した距離内にある場合に、TRUEを返します。そうでない場合は、FALSEを返します。

:command:`ST_Intersects(geometry, geometry)`
  ジオメトリが、「空間的にインタセクトする」(空間に共有部分がある)場合には、TRUEを返します。そうでない場合は、FALSEを返します。

:command:`ST_Contains(geometry, geometry)`
  もし、最初のジオメトリが完全に次のジオメトリに含まれている場合は、TRUEを返します。そうでない場合は、FALSEを返します。

:command:`ST_Crosses(geometry, geometry)`
  ライン、またはポリゴンの境界が、他のライン、またはポリゴンの境界とクロスする場合は、TRUEを返します。そうでない場合は、FALSEを返します。
