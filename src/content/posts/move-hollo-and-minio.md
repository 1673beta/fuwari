---
title: HolloのminioをCloudflare R2に移行する
published: 2025-06-05
description: 意外とめんどくさかった
tags: [Fediverse, Hollo, docker, minio]
category: 技術
draft: false
---

私が引っ越すためHolloの引っ越し作業をするついでに、minioをCloudflare R2に移行したので備忘録。  

# minioを辞めた理由
最新のminioでは、ダッシュボードが以前のバージョンに比べてほとんどの機能を削減されてしまい、セルフホストするメリットがほとんど消失してしまった。  
また、VPSはストレージ容量が自宅サーバーにしているマシンに比べ少ないため、Cloudflare R2に移行することにした。  

# 移行方法
あらかじめ、AWS CLIをインストール・セットアップしておく。  
また、Holloのコンテナは止めておくこと。  

```bash
docker compose exec minio mc alias set hollo http://localhost:9000 {MINIO_ACCESS_KEY} {MINIO_SECRET_KEY}
docker compose exec minio mc alias set r2 {R2_ENDPOINT_URL} {R2_ACCESS_KEY} {R2_SECRET_KEY}
```
以下のコマンドで、正しく設定されていればOK。
```bash
~$ docker compose exec minio mc alias ls

hollo
  URL       : http://localhost:9000
  AccessKey : MINIO_ACCESS_KEY
  SecretKey : MINIO_SECRET_KEY
  API       : s3v4
  Path      : auto

r2
  URL       : https://{R2_ENDPOINT_URL
  AccessKey : {R2_ACCESS_KEY}
  SecretKey : {R2_SECRET_KEY}
  API       : s3v4
  Path      : auto
```

次に、minioのデータをR2にミラーする。
```bash
docker compose exec minio mc mirror minio/hollo r2/hollo
```
その後、AWS CLIで以下を実行する。
```bash
aws s3 mv --profile r2 --endpoint-url {R2_ENDPOINT_URL} --recursive s3://hollo/ s3://hollo/hollo/
```
これで、DBマイグレーションをすれば概ね元通りになる。  
