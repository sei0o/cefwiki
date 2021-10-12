---
title: "ファイルの送受信"
linkTitle: "ファイルの送受信"
weight: 8
date: 2021-10-12
description: ''
---

まずはファイルをceforeで送ってみましょう。

## 前提
[Dockerイメージの導入](/docs/quickstart) が済んでいること。

適宜読み替えれば物理的な環境でも実行できます。（ローカル）キャッシュが有効になっていることを確認してください。TODO: 詳細な説明

## 手順
### 環境をつくる
ここではDocker Composeを使って仮想環境を構築します。`tutorial/`ディレクトリを作成して、そこに移動してください。
```shell
$ mkdir tutorial
$ cd tutorial
```

以下の設定を`tutorial/docker-compose.yml`として保存してください。
```docker-compose.yml
version: '3.8'

services:
  consumer:
    image: cefore/cache
    hostname: "consumer"
    working_dir: "/cefore"
    networks:
      shared:
        ipv4_address: 10.0.1.10

  publisher:
    image: cefore/cache
    hostname: "publisher"
    working_dir: "/cefore"
    networks:
      shared:
        ipv4_address: 10.0.1.30

networks:
  shared:
    name: shared
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 10.0.1.0/24
```

この設定では、consumerとpublisherという2つのノードを、10.0.1.0/24に接続しています。このチュートリアルでは、publisherに登録したファイルをconsumerに転送するのがゴールです。

`tutorial`ディレクトリにいる状態で、仮想環境を立ち上げます。必要に応じて `sudo` をつけてください。
```shell
$ docker-compose up -d
Starting tutorial_consumer_1  ... done
Starting tutorial_publisher_1 ... done
```

ノードが動いていることを確認します。`$ docker stats`を実行して、次のような表示が出れば大丈夫です（容量などは当然異なります）。
```shell
$ docker stats
CONTAINER ID   NAME                   CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O    PIDS
c549ba77183d   tutorial_publisher_1   0.00%     1.18MiB / 15.55GiB    0.01%     5.77kB / 0B   164kB / 0B   3
67991cfb4878   tutorial_consumer_1    0.00%     1.023MiB / 15.55GiB   0.01%     5.77kB / 0B   0B / 0B      3
```

consumer, publisherをシェルから操作できるようにします。端末を2つ立ち上げて、それぞれで次のコマンドを実行してください。
```shell
// consumer （←以降、操作する端末をこのように表記します）
$ docker exec -it tutorial_consumer_1 bash

// publisher
$ docker exec -it tutorial_publisher_1 bash
```

Dockerを使わない場合は、2台PCを用意して、それぞれでCeforeを起動してください。以降の手順では、`10.0.1.10`をconsumerのIPアドレスに、`10.0.1.30`をpublisherのIPアドレスに読み替えてください。

### FIBの設定
**FIB (Forwarding Interest Table)** は、情報の名前（prefix）とそのありかの組み合わせを経路情報として記録するテーブルです。情報を取得するときには、ノード自身が持つFIBを参照して、転送先を決定します。転送されたノードは、自身の持っている情報であればそれを応答として返し、持っていなければまた自身のFIBを使ってたらい回しに転送します。

さて、現状２つのノードはIPネットワークでは接続されているものの、Ceforeは互いを認識していません。これではICNによる通信ができないので、手動でFIBを設定します（あくまでCeforeはIPネットワーク上で動いているので、IPアドレスによって転送先を表現する必要があります）。consumerの端末にて、次のコマンドを実行してください。
```shell
// consumer
$ cefroute add ccnx:/example tcp 10.0.1.30
```

`cefstatus`コマンドを使って、FIBが更新されたことを確認します。`ccnx:/example` の下に対応するFaceの番号が表示されます（FaceはTCPでいうポートのようなものです）。
```shell
// consumer
$ cefstatus
Version    : 1
Port       : 9896
Rx Frames  : 0
Tx Frames  : 0
Cache Mode : Localcache
Faces : 7
  faceid =   4 : IPv4 Listen face (udp)
  faceid =   0 : Local face
  faceid =  18 : address = 10.0.1.30:9896 (tcp)
  faceid =  19 : Local face
  faceid =   5 : IPv6 Listen face (udp)
  faceid =   6 : IPv4 Listen face (tcp)
  faceid =   7 : IPv6 Listen face (tcp)
FIB(App) :
  Entry is empty
FIB : 1
  ccnx:/example // ←
    Faces : 18 (-s-)
PIT(App) :
  Entry is empty
PIT :
  Entry is empty
```

一度consumer側でFIBに登録すると、publisher側ともやりとりがなされます。publisher側でも`cefstatus`を実行してみると、Faceが更新されconsumerのIPアドレスが設定されていることがわかります。
```shell
// publisher
$ cefstatus
Version    : 1
Port       : 9896
Rx Frames  : 0
Tx Frames  : 0
Cache Mode : Localcache
Faces : 7
  faceid =   4 : IPv4 Listen face (udp)
  faceid =   0 : Local face
  faceid =   5 : IPv6 Listen face (udp)
  faceid =  17 : address = 10.0.11.10:9896 (tcp) // ←
  faceid =   6 : IPv4 Listen face (tcp)
  faceid =  18 : Local face
  faceid =   7 : IPv6 Listen face (tcp)
...
```

### 送受信
経路が設定できたので、実際にファイルを送りましょう。まずpublisherノードに適当なファイルを登録します。`cefputfile`コマンドを使うのが手軽です。ここでは `ccnx:/example` というアドレス（プレフィックス）に`Hello, Cefore!` という中身のファイルを登録します。
```shell
$ echo "Hello, Cefore!" > example
$ cefputfile ccnx:/example -f example
[cefputfile] Start
[cefputfile] Parsing parameters ... OK
[cefputfile] Init Cefore Client package ... OK
[cefputfile] Conversion from URI into Name ... OK
[cefputfile] Checking the input file ... OK
[cefputfile] Connect to cefnetd ... OK
[cefputfile] URI         = ccnx:/example
[cefputfile] File        = example
[cefputfile] Rate        = 5.000 Mbps
[cefputfile] Block Size  = 1024 Bytes
[cefputfile] Cache Time  = 300 sec
[cefputfile] Expiration  = 3600 sec
[cefputfile] Start creating Content Objects
[cefputfile] Unconnect to cefnetd ... OK
[cefputfile] Terminate
[cefputfile] Tx Frames  = 1
[cefputfile] Tx Bytes   = 14
[cefputfile] Duration   = 0.004 sec
[cefputfile] Throughput = 34063 bps
```

TODO: キャッシュ見る

consumerノードでファイルを取得します。`Complete`が見えれば成功です。`Incomplete`が表示されると失敗です。
```shell
// consumer
$ cefgetfile ccnx:/example
[cefgetfile] Start
[cefgetfile] Parsing parameters ... OK
[cefgetfile] Init Cefore Client package ... OK
[cefgetfile] Conversion from URI into Name ... OK
[cefgetfile] Checking the output file ... OK
[cefgetfile] Connect to cefnetd ... OK
[cefgetfile] URI=ccnx:/example
[cefgetfile] Start sending Interests
[cefgetfile] Complete
[cefgetfile] Unconnect to cefnetd ... OK
[cefgetfile] Terminate
[cefgetfile] Rx Frames = 1
[cefgetfile] Rx Bytes  = 14
[cefgetfile] Duration  = 0.000 sec
[cefgetfile] Jitter (Ave) = 0 us
[cefgetfile] Jitter (Max) = 0 us
[cefgetfile] Jitter (Var) = 0 us
```

取得したファイルはデフォルトでprefixの最後のスラッシュ以降と同じ名前になります（`ccnx:/example`なら`example`）。内容を見てみましょう。
```shell
// consumer
$ cat example
Hello, Cefore!
```

`Hello, Cefore!`となっていれば正しく受信できています。やったー！

なお、Docker Composeで作った環境を終了するには、`tutorial/`ディレクトリにて`$ docker-compose stop`を実行してください。
```shell
$ docker-compose stop
Stopping tutorial_publisher_1 ... done
Stopping tutorial_consumer_1  ... done
```

## 発展
- 間にもう1つノード（router）を挟んで、publisher→router→consumerのように、間接的にファイルを送ってみましょう。
	- `docker-compose.yml`はどこを書き換えればいいですか？
	- FIBはどのようにすればいいでしょうか？
	- ヒント：公式ユーザマニュアル
- publisherのキャッシュを無効にするとどうなるでしょうか？
- FIBが更新されたことはどのように伝えているのでしょうか？

## トラブルシューティング
### ファイルが送れない
- キャッシュは有効になっていますか？
	- `$ cefstatus`を実行して、5行目ぐらいに`Cache Mode: Localcache`とあればローカルキャッシュが有効になっています。
- pingは通りますか？
	- consumerから`$ ping 10.0.1.30`を実行して応答があれば、IPレベルでは接続できています。
- FIBは正しい？ccninfo.

### cefnetdに接続できない
`ERROR: Failed to connect to cefnetd.` などと表示される場合。

対策：`cefnetdstart` をbashから実行してみてください。`ERROR: Another cefnetd may be running with the
same port. If not, remove the old socket file (/tmp/cef_9896.0) and restart cefnetd.` などと出る場合は、該当のソケットファイル（ここでは `/tmp/cef_9896.0`）を削除してからもう一度試してください。