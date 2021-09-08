---
title: "プラグインを作成する"
linkTitle: "プラグインを作成する"
---

現状のCeforeの「プラグインシステム」ではいろんなところに書き込まねばならないので、 plug-in というより soldering に近いです。

まずはTransportプラグインの場合の変更箇所を示します。具体的な変更は samptp 等を参考にしてください。

プラグインの名前を *foo* とします。

- src/plugin/transport/*foo* / 以下
    - *foo_plugin*.c: プラグイン実装本体。以下のcallbackを（必要に応じて）実装する。
        - `init` : 初期化。返り値は使われないのでなんでもよい。
        - `destroy`: 後片付け。initと違って返り値は void。
        - `cob` : Objectが到着したとき（Objectって何？）
            - 返り値はint型。`CefC_Pi_Object_Send` を返すと転送を続け、 `CefC_Pi_Object_NoSend` を返すと破棄する。
        - `interest` : Interestが到着したとき
            - 返り値はint型。`CefC_Pi_Interest_Send` を返すと転送を続け、 `CefC_Pi_Interest_NoSend` を返すと破棄する。
            - また、他にも`CefC_Pi_Object_Match`などを返せる（挙動は cef_netd.c 参照）。
        - `delpit` : PITエントリが削除されたとき
- src/include/ 以下
    - *foo_plugin*.h: 上の .c に対応するもの。デフォルトのプラグインのプロトタイプ宣言などは cef_plugin_com.h にまとめられているから、これに追記しても良い。
    - cef_plugin.h
        - `CefC_T_OPT_TP_foo`マクロを設定する。値は適当で良いが、他の Transport プラグインと被らないようにする。
            - `CefC_T_OPT_TP_*` マクロは、Ceforeが使用するoptionalのTLVフィールドで使われる値（ということになっている）。
                - あるいはプラグインが併用可能と考えれば、ORでつなげて表現する想定もありうる （要検証）
                    - プラグインは併用可能ではない？
        - `CefC_T_OPT_TP_NUM` マクロの値も1足しておく。このマクロは Transport プラグインの登録された総数を表している。
- src/plugin 以下
    - cef_tp_plugin.c
        - `cef_tp_plugin_init` 関数で、プラグインの登録と初期化を行う。
        - `cef_tp_plugin_destroy`関数で、プラグインの後処理をする関数を呼び出す（init同様勝手には呼び出してくれない）。
- ビルド関連
    - [Ceforeチュートリアル](https://www.ieice.org/~icn/wp-content/uploads/2017/08/Cefore-tutorial.pdf)のp23, 24が詳しい
    - configure.ac: 以下のMakefile.amで使う `VIS_ENABLE` 変数を設定する
    - Makefile.am: `DISTCHECK_CONFIGURE_FLAGS` に追記
    - src/include/cefore/Makefile.am: プラグインのヘッダファイルを追加
        - cef_plugin_com.h に追記した場合は変更不要
    - src/plugin/Makefile.am: ビルド時のフラグを追加、ビルド対象にCソースを追加
    - 書き終わったら `$ autoreconf -f -i` してからいつもどおりビルドする