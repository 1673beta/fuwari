---
title: 各種軽量ActivityPub実装比較
published: 2024-10-11
description: お好み
tags: [Fediverse, docker, ActivityPub, Go, Rust, Hollo]
category: 技術
draft: true
---

[この前](https://blog.esurio1673.net/posts/hollo/)軽量なActivityPub実装であるHolloを建てた。  
その後「OSCまでにPleromaとの比較記事書きます」と宣言したのでPleromaや他の軽量な色々なAP実装を建ててみて比較してみる。  
諸事情でPleromaの代わりにフォークのAkkomaを使っているが許してほしい。  

## 比較対象一覧

|ソフトウェア|リポジトリ|ライセンス|開発言語|API|絵文字リアクション
|---|---|---|---|---|---|
|Hollo|https://github.com/dahlia/hollo|AGPL3.0|TypeScript|Mastodon API|OK|
|Akkoma|https://akkoma.dev/AkkomaGang/akkoma/|AGPL3.0|Elixir|Mastodon API/Pleroma API|OK|
|Mitra|https://codeberg.org/silverpill/mitra|AGPL3.0|Rust|Mastodon API/Pleroma API|OK|
|GoToSocial|https://github.com/superseriousbusiness/gotosocial|AGPL3.0|Go|Mastodon API|No|

# 動かしてみた

## 環境
- OS: Fedora 40
- CPU: Ryzen 7900X
- RAM: 32GB(DDR5-4800)
- ストレージ: 128GB(RAID1, SN770 + T700)

## Hollo
メモリ使用量は多くて1GB程度といったところ。GitHub Container Registryで配布されているイメージを使うのでビルドのリソースはあまり考慮しなくて良い。
メイン開発者の方がFedibirdにいるので日本語で色々聞ける。

## Mitra
メモリ使用量は多くて100MB程度。
ただしRustなのでビルドが非常に重い。
バイナリを実行する形なので、手元のマシンがリソース豊富ならそちらでビルドして転送するのもいいと思う。
フロントエンドとバックエンドが分かれている。

## GoToSocial
メモリ使用量は多くて200MB程度。
ヘッドレスなおひとり様向けActivityPub実装という点はHolloと似ているが、Holloと違い、ダイレクトが送信できない。Golangなのでビルドしてもそこまで重くないし、GitHub Container Registryでイメージも配布されている。

## Akkoma
メモリをあればあるだけ食べていくらしい。私の環境では8GBくらい食べている。
一応低メモリ環境でも動く。

## 雑まとめ(個人の感想)
絵文字リアクションがしたい→ Hollo, Mitra  
とにかくGCPの無料枠くらいの少ないリソースで動かしたい→Mitra, GoToSocial  
ビルドするリソースがない→Hollo, GoToSocial  
日本語でサポートを受けたい→Hollo  
