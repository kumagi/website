Burst
-----

* 詳細な仕様は `IDL 定義 <https://github.com/jubatus/jubatus/blob/master/jubatus/server/server/burst.idl>`_ を参照してください。
* 使用されているアルゴリズムの詳細については :doc:`method` を参照してください。

Configuration
~~~~~~~~~~~~~

設定は単体の JSON で与えられる。
JSON の各フィールドは以下の通りである。

.. describe:: parameter

  バースト検知に渡すパラメータを指定する。

  :window_batch_size:
    バースト検知に用いるウィンドウのサイズを指定する。

    * 値域: 0 < ``window_batch_size``

  :batch_interval:
    

    * 値域: 0 < ``batch_interval``

  :result_window_rotate_size:

    * 値域: 0 < ``result_window_rotate_size``

  :max_reuse_batch_num:

    * 値域: 0 < ``max_reuse_batch_num``

  :costcut_threshold:

    負の数を指定した場合には無限大として作用します。

Data Structures
~~~~~~~~~~~~~~~


.. mpidl:message:: st_keyword

   バーストを検知する対象とその閾値を表す

   .. mpidl:member:: 0: string txt

      バーストを検知したいキーワードを表す

   .. mpidl:member:: 1: double scaling_param

      このキーワードの検知に用いるスケーリングパラメータを表す

   .. mpidl:member:: 2: double gamma

      このキーワードのバースト検知に適用するγ値を表す

.. mpidl:message:: st_batch

   ウィンドウを構成するバッチの構造

   .. mpidl:member:: 0: int d

      バッチに入る全データ数を表す

   .. mpidl:member:: 1: int r

      バッチに入るキーワード数を表す

   .. mpidl:member:: 2: double batch_weight

      バースト検知の結果を表す

.. mpidl:message:: st_window

   バースト検知を実行するウィンドウ

   .. mpidl:message:: 0: int start_pos

      このウィンドウの開始位置を表す

   .. mpidl:message:: 1: list<st_batch> batches

      このウィンドウを構成するバッチの集合を表す

Methods
~~~~~~~


.. mpidl:service:: burst

  # 学習データとしてのドキュメントを追加する
  # 引数
  # name：クラスタ識別名
  # data：入力するデータ。tupleの1番目には入力の位置（一般に時間）、2番目には入力するデータ
  # 戻り値
  # 全サーバでの関数実行成否(bool)のand
  #@broadcast #@update #@all_and

  .. mpidl:method:: bool add_documents(0: string name, 1: list<tuple<double, datum> > data)

  

  # 最新のバースト検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # keyword_txt：対象とするキーワード
  # 戻り値
  # バースト検知結果（バッチweight）が入ったウィンドウ
  # 備考
  # 「最新」とは一番最後にバースト検知されたウィンドウを意味する
  #@cht #@analysis #@pass
  st_window get_result(0: string name, 1: string keyword_txt) # //@cht

  # 最新のバースト検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # keyword_txt：対象とするキーワード
  # 戻り値
  # バースト検知結果（バッチweight）が入ったウィンドウ
  # 備考
  # get_result との違いは「get_result」はコンシステントハッシュでキーワードを担当しているサーバに問い合わせるが、～_by_all はキーワードを担当しているかどうかに関係になく、全サーバに問い合わせる。@paそして、全サーバから戻ってきた中からランダムに選んでクライアントに返す。
  #@broadcast #@analysis #@pass
  st_window get_result_by_all(0: string name, 1: string keyword_txt) # //@broadcast

  # 指定した位置（時間）の検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # keyword_txt：対象とするキーワード
  # pos：指定位置
  # 戻り値
  # バースト検知結果（バッチweight）が入ったウィンドウ
  # 備考
  # 指定した位置を含むウィンドウを検索し、そのウィンドウを返す。
  #@cht #@analysis #@pass
  st_window get_result_at(0: string name, 1: string keyword_txt, 2: double pos) # //@cht

  # 指定した位置（時間）の検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # keyword_txt：対象とするキーワード
  # pos：指定位置
  # 戻り値
  # バースト検知結果（バッチweight）が入ったウィンドウ
  # 備考
  # get_result_at との違いについては、get_result_by_all の備考を参照。
#@broadcast #@analysis #@pass
  st_window get_result_by_all_at(0: string name, 1: string keyword_txt, 2: double pos) # //@broadcast

  # 現在（＝最新のバッチ）がバーストしている全キーワードのバースト検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # 戻り値
  # バースト検知結果が入ったウィンドウの集合。mapの1番目はキーワード、2番目はその
  # キーワードのバースト検知結果が入ったウィンドウ
  #@broadcast #@analysis #@merge
  map<string, st_window > get_all_bursted_results(0: string name) # //@broadcast

  # 指定した位置がバーストしている全キーワードのバースト検知結果を取得する
  # 引数
  # name：クラスタ識別名
  # pos：指定位置
  # 戻り値
  # バースト検知結果が入ったウィンドウの集合。mapの1番目はキーワード、2番目はその
  # キーワードのバースト検知結果が入ったウィンドウ
  #@broadcast #@analysis #@merge
  map<string, st_window > get_all_bursted_results_at(0: string name, 1: double pos) # //@broadcast

　# 現在登録されている全キーワードリストを取得する
  # 引数
  # name：クラスタ識別名
  # 戻り値：パラメータを含めた（全）キーワードリスト
  #@random #@analysis #@pass
  list<st_keyword> get_all_keywords(0: string name) # //@random

　# キーワードを追加する
  # 引数
  # name：クラスタ識別名
  # keyword：パラメータを含めたキーワード情報
  # 戻り値
  # 全サーバでの関数実行成否(bool)のand
  #@broadcast #@update #@all_and
  bool add_keyword(0: string name, 1: st_keyword keyword) # //@broadcast

  # 指定したキーワードを削除する（バースト検知情報
  # 引数
  # name：クラスタ識別名
  # keyword_txt：削除するキーワード
  # 戻り値
  # 全サーバでの関数実行成否(bool)のand
  #@broadcast #@update #@all_and
  bool remove_keyword(0: string name, 1: string keyword_txt) # //@broadcast

  # 全キーワードを削除する
  # 引数
  # name：クラスタ識別名
  # 戻り値
  # 全サーバでの関数実行成否(bool)のand
  #@broadcast #@update #@all_and
  bool remove_all_keywords(0: string name) # //@broadcast

# 他のモジュールの get_config と同じなので説明省略
  #@broadcast #@analysis #@merge
  map<string, map<string, string> >  get_status(0: string name) # //@broadcast
