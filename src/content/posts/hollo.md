---
title: HolloをDocker ComposeとCloudflare Tunnelで建てる
published: 2024-10-03
description: おひとり様向けActivityPub実装
tags: [Fediverse, Hollo, docker]
category: 技術
draft: false
---

:::caution
この記事はHollo 0.1.0時点の記事であり、古い内容が含まれています。  
最新の情報は必ずドキュメントを参照してください。
:::

おひとり様向けActivityPub実装のHolloを自宅に建てたので備忘録。

# Holloとは
おひとり様向けの軽量なActivityPub実装で、FedifyとBunを使って作成されている。  
FedifyはDenoで作成されたActivityPubフレームワークで、Deno以外にもBunやNode.jsでも利用することができる。  
CLIも独立してインストールすることができる。  
Holloのドキュメントはここ：https://docs.hollo.social

:::important
2025年1月11日追記: この記事はHollo 0.1.0時に執筆されたもので、現在はBunではなくNode.jsを使用しています。
:::

# 方法
例によってCloudflare Tunnelを経由して外部と通信する。  
今回は~~systemd派を名乗っているにも関わらず~~Docker Composeを使用する。

:::warn
2025年1月29日修正: リポジトリのURLを修正
:::

```bash
git clone https://github.com/fedify-dev/hollo.git
cd hollo
cp .env.sample .env
```
次のように`.env`を編集する。
`BEHIND_PROXY=true`にしないとCloudflare Tunnel環境下でフォローができないなどの不具合があるので気をつけること。
:::warning
2025年1月11日追記: 現在はRedisを使用しないため、REDIS_URLは不要です。
:::
```env
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
REDIS_URL=redis://localhost/0
HOME_URL=https://hollo.example.com/ # optional; if present, the home page will redirect to this URL
SECRET_KEY=nankaiikanjinamojiretsu  # generate a secret key with `openssl rand -base64 32`
LOG_LEVEL=info
BEHIND_PROXY=true 
```

続いてcompose.yamlを編集する。
今回はCloudflare Tunnelを使うのでtunnelの部分を書き加える。

:::warning
2025年1月11日修正: Redisを必要としないためコメントアウト
2025年1月29日修正: リポジトリがdahlia/holloからfedify-dev/holloに
:::

```yml
services:
  hollo:
    image: ghcr.io/fedify-dev/hollo:latest
    ports:
    - 3033:3000
    environment:
      DATABASE_URL: "postgres://user:password@postgres:5432/database"
      REDIS_URL: "redis://redis:6379/0"
      SECRET_KEY: "do openssl rand -hex 32"
      LOG_LEVEL: "info"
      BEHIND_PROXY: "false"
      S3_REGION: us-east-1
      S3_BUCKET: hollo
      S3_URL_BASE: http://localhost:9000/hollo/
      S3_ENDPOINT_URL: http://minio:9000
      S3_FORCE_PATH_STYLE: "true"
      AWS_ACCESS_KEY_ID: minioadmin
      AWS_SECRET_ACCESS_KEY: minioadmin
    depends_on:
    - postgres
    # - redis
    - minio
    - create-bucket
    restart: unless-stopped

  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: database
    volumes:
    - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  #redis:
  #  image: redis:7
  #  restart: unless-stopped

  minio:
    image: minio/minio:RELEASE.2024-09-13T20-26-02Z
    ports:
    - "9000:9000"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
    - minio_data:/data
    command: ["server", "/data", "--console-address", ":9001"]

  create-bucket:
    image: minio/mc:RELEASE.2024-09-16T17-43-14Z
    depends_on:
    - minio
    entrypoint: |
      /bin/sh -c "
        /usr/bin/mc alias set minio http://minio:9000 minioadmin minioadmin;
        /usr/bin/mc mb minio/hollo;
        /usr/bin/mc anonymous set public minio/hollo;
        exit 0;
      "
  
  tunnel:
    image: cloudflare/cloudflared:latest
    command:
      - tunnel
      - --no-autoupdate
      - run
    environment:
      - TUNNEL_TOKEN=value

volumes:
  postgres_data:
  minio_data:

```

全て終わったら保存して、`docker compose up -d`をすると動く。
正常に起動している場合、`HOME_URL`に設定したURLにアクセスするとセットアップ画面にアクセスできる(初回のみ)。  
セットアップが終了したら、アカウントを作成することができるようになる。  

Mastodon API互換ではあるので多くのクライアントで動くが、一部のクライアントでは動かない(Nightfox DAWNだとダメだった)。
ドキュメントではPhanpy.socialをお勧めしていた。  
Elkでも動くが、絵文字リアクションに対応していないので通知画面でエラーが表示されることがある。  

# 感想
:::note
2025年1月11日現在、対応しているクライアントではカスタム絵文字リアクションの送受信が可能です。
使用可能なクライアントについては[こちら](https://docs.hollo.social/ja/clients/)を参照してください。
:::

非常に軽量でシンプルなため、「おひとり様が欲しい！でもサーバーのスペックが足りない！or 低い！」という人向けにかなりおすすめできる。  
おひとり様MastodonやMisskeyは管理コストが負担になることもあるので、その辺りを割り切っているHolloは、維持管理に疲れた、という人にもいいかもしれない。  
Misskey系の絵文字リアクションにはまだ非対応なようだが、Fedify側で対応するようなのでそのうち追加されることが期待できる。  
Pleroma、Akkomaに代わるおひとり様実装として今後の発展を願う。  
