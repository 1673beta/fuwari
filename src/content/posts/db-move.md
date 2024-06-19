---
title: PostgreSQLのDBを別サーバーに移行する
published: 2024-02-22
description: 自鯖の移行
tags: [PostgreSQL, nginx]
category: 技術
draft: false
---

個人鯖をVultrから自宅のWSLに移行したので備忘録。

## DBバックアップ
gzipを使って圧縮して取り出す。

```bash
sudo apt update #いつもの
sudo apt install -y gzip #こいつで圧縮する
sudo -i #root切り替え
sudo -u postgres pg_dumpall | gzip -c > /var/backups/BACKUP_NAME.gz #pg_dumpallでバックアップを取り、gzipで指定した場所に指定した形式で保存
```
終わったらSFTPなどでバックアップしたファイルを一旦作業用環境に移動させておく。

## DBリストア
[Misskey Hubの公式手順のここまで](https://misskey-hub.net/ja/docs/for-admin/install/guides/ubuntu-manual/#nginx%E3%81%AE%E8%A8%AD%E5%AE%9A)終わらせておくこと。追加の設定はしなくてよい。  
**DBのユーザー名とDB名は必ず前と同じにすること。**  
作業用ユーザーのhomeディレクトリ(sudoが使える必要あり)にバックアップファイルを入れておくこと。

```bash
gunzip BACKUP_NAME.gz
sudo -i
mv /home/USER/BACKUP_NAME /var/lib/postgresql
sudo chown postgres:postgres /var/lib/postgres/BACKUP_NAME
su - postgres
ls -l
psql -d DB_NAME -f BACKUP_NAME
```
これでリストア終わり。  
下は確認用。

```bash
psql
\c DB_NAME
\dt #これで大量にMisskey関連のテーブルが出たら成功
```

## Argo Tunnel
ダッシュボードに入ってZeroTrustからTunnelを作り、手順通りにやる。

## nginx
DBと同じように`/etc/nginx/conf.d`以下に移植していいが、以下のようにしておくこと。rootで作業する。
```

# nginx configuration for Misskey
# Created by esurio, from koliosky.com

# For WebSocket
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache1:16m max_size=1g inactive=720m use_temp_path=off;

server {
    listen 80;
    listen [::]:80;
    server_name engawa.esurio1673.net;

    # For SSL domain validation
    # root /var/www/html;
    # location /.well-known/acme-challenge/ { allow all; }
    # location /.well-known/pki-validation/ { allow all; }

	# with https
    location / { return 301 https://$server_name$request_uri; }
}
server {
    #listen 443 ssl http2;
    # listen [::]:443 ssl http2;
    # server_name engawa.esurio1673.net;

    # ssl_session_timeout 1d;
    # ssl_session_cache shared:ssl_session_cache:10m;
    # ssl_session_tickets off;

    # To use Let's Encrypt certificate
    # ssl_certificate     /etc/letsencrypt/live/esurio1673.net/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/esurio1673.net/privkey.pem;
    # SSL protocol settings
    ssl_protocols TLSv1.2 TLSv1.3;
 以下略

```

## serviceファイルを移植する
`[Service]`の`User`はMisskey(CherryPick)をインストールしたユーザーにする。ディレクトリも合わせて変更しておくこと。