---
title: CherryPickをWSLからFedora Serverに移動させる
published: 2024-09-04
description: さらばWindows
tags: [PostgreSQL, nginx]
category: 技術
draft: false
---

[前回](https://blog.esurio1673.net/posts/db-move)WSL上のUbuntu-22.04に移行させたが、WindowsにRecallが正式導入されるようなので、別れを告げることにした。  
加えて、私自身もゲームをすることがほとんどなくなったため、別にWindowsで困ることがなくなった。  

## FedoraのUSBを焼く
FedoraにはDebianなどと違ってFedora Media Writerという公式のイメージ書き込みツールがあるのでそれを使うと非常に高速に書き込みができた。  
Fedora Workstationと迷ったが、どっちでもあまり変わりはないようなのでとりあえずServerにした。

## ばいばいWindows
Windowsから必要なデータをバックアップしたら、電源を一旦落としてBIOSに入る。  
私はASRock製のマザーボードを使用してたので、BIOSのツール＞NVMe Tool ＞ Sanitaizationだかが書いてあるとこを選んでWindowsが入ったディスクのデータを吹き飛ばす。  
また、起動メニューからWindows Boot ManagerをDisabledにしておく。  
一旦保存させて再起動。

## こんにちは、Fedora
Fedora自体はAsahi Linuxをインストールしたときに一度使ってるので初めてではない。  
Fedoraを焼いたUSBを挿して、再起動。  
なんか出てくるのでInstallだか書いてある一番上を選ぶ。  
するとAnaconda Installerが起動するので、あとはそれの指示に従う。  
体感としては、UbuntuやDebianよりもわかりやすく便利だと思った。

## DBバックアップ
gzipを使って圧縮する。前回と同じだけどメモ。
```bash
sudo apt update
sudo apt install -y gzip
sudo -i
sudo -u postgres pg_dumpall | gzip -c > /var/BACKUP.gz
```

## Tailscale経由で色々する
SSHやCockpitへのアクセスを全部Tailscale経由にする。
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --accept-dns --accept-routes --ssh
sudo firewall-cmd --permanent --zone=public --add-interface=tailscale0
sudo firewall-cmd --permanent --zone=public --add-port=9000/tcp
sudo firewall-cmd --reload
sudo tailscale funnel -bg --htttps=443 9090
```
多分これでできたはず…

## DBリストア
前回と違い、今回はrpm系なのでちょっと違う。

### PostgreSQL 16のインストール
Ubuntuと微妙に違う。
```bash
sudo dnf install -y postgresql16 postgresql16-server
sudo su - postgres
/usr/pgssql-16/bin/initdb
exit
sudo systemctl start postgresql-16.service
```
とりあえずこれでDBが立ち上がり、`postgres`ユーザーで`psql`コマンドが実行できる。

```sql
CREATE ROLE misskey LOGIN PASSWORD 'STRONGPASSWORD';
CREATE DATABASE hoge OWNER misskey;
\q
```
ひとまずここまで。
ダンプファイルを`scp`などを使って転送すること。
```bash
gzip BACKUP.gz
sudo -i
cp /home/User/BACKUP /var/lib/pgsql
sudo chown postgres:postgres /var/lib/pgsql/BACKUP
su - postgres
ls -l #この時点でバックアップがないとどこかでミスってる
psql -d hoge -f BACKUP
```

## Valkeyのインストール
Redisでいいかな？と思ったけど、せっかくなので前から興味のあったValkeyにすることにした。
```bash
sudo dnf install -y valkey
sudo systemctl start valkey
sudo systemctl enable valkey
valkey-cli #ここで繋がれば成功
```

## Valkeyを複数ポートで立ち上げる
今回はサーバーを2つ移行するつもりなので、Valkeyのプロセスを一旦止める。
```bash
sudo systemctl stop valkey
sudo systemctl disable valkey
```
Valkeyのsystemdのserviceファイルを複数用意する。
```bash
sudo cp -upv /usr/lib/systemd/system/valkey.service /usr/lib/systemd/system/valkey_nyan.service
sudo cp -uvp /etc/valkey/valkey.conf /etc/valkey/valkey_nyan.conf
```
なんかいい感じに設定変える。
で、daemon-reloadしてstart & enable。

あとは前回と同じようにやれば動く。