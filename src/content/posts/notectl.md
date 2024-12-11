---
title: Misskey管理者向けCLIツールnotectlを作った
published: 2024-12-12
description: Misskey版tootctl
tags: [Fediverse, notectl, Rust]
category: その他
draft: false
---

タイトル通り、Misskey管理者向けCLIツールのnotectlを作って公開してる、という記事。  

## できること
- webpushの公開鍵と秘密鍵を生成できる
- aid/aidx/ULIDから日時を取得する
- .config/default.ymlの中身を表示できる
- リモートサーバーの状態を管理できる
- Meilisearchに対して検索インデックスを張れる
- Meilisearchに張った検索インデックスを削除できる
- Meilisearchのヘルスチェック

## なんで作ったの
Mastodonにはtootctlというメンテナンスや管理に使えるCLIツールがあるのですが、現在のMisskeyにはありません(6年前はあったようですが)。  
元々GoでCLIを作る練習としてMisskey管理用のCLIを作っていたのですが、私以外にも需要がある可能性があるので、きちんと書き直そうということでRustで書き直しました。  

## 作る上で困ったこと
MeilisearchのSDKがそこそこめんどくさかったなという印象です。  
あとは非同期の関数を`.await`をつけずに実行したせいでなんで出力されないんだろうと唸っていたり、コード補完側で`std::core::Result`が`std::fmt::Result`になっていてエラーが出てなんでだろうと唸ってたりしていました。  

## 今後実装する予定がある機能
- 指定した日時以前のリモートのノートを削除する機能
- objectid, meidの対応

## 実装するかもしれない機能
- FTTのキャッシュを吹き飛ばすコマンド

## 実装しない機能
- リモートにActivityを飛ばす必要のある機能(example: ローカルのノートを消す機能)