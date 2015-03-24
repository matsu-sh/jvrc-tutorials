RTコンポーネントにおけるトルク指令の入力
========================================


ここでは、トルク指令を出力するようにRTコンポーネントを拡張し、ロボットが直立を
維持できるようにします。

プロジェクトを開く
------------------

「メニュー」の「プロジェクトの読み込み」から JVRC モデルファイル用のプロジェク
トファイルを読み込みます。プロジェクトファイル名は「モデルファイルのインス
トール」でダウンロードしたリポジトリの「model/robot/samples/sample2.cnoid」です。

コントローラのソースコード
--------------------------

コントローラのソースコードは以下になります。Choreonoid の
SR1WalkControllerRTC.cpp を基にしています。 ::



コントローラのヘッダのソースコードは以下になります。Choreonoid の
SR1WalkControllerRTC.h を基にしています。 ::


これらのソースコードは 「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/RobotTorqueControllerRTC.cpp」と 「model/robot/RTC/RobotTorqueControllerRTC.h」に保存されています。

コントローラの設定
------------------

アイテムビューで「BodyRTC」を選択し、プロパティビューの「コントローラのモジュール名」を「RobotTorqueControllerRTC」とします。これは「コントローラのビルド」で作成したモジュールのパスと対応しています。
さらに、プロパティビューの「自動ポート接続」を true にします。

.. image:: images/property_torque.png

ポーズ列の追加
--------------

まずアイテムビューで「JVRC」を選択します。
次に、「メニュー」の「ファイル」「新規」より「ポーズ列」を選択し「SampleMotion」という名前で追加します。

.. image:: images/motion.png

次に、「表示」の「ビューの表示」から「ポーズロール」を選択します。次の画面が表示されるはずです。

.. image:: images/pose_role.png

ポーズロールにおいて、1.0 を選択して「挿入」を押します。
同様に 2.0, 3.0, 4.0 を選択して「挿入」を押します。

ポーズロールは次のようになるはずです。

.. image:: images/pose_role2.png


プログラムで使用するモーションを生成させます。
ツールバーから「ボディモーションの生成」ボタンを押します。

.. image:: images/motion_toolbar.png

SampleMotion の子供に motion があるので、これを選択し名前を付けて保存ボタンを押します。

.. image:: images/item_motion.png

「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/」ディレクトリに「RobotMotion.yaml」というファイルで保存します。

コントローラのビルド
--------------------

「モデルファイルのインストール」でダウンロードしたリポジトリの「model/robot/RTC/」ディレクトリに移動し、make コマンドを実行します。

「model/robot/RTC/」ディレクトリに「RobotTorqueControllerRTC.so」というファイルが作成されるはずです。

その後、次のコマンドを実行します。 ::

   sudo make install DESTDIR=/usr


シミュレーションを実行する
--------------------------

シミュレーションツールバーの「シミュレーション開始ボタン」を押します。
シミュレーションを実行するとロボットが崩れ落ちず、立ったままの状態になったはずです。

.. image:: images/simulation_no_controller.png


サンプルプロジェクトについて
----------------------------

このサンプルのプロジェクトファイルは「モデルファイルのインストール」でダウン ロードしたリポジトリの「model/robot/samples/sample3.cnoid」に保存されています。

.. toctree::
   :maxdepth: 2

