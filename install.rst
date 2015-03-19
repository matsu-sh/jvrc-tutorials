インストール
============

本ドキュメントでは Choreonoid とモデルファイルのインストール方法について解説します。
インストールを実行するのは Ubuntu 14.04 x64 版とします。


Choreonoidのインストール
------------------------

端末上で次のコマンドを実行し、Choreonoid と周辺ツールをインストールします。 ::

 sudo add-apt-repository ppa:hrg/daily
 sudo apt-get update
 sudo apt-get install choreonoid openrtm-aist


モデルファイルのインストール
----------------------------

モデルファイルはgithub上で公開されています。

  https://github.com/jvrc/model

このリポジトリの利用にあたってはgitコマンドが必要です。Ubuntu 環境では以下のコマンドでgitをインストールできます。 ::

 sudo apt-get install git

モデルファイルのリポジトリは以下のコマンドを実行することで取得できます。 ::

 git clone https://github.com/jvrc/model

これによってリポジトリを格納した "model" というディレクトリが生成されます。

