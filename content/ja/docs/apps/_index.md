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

TODO: FFI

### ビルド
libcrypto (OpenSSL) とlibcefore (cefore)をリンクします。

```shell
$ gcc example.c -lcefore -lcrypto -o example
```

### 実装例
ceforeソースコードの`tools/`以下のツール群の実装が参考になります。

ここでは、コマンドライン引数に与えられたURIにInterestを発行し、応答を`stdout`に出力する実装`easygetfile.c`を示します。パフォーマンスの計測などを行わないため、`cefgetfile`と比べて簡潔なコードになっています。

```c:easygetfile.c
#include <stdio.h>
#include <stdlib.h>

#include <cefore/cef_client.h>
#include <cefore/cef_frame.h>

CefT_Client_Handle setup_client() {
  char config_path[1] = {0};
  int res = cef_client_init(CefC_Unset_Port, config_path);
  if (res < 0) {
    fprintf(stderr, "Could not init the client\n");
    exit(-1);
  }

  CefT_Client_Handle fhdl = cef_client_connect();
  if (fhdl < 1) {
    fprintf(stderr, "Could not connect to cefnetd. Is cefnetd running?\n");
    exit(-1);
  }

  return fhdl;
}

CefT_Interest_TLVs generate_params(char *uri) {
  CefT_Interest_TLVs params;
  memset(&params, 0, sizeof(CefT_Interest_TLVs));

  cef_frame_init();

  // Convert URI (ccnx:/whatever) to name (TLV)
  int name_len = cef_frame_conversion_uri_to_name(uri, params.name);
  if (name_len < 0) {
    fprintf(stderr, "The URI is invalid. \n");
    exit(-1);
  }
  fprintf(stderr, "URI: %s\n", uri);
  params.name_len = name_len;

  // Other parameters (explicitly setting all the flags for guide)
  params.hoplimit = 32;
  params.chunk_num_f = 1;
  params.chunk_num = 0;
  params.nonce_f = 0;
  params.symbolic_code_f = 0;

  params.opt.symbolic_f = CefC_T_OPT_REGULAR;
  params.opt.lifetime_f = 0;

  return params;
}

void *send_interest(CefT_Client_Handle fhdl, CefT_Interest_TLVs *params) {
  cef_client_interest_input(fhdl, params);
  fprintf(stderr, "Reading Chunk %d....\n", params->chunk_num);

  void *payload = calloc(CefC_AppBuff_Size, 1);
  unsigned char *buff = malloc(CefC_AppBuff_Size);
  struct cef_app_frame *app_frame = malloc(sizeof(struct cef_app_frame));

  int position = 0;
  int payload_len = 0;
  int end_chunk_num = INT32_MAX;
  while (1) {
    int size =
        cef_client_read(fhdl, &buff[position], CefC_AppBuff_Size - position);

    if (size > 0) {
      int remaining = cef_client_payload_get_with_info(
          &buff[position], CefC_AppBuff_Size - position, app_frame);
      position = CefC_AppBuff_Size - remaining;
      memcpy((void *)(payload + payload_len), app_frame->payload,
             app_frame->payload_len);
      payload_len += app_frame->payload_len;
      end_chunk_num = app_frame->end_chunk_num;

      fprintf(stderr, "Current position: %d\n", position);
      fprintf(stderr, "Total Payload Length: %d\n", payload_len);
    } else if (params->chunk_num == end_chunk_num) {
      break;
    } else {
      params->chunk_num++;
      cef_client_interest_input(fhdl, params);
      fprintf(stderr, "Reading Chunk %d....\n", params->chunk_num);
    }
  }

  return payload;
}

int main(int argc, char **argv) {
  if (argc <= 1) {
    fprintf(stderr, "No prefix given. Usage: ./easygetfile ccnx:/example\n");
    return -1;
  }

  char uri[256];
  strcpy(uri, argv[1]);

  CefT_Client_Handle fhdl = setup_client();
  CefT_Interest_TLVs params = generate_params(uri);

  void *payload = send_interest(fhdl, &params);

  printf("%s\n", (char *)payload);

  cef_client_close(fhdl);

  return 0;
}
```

以下のコマンドでビルドと動作確認ができます。
```shell
$ gcc easygetfile.c -lcefore -lcrypto -o easygetfile
$ ./easygetfile ccnx:/example
```

## Pythonを使う

**cefpyco**を使えば、Pythonでアプリを作成できます。Dockerイメージにはデフォルトでインストールされているので、すぐに使えます。

{{% alert color="info" %}}

[Ceforeアプリ開発ツール cefpyco](https://www.ieice.org/~icn/wp-content/uploads/2018/08/hands_on_02_Cefpyco.pdf)（公式資料）も参照してください。

{{% /alert %}}

TODO: なんか簡単な例