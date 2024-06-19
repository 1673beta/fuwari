---
title: nginxでIP直打ちを拒否する
published: 2023-12-28
description: DDoS対策
tags: [nginx]
category: 技術
draft: false
---

# 概要
`/etc/nginx/conf.d/default.conf`を修正する。

```
server {
    listen 80 default_server;
    server_name _;
    return 444;
}

server {
    listen 443 ssl default_server;
    server_name _;
    ssl_certificate ***.pem;
    ssl_certificate_key ***.pem;
    return 444;
}
```

# メモ
- 444はnginxの拡張コードで、何もデータをレスポンスしない、というものらしい

- なのでこの状態だと5432ポート(デフォルト)からpsqlのデータを取り出せない

- `allow/deny`や`geo`をうまく使うとIP単位でアドレスごとに制限ができるっぽい

- もちろんWAFで制限をかけるのも大事