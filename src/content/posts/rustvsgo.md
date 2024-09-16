---
title: GoとRustで同じ用途のCLIツールを作ってみた感想
published: 2024-09-16
description: 煽りではない
tags: [Rust, Go]
category: 技術
draft: false
---

## はじめに
最近自鯖でActivityPub周りの対応をやっていたのだが、その時にデバッグするためには`curl`を叩かないといけない。  
でもいちいちcurlにオプションつけて叩くのもめんどくさい。3日も経てば忘れている。  
で、CLIツールを作ることにした。  

## RustとGo
とりあえず最初はGoで書いた。Goは文法がシンプルで可読性も高く、何よりCobraというCLI作成用のフレームワークのおかげで雑に書いても何とかなる、というのがある。私のようなヒトモドキにはちょうどいい。  
でも頭が悪いので、Rustにしたら実行速度が速くなるのか気になった。  
正直Goもバイナリを生成するのでそれなりに高速である。  
しかしRustは実行速度がうりの1つ。なので書いてみることにした。

## 作ったもの
作ったのは次の`curl`の実行を楽にしてくれるやつである。
```bash
curl --head "Accept: application/activity+json" https://example.com | jq .
```

#### Rust版
https://github.com/1673beta/aplookup-rs
##### インストール方法
```bash
cargo install aplookup
```

#### Go版
https://github.com/1673beta/aplookup
##### インストール方法
```bash
cd aplookup
go build
export PATH=$PATH:/home/You/aplookup
```

## 計測方法
macOSには`time`コマンドがあり、`time ls`のようにするとlsコマンドの実行時間を調べることができる。Rustはv1.79.0、Goは1.22.4を使用している。  

## 測定結果

timeコマンドを使用し、生成したバイナリを実行しその平均を取った結果次の通りになった。
各プロパティは次を意味する。
- CPU: プログラムがCPU使用した割合
- user: ユーザーモードでのCPU時間
- system: システムモードでのCPU時間
- total: 実際の経過時間

|言語|CPU|user|system|total|  
|----|----|----|----|----|
|Rust|5.3%|0.0148s|0.0075s|0.3952s|
|Go|6.8%|0.0085s|0.0166s|0.3493s|

## 感想
Rustは必ずしも速いわけではないし、「速いからRustで書こうぜ！」は絶対にやめた方がいいなと思った。
CLI程度ならむしろGoの方がシンプルで雑に書きやすく、エラーも修正しやすくていいなと思った。