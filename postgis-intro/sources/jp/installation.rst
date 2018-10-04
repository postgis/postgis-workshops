.. _installation:

第2章: インストール
=======================

まず最初にOpenGeoスイートのインストールを行います。OpenGeoスイートは、PostGIS/PostgreSQLに加えGeoServer、OpenLayersやその他様々なWebツールを含んだソフトウェアパッケージであり、Windows、Apple OS/X、Linux向けにインストーラが提供されています。
.. note::

 PostgreSQLだけをインストールしたい場合には、PostgreSQLプロジェクトサイト( http://postgresql.org/download/ )から直接ソースコードやバイナリをダウンロードすることができます。PostgreSQLインストール後に、"StackBuilder"ユーティリティーを使ってPostGISエクステンションを追加してください。

.. note::

 この文書はWindows向けに書かれていますが、OS/Xでも大きな違いはありません。OpenGeoスイートのインストール後は、どちらのOSでも同じような操作となります。

#. ディレクトリ :file:`postgisintro\\software\\` の配下に :file:`opengeosuite-2.1.3.exe` (OS/Xの場合は, :file:`opengeosuite-2.1.3.dmg`)というファイル名のインストーラがあります。これをダブルクリックして実行してください。

#. OpenGeoからの心のこもったメッセージが表示されます。一読した後に **Next** ボタンをクリックしてください。

   .. image:: ./screenshots/install_01.png


#. OpenGeoスイートはGNU GPLライセンスのもとで提供されています。規約に同意されたら **I Agree** ボタンを押してください。

   .. image:: ./screenshots/install_02.png


#. OpenGeoスイートのインストール先は通常 ``C:\Program Files\`` 以下となります。データの配置場所はホームディレクトリの :file:`.opengeo` となります。指定したら **Next** ボタンを押してください。

   .. image:: ./screenshots/install_03.png


#. スタートメニューの中にショートカットを作成します。指定したら **Next**ボタン を押してください。

   .. image:: ./screenshots/install_04.png


#. OpenGeoスイートのすべてのコンポーネントが必須となります。すべて選択したうえで**Next**ボタンを押してください。

   .. image:: ./screenshots/install_05.png


#. インストールの準備が整いました。 **Install** ボタンを押してください。

   .. image:: ./screenshots/install_06.png


#. インストールが実行されます。所要時間は数分程度です。

   .. image:: ./screenshots/install_07.png


#. インストールが完了したら、ワークショップの次のステップに進むためにダッシュボードを起動してください。 **Finish** ボタンを押して、インストールは完了です。

   .. image:: ./screenshots/install_08.png


