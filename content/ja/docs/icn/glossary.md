---
title: "用語集"
linkTitle: "用語集"
weight: 4

---

ICNやCeforeでよく使われる言葉をまとめます。公式資料やRFCを参考にしています。

## Interest
類義語：リクエスト

「こんなデータがほしい」ということを表現するパケットです。

## Data
類義語：レスポンス

## Cob
→Data

## Face
類義語：ポート

それぞれのノード（cefnetd）が持っていて、他のノードとの窓口になるものです。`cefstatus`を実行すると、そのノードが持つポートを見ることができます。FIBを設定すると、それに応じて自動でFaceが追加されます。

```shell
$ cefstatus
Version    : 1
...(snip)...
Faces : 9
  faceid =   4 : IPv4 Listen face (udp)
  faceid =   0 : Local face
  faceid =  16 : address = 10.0.1.30:9896 (udp)
  faceid =  29 : Local face
  faceid =   5 : IPv6 Listen face (udp)
  faceid =  17 : address = 10.0.1.10:9896 (tcp)
  faceid =   6 : IPv4 Listen face (tcp)
  faceid =   7 : IPv6 Listen face (tcp)
  faceid =  19 : address = 10.0.1.30:9896 (tcp)
FIB(App) :
  Entry is empty
FIB : 2
  ccnx:/example
    Faces : 16 (-s-)
  ccnx:/hoge
    Faces : 19 (-s-)  16 (-s-)
...(snip)...
```

## Outgoing

## Incoming

## CS (Content Store)

## FIB (Forwarding Interest Base)

## PIT (Pending Interest Table)

## AppComp