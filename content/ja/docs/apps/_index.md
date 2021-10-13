---
title: "プログラムで操作する"
linkTitle: "プログラムで操作する"
weight: 3
date: 2017-01-05
description: >
  API・ライブラリの使い方を解説します。
---

CeforeはC言語とPythonのそれぞれから操作することができます。このようにCeforeを操作するプログラムのことを**アプリ**と呼びます。Cefore自身の機能を拡張するには、[プラグイン](/docs/plugins)を使う必要があります。

## C言語を使う

関連するヘッダファイルをincludeすれば使い始められます。`include/cefore/cef_client.h`などに呼び出せる関数が定義されています。

{{% alert color="info" %}}

[Ceforeアプリ開発ツール cefpyco](https://www.ieice.org/~icn/wp-content/uploads/2018/08/hands_on_02_Cefpyco.pdf)（公式資料）の24ページ以降に主要なC言語用の関数と構造体がまとめられています。

{{% /alert %}}

TODO: simpleなcefgetfile, cefputfileの実装？

```c
#include <cefore/cef_define.h>
#include <cefore/cef_frame.h>
#include <cefore/cef_client.h> 
#include <cefore/cef_log.h> // ログ関連

int main(int argc, char **argv) {
    cef_frame_init();
    int result = cef_client_init(port, config_path);
    CefT_Client_Handle handler = cef_client_connect();

    ...(処理)...

    cef_client_close(handler);
    return 0;
}
```

TODO: FFI

## Pythonを使う

**cefpyco**を使えば、Pythonでアプリを作成できます。Dockerイメージにはデフォルトでインストールされているので、すぐに使えます。

{{% alert color="info" %}}

[Ceforeアプリ開発ツール cefpyco](https://www.ieice.org/~icn/wp-content/uploads/2018/08/hands_on_02_Cefpyco.pdf)（公式資料）も参照してください。

{{% /alert %}}

TODO: なんか簡単な例