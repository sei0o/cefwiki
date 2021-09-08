---
title: "プラグイン"
linkTitle: "プラグイン"
weight: 9
description: >
  プラグインを作ったり動かしたりする方法を紹介します。
---

アプリとは異なり、Cefore自身の機能を拡張するのが**プラグイン**です。現状はceforeのソースコードに組み込む形になっており、C言語が必須です。

プラグインには、cefnetdに適用されるものとcsmgrdに適用されるものがあります。

## cefnetdに適用されるプラグイン

src/plugin 以下。

- Transport プラグイン
    - 例として samptp が実装されている
    - L4C2 の記述もあるが、これは未実装
- NDN プラグイン
    - 例として ndn_v050 が実装されている
- Mobility プラグイン (未実装)
- EFI プラグイン (未実装)

## csmgrdに適用されるプラグイン

- Cache プラグイン: src/**csmgrd**/plugin/lib/ 以下
    - デフォルトでは fifo, lru, lfu が実装されている
    - [frequency-filter](https://github.com/ICN2020/frequency-filter) など
- 保管場所を指定するfilesystem_cache, mem_cacheも Cache Plugin と呼ばれている
    - それぞれの `csmgrd_*_plugin_load` 関数がcsmgrd本体から呼ばれ、 `CSMGRD_SET_CALLBACKS` マクロで諸々を設定する
    - それぞれの保管場所のプラグインで、入れ替えアルゴリズムを実装したプラグインを実行する

## internals

プラグインという機構の実装なのか、その上で動作するプラグイン自身の実装なのか混乱しないように読み進める。

- src/plugin/
    - transport/samptp/
        - cef_plugin_samptp.c:
- src/csmgrd
    - plugin/
        - mem_cache.c
            - mem_cs_create関数: 初期化。入れ替えアルゴリズムのプラグインも読み込む。
- src/include/
    - cef_plugin.h: それぞれの種類のプラグインで呼ばれる関数が CefT_Plugin_* 構造体にかかれている
    -