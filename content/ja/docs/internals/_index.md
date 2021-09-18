---
title: "内部実装"
linkTitle: "内部実装"
weight: 8
date: 2017-01-04
description: ''

---

ソースリーディングとか。

## ディレクトリ構成

### `src/`

cefnetdやcsmgrdのソースコードがあります。

### `tools/`

各種コマンドのうち、C言語で実装されているものが入っています。

### `utils/`

各種コマンドのうち、シェルスクリプトで実装されている比較的簡素なものが入っています。

### `config/`

設定ファイルのテンプレートが入っています。

### `apps/wireshark/`

WiresharkでCCNのパケットを読むためのluaスクリプトが入っています。詳しくは[Wireshark](../tools/wireshark)を参照してください。
