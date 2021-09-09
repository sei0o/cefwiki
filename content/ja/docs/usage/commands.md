---
title: "コマンド"
linkTitle: "コマンド"
weight: 3
description: >
  Ceforeで使えるコマンドの一覧。
---

Ceforeをインストールすると、シェルから操作するためのコマンドが多数使えるようになります。Cefore-Emuなど、Cefore本体に同梱されていないツールのコマンドについては、それぞれのツールのページを参照してください。

## 起動と停止

### `cefnetdstart`

### `cefnetdstop`

### `cefnetd`

## 状態確認

### `cefstatus`

出力の例を以下に示す。

```
$ cefstatus
Version    : 1
Port       : 9896
Rx Frames  : 0
Tx Frames  : 0
Cache Mode : Excache
Faces : 7
  faceid =   4 : IPv4 Listen face (udp)
  faceid =   5 : IPv6 Listen face (udp)
  faceid =  16 : address = 1.0.0.2:9896 (udp)
  faceid =  17 : Local face
  faceid =   0 : Local face
  faceid =   6 : IPv4 Listen face (tcp)
  faceid =   7 : IPv6 Listen face (tcp)
FIB(App) :
  Entry is empty
FIB : 1
  ccn:/
    Faces : 16 (-s-)  
PIT(App) :
  Entry is empty
PIT :
  Entry is empty
```

### `cefinfo`

### `cefping`

## 送受信

### `cefgetfile`

### `cefputfile`

### `cefgetchunk`

### `cefgetstream`

### `cefputstream`

## キャッシュ関連

### `csmgrdstart`

### `csmgrdstop`

### `csmgrstatus`

### `csmgrecho`

### `csmgrd`

## その他

### `cefroute`

### `cefctrl`