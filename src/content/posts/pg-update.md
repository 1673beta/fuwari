---
title: PostgreSQlのバージョンアップデート
published: 2023-11-17
description: お風呂入れ
tags: [PostgreSQL]
category: 技術
draft: false
---


PostgreSQLのバージョンを15から16にした備忘録。

## 1. 前提
Misskeyなど、DBを利用しているdaemonがあったら止めておくこと。

## 2. 当時の環境
- Ubuntu 22.04.3 LTS
- Vultr Cloud Compute


## 3. 手順
```bash
sudo apt update #いつもの
sudo apt install postgresql-16
pg_lsclusters #動作中のpsqlのcluster確認
sudo -i #root
sudo -u postgres pg_dumpall | gzip -c > /var/backups/BACKUP_NAME.gz #gz形式に圧縮して/var/backupsにバックアップ
ls -la /var/backups #取れたか確認
sudo systemctl stop postgresql@15-main
sudo systemctl stop postgresql@16-main
sudo -u postgres pg_dropcluster 16 main --stop
sudo -u postgres pg_upgradecluster -v 16 15 main
pg_lsclusters
exit
sudo pg_dropcluster 15 main
pg_lsclusters
sudo apt-get purge postgresql-15 postgresql-client-15
```

## 3. 注意事項
1. `pg_dumpall`はDBクラスタ単位であることに注意。

2.  `su - postgres`でpostgresユーザーになるか、`sudo -u`でpostgresユーザーとして`pg_dropcluster`を実行する必要がある。

3. DBがGB単位だとバックアップにとても時間がかかるので風呂に入っておくこと。

4. 焦らない。 