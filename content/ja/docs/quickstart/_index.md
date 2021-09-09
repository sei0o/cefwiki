---
title: "セットアップ"
linkTitle: "セットアップ"
weight: 2
description: >
  Ceforeを動かそう
---

Ceforeを動かすには、手元のマシンでビルドする方法と、Dockerイメージを使う方法があります。

## 手元でビルドする

オーソドックスな手段です。Automakeをインストールしていることが前提です。

1. [ダウンロードページ](https://cefore.net/download)から`cefore-0.???.zip`をダウンロード・展開する。
2. `$ autoconf && automake`を実行する。
3. `$ ./configure` を実行する。必要に応じて以下のオプションを指定する。

| オプション              | 説明                           |
| ------------------ | ---------------------------- |
| `--enable-csmgr`   | キャッシュ（[csmgr](#)）関連の機能をビルドする |
| `--enable-cefinfo` | CCNinfoの実装をビルドする             |
| `--enable-cefping` | cefpingをビルドする                |
| `--enable-samptp`  | samptpプラグインをビルドする            |
| `--enable-ndn`     | NDNプラグインをビルドする               |

3. `$ make && sudo make install` を実行する。

## Dockerを使う

2021年のICN研究会で配布されたDockerイメージを用いる方法です。多数の仮想的なノードを用いた実験などに便利です。Git, DockerおよびDocker Composeをインストールしていることが前提です。

```shell
$ git clone http://github.com/cefore/2021-hands-on.git
$ cd practice-A
$ ./build.bash # cefore/base,min,cache,csmgr という4種類のイメージをビルド
```

必要に応じて `./build.bash` を使わず、直接 `docker build` してもかまいません。

TODO: dockerを動かす方法