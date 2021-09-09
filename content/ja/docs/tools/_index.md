---
title: "ツール"
linkTitle: "ツール"
weight: 2
description: >
  Ceforeのツール群を扱います。
---

## CCNinfo

ネットワーク全体で、コンテンツを配信しているノードやキャッシュの様子を知るためのツールです。Request, Replyパケットという特殊なパケットを通じて情報を取得する。暫定的な仕様はIRTFの[draft-irtf-icnrg-ccninfo](https://datatracker.ietf.org/doc/draft-irtf-icnrg-ccninfo/)にまとめられています。

Ceforeでは、`cefinfo`コマンド（またはエイリアスの `ccninfo`コマンド）として実装されています。

## Cefore-Emu

Python2系で動作するCeforeのエミュレータ。Mini-CCNx（開発停止）をベースに開発された。

## CeforeSim

## cefbabel