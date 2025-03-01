---
title: 超初心者でもわかるかもしれないCherryPickを立てる方法
published: 2025-03-01
description: for 身内
tags: [PostgreSQL, nginx, redis, CherryPick]
category: 技術
draft: false
---

CherryPickの立て方は意外とどこにも載っていないので、身内向け兼備忘録がてら。  
といっても、99.9%はMisskeyと同じ。  
コピペフレンドリー(当社比)な記事。  
今回はたつまで、初期設定とか諸々はそのうちやるかもしれないしやらないかもしれない。  

## 今回の環境
### サーバー側
- シンVPS 2GB
- Ubuntu 24.04

### クライアント側
- macOS Sequoia 1.52
- ターミナルソフト: macOS標準搭載のターミナル

### DNSなど
- Cloudflare

### 前提
- Cloudflareでドメインを取得しているものとする

## シンVPS側の設定
パケットフィルターをOFFにしておく。また、VPSパネルからIPアドレスをコピーしておく。

## Cloudflareの設定
### DNS設定
ダッシュボードを開き、サーバーに使用したいドメインを選択する。  
ドメインの概要ページを開いたら、左のサイドバーからDNSを選択する。  
「レコードの追加」を押し、「タイプ」をA、「名前」を好きなサブドメイン(NOTE: 基本的に英数字にしておくほうが不便がない)、「コンテンツ」をサーバーの提供するIPv4アドレスにして保存する。シンVPSの場合、IPv4アドレスしか提供されていないため、コピーしたIPアドレスをそのままコンテンツに入力する。  

### SSLの設定
左のサイドバーから「SSL/TLS」を選択し、現在の暗号化モードが「フレキシブル」であれば、「設定」を押して「フル」にしておく。  
フレキシブルの場合、無限ループしてアクセスできなくなる。  

### Cache Rulesの設定
左のサイドバーから「Caching」を選択し、その状態でさらに左のサイドバーから「Cache Rules」を選択する。
「ルールを作成」を押し、「ルール名」に任意のわかりやすいルール名を、「フィールド」に「URIパス」、「オペレーター」に次を含む、値に「/api/」、実行内容を「キャッシュをバイパスする」にして保存する。  

### APIキーの取得
ダッシュボード右上のプロファイルからプロフィールを選択し、APIトークンを選択する。  
Global API Keyを「表示」してコピーして控えておく。このトークンは後に使う。  


## SSHする
### 1. SSHの用意をする
#### 鍵の作成
ユーザーディレクトリ(macOSなら`/home/自身のユーザー名`、Windowsなら`C:\%USERPROFILE%\`、通常`C:\Users\自身のユーザー名`)をターミナル(WindowsならターミナルかPowerShellかその他任意の好きなソフト)上で開いて、次のコマンドを実行する。
```bash
ssh-keygen -t ed25519
```
この際に保存場所やパスフレーズを聞かれるが、パスフレーズ以外は基本的にEnterをそのまま押して良い。  
パスフレーズは作成して必ずパスワードマネージャーなどに控えておくこと。  

#### configの設定
鍵の作成が終わったら、`.ssh`というフォルダが作成されているので、そこに`config`というファイルを設定する。拡張子は設定しない。  
VSCodeでもメモ帳でも好きなテキストエディタで、この`config`を開き、下記のようにする。
```txt
Host tutorial
  HostName 100.0.0.0
  Port 22
  User esurio
  IdentityFile ~/.ssh/id_ed25519
```
- Hostには、これから接続する際に使いたい任意の名前を入力する。ここでは、`tutorial`とする。
- HostNameには、サーバーのIPv4アドレスを入力する。ここではぼかすために`100.0.0.0`を入力して誤魔化しているが、実際にはVPS事業者から提供されたVPSのIPアドレスを入力すれば良い。  
- Portには、SSHで使用するポート番号を記述する。ここでは、標準の22番ポートを設定する。
- Userには、サーバー上に作成するユーザー名を設定する。ここでは、esurioとする。
- IdentityFileには、SSH接続する際に使う秘密鍵のパスを記述する。ここでは、先ほど作成した`id_ed25519`を使用したいため、`~/.ssh/id_ed25519`を設定する。

### 2. 接続する
[1.](#1-sshの用意をする)の内容が全て終わったら、再度ターミナルを開いて、次のように実行する。

```bash
ssh root@tutorial -p 22
```
何か聞かれると思うので、`yes`を入力してEnterをおす。 そうするとSSHで無事サーバー内に入ることができる。

## サーバーに入ってからの初期設定作業
### 1. パッケージの更新
[前項](#sshする)で接続した状態のままという前提で続ける。  
まず、次のコマンドを実行する。  
```bash
apt update && apt upgrade -y
```
:::tips  
`&&`はコマンドを連続実行する際に、前のコマンドがcode 0で終了したら次のコマンドを実行するというオペレーターである。  
:::

### 2. 作業用ユーザーの作成
rootでSSH接続することは、通常**高いセキュリティリスクを伴う**ので、SSH接続した際に使う作業用のユーザーを作成する。  
次のコマンドを実行する。この場合、`esurio`というユーザー名で作成されるので適宜変更すること。  
```bash
adduser esurio
```
`New password`と`Retype new password`を聞かれるので、パスワードマネージャなどで安全なパスワードを作成して、入力する。それ以外の項目は何も入力せずEnterを押してよい。  

**これ以降、`sudo`をつけて実行する際に聞かれるパスワードは`adduser`実行時に入力したパスワードであるため、パスワードマネージャなどで安全に保管しておくこと。**

### 3. 作業用ユーザーにsudo権限を付与する
このままでは、作業用ユーザーにログインしたとしてもroot権限が必要な場面で作業が滞る場面があるので、次のコマンドでsudoを許可するグループに入れる。  
```bash
usermod -aG sudo esurio
```
:::tips  
Ubuntu/Debianはsudoだが、RHEL/CentOS/Fedoraなどはwheelでsudoを許可することができる  
:::  


### 4. 作業用ユーザーでSSH接続できるようにする
実際に作業用ユーザーでSSH接続できるようにする。  
以下のコマンドで作業用ユーザーに切り替える。  
```bash
sudo -iu esurio
```
作業用ユーザーにログインできたら、次のコマンドを順に実行してsshする。
```bash
mkdir .ssh
```
```bash
chmod 700 .ssh
```
```bash
cd .ssh
```
```bash
touch authorized_keys
```
```bash
chmod 600 authorized_keys
```
```bash
nano authorized_keys
```
nanoの使い方は[付録](#nanoの基本的な使い方)を参照すること。  
nanoに入ったら、クライアント側で作成した公開鍵の中身をペーストする。  
Windowsの場合、`~/.ssh/id_ed25519.pub`をVSCodeやメモ帳で開いて全て選択してコピーする。Officeが入っている環境下では**Microsoft Publisherが公開鍵をMicrosoft Publisherのファイルと認識してそのままクリックするだけでは開けず無駄な時間を過ごすことになるので気をつけること。**  

### 5. サーバー側のsshdの設定を変更する
次のコマンドで、サーバー側のsshdの設定を変更する。  
この設定を変更しないままではセキュリティリスクが高い状態になるため注意すること。  
```bash
sudo nano /etc/ssh/sshd_config
```
以下の項目を右の項目の通りに変更する。
- `PermitRootLogin`: `no`
- `PubkeyAcceptedTypes`: `+ssh_rsa` → `+ssh_rsa,ssh_ed25519`
- `PasswordAuthentication`: `no`
- `UsePAM`: `no`
変更し終わったら、保存して退出する。

### 6. 変更が問題ないか確認する
以下のコマンドで確認する。  
```bash
sudo sshd -t
```
問題がなければ次へ、問題があれば修正すること。  

### 7. 設定を有効にする
以下のコマンドを実行する。  
```bash
sudo systemctl restart ssh
```

### 8. 接続できるか確認する
現在のターミナルを開いたままターミナルを新規に開いて、次のコマンドを実行して接続できるか確認する。  

```bash
ssh tutorial
```
接続できたら**新規に開いた方を**`exit`で退出する。

### 9. ufwを設定する
現在の設定ではファイアウォールが設定されていないので、セキュリティ上危険である。そのため、ファイアウォールを設定する。  
Ubuntuにはufwがあるので、これを利用する。  
以下のコマンドを実行する。  
```bash
sudo ufw limit 22/tcp
```
```bash
sudo ufw enable
```
`Firewall is active and enabled on system startup`と表示されたら、先ほどと同じようにターミナルを新規に開いて、SSH接続できるか確認する。 
接続を確認したら、**最初に開いた方の**ターミナルを終了させる。   

## Misskey/CherryPickの環境構築  
Misskey/CherryPickをビルド・動作させるための環境構築をする。ここではsystemd環境で動かすことを想定する。  
### 諸々の必要なパッケージ
```bash
sudo apt install -y git ffmpeg build-essential curl gpg lsb-release libjemalloc-dev libjemalloc2 gnupg2 ubuntu-keyring ca-certificates
```

### Node.js
ここでは、`n`を使ってNode.jsのバージョンを管理することとする。  
```bash
sudo apt install -y nodejs npm
```
```bash
sudo npm install -g n
```
```bash
sudo n stable
```
```bash
hash -r
```
```bash
node -v
```
### Redis
最新の情報は[公式ドキュメント](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/)を確認すること
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
```
```bash
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
```
```bash
sudo apt-get update
```
```bash
sudo apt-get -y install redis
```
```bash
sudo systemctl enable redis-server
```
```bash
sudo systemctl start redis-server
```
以下のコマンドを実行して`active (running)`だったらOK。  
```bash
sudo systemctl status redis-server
```
### PostgreSQL
```bash
sudo apt install -y postgresql-common
```
```bash
sudo sh /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -i -v 17;
```
以下のコマンドを実行して`active`ならOK。
```bash
sudo systemctl status postgresql
```

### nginx
```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```
```bash
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```
次のような出力があればOK。(私のためした限り上から2番目にこの出力がある)
```bash
pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>

```
```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```
```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```
```bash
sudo apt udpate
```
```bash
sudo apt install -y nginx
```

## データベースの作成
ロール名、パスワード、データベース名は適宜変更すること。また、ロール名、パスワードとデータベース名はCherryPick側でも使うので控えておくこと。  
```bash
sudo -u postgres psql
```
```sql
CREATE ROLE cherrypick LOGIN PASSWORD 'hoge';
```
```sql
CREATE DATABASE cp1 OWNER cherrypick;
```
```sql
\q
```
## ファイアウォールの調整
ufwを調整する。
```bash
sudo ufw allow 80
```
```bash
sudo ufw allow 443
```

## 証明書の発行
Let's EncryptからSSL証明書を取得する。これがないと他のMisskeyサーバーとは連合できない。  
Certbot(Let's encrypt)とCloudflareプラグインをインストールする。  
```bash
sudo apt install -y certbot python3-certbot-dns-cloudflare
```
```bash
mkdir /etc/cloudflare
```
```bash
nano /etc/cloudflare/cloudflare.ini
```
`cloudflare.ini`には、次を記述する。
- 自身がCloudflareに登録しているメールアドレス(ここではhoge@example.com)
- [先ほど](#apiキーの取得)控えたAPIキー
```ini
dns_cloudflare_email = hoge@example.com
dns_cloudflare_api_key = fugapiyo
```
保存し、次のコマンドを実行する。
```bash
sudo chmod 600 /etc/cloudflare/cloudflare.ini
```
後ろのexample.tldは自身のCloudflareに登録しているドメインに変更しておくこと。
```bash
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/cloudflare/cloudflare.ini --dns-cloudflare-propagation-seconds 60 --server https://acme-v02.api.letsencrypt.org/directory -d example.tld -d *.example.tld
```
発行に成功したら`Successfully received certificate.`と表示され、証明書の保存されたパスも出力されるのでメモしておく。  

## CherryPickのインストール
gitでCherryPickのソースコードをサーバー側に複製する。  
```bash
git clone -b master https://github.com/kokonect-link/cherrypick.git --recurse-submodules
```
cherrypickに移動する。
```bash
cd cherrypick
```
corepackを有効化しておく。Ubuntuなどの場合、標準でcorepackも付属している。  
```bash
sudo corepack enable
```
CherryPickの設定ファイルを作成する。
```bash
cp .config/example.yml .config/default.yml
```
設定ファイルを編集する。
```bash
nano .config/default.yml
```
変更すべきは以下の箇所。
- `setupPassword`: 初期設定時のパスワード。設定しておかないと乗っ取られるリスクがある。デフォルトではコメントアウトされているので、外しておくこと。
- `url`: 自身のサーバーに使いたいURLに変更する。**運用開始後に変更してはならない。** 末尾に`/`をつける。
- `db`: `db`に[データベースの作成](#データベースの作成)で作成したDB名、`user`にロール名、`pass`にパスワードを設定する。

終わったら保存する。

依存関係をインストールする。
```bash
NODE_ENV=production pnpm install --frozen-lockfile
```
初期状態では、`Corepack is about to download (ここにURL)`と聞かれるので、Yを入力してEnterを押して次へ。  
終わったら、ビルドする。
```bash
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=3072 pnpm run build
```
メモリが潤沢にある環境では`NODE_OPTIONS=--max-old-space-size`は外して良い。  
DBを初期化する。  
```bash
pnpm run init
```

## nginxの設定
今の状態で起動したところでWebからは見えないので、nginxを設定する。
```bash
sudo nano /etc/nginx/conf.d/cherrypick.conf
```
下記のconfigをコピペして、以下の箇所を変更する。  
- `server_name`: 自身がサーバーのURLとして設定したいURL(`.config/default.yml`の`url`に記述したURLから末尾の`/`を取り除く)
- `ssl_certificate`: 証明書発行時に保存した`fullchain.pem`がある方のパス
- `ssl_certificate_key`: 証明書発行時に保存した`privkey.pem`がある方のパス
```nginx
# For WebSocket
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache1:16m max_size=1g inactive=720m use_temp_path=off;

server {
    listen 80;
    listen [::]:80;
    server_name example.tld;

    # For SSL domain validation
    root /var/www/html;
    location /.well-known/acme-challenge/ { allow all; }
    location /.well-known/pki-validation/ { allow all; }
    location / { return 301 https://$server_name$request_uri; }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name example.tld;

    ssl_session_timeout 1d;
    ssl_session_cache shared:ssl_session_cache:10m;
    ssl_session_tickets off;

    # To use Let's Encrypt certificate
    ssl_certificate     /etc/letsencrypt/live/example.tld/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.tld/privkey.pem;

    # To use Debian/Ubuntu's self-signed certificate (For testing or before issuing a certificate)
    #ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
    #ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;

    # SSL protocol settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Change to your upload limit
    client_max_body_size 80m;

    # Proxy to Node
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_http_version 1.1;
        proxy_redirect off;


        # For WebSocket
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        # Cache settings
        proxy_cache cache1;
        proxy_cache_lock on;
        proxy_cache_use_stale updating;
        proxy_force_ranges on;
        add_header X-Cache $upstream_cache_status;
    }
}
```
保存したら、次のコマンドでエラーが出ないか確認する。
```bash
sudo nginx -t
```
特に何も出力されなければ次へ、出力されれば出力の指示に従って修正する。  
何も出力されなければ、nginxを再起動する。  
```bash
sudo systemctl restart nginx
```
```bash
sudo systemctl status nginx
```
activeならOK。

## systemdの設定
systemdで起動できるように設定する。  
次のコマンドを実行する。  
```bash
sudo nano /etc/systemd/system/cherrypick.service
```
以下をコピペして適宜改変する。  
- WorkingDirectoryは自身がgit cloneしたディレクトリ(通常なら/home/{作業用ユーザー}/cherrypick)
```
[Unit]
Description=CherryPick daemon

[Service]
Type=simple
User=esurio
ExecStart=/usr/local/bin/npm start
WorkingDirectory=/home/esurio/cherrypick
Environment="NODE_ENV=production"
Environment="LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2"
TimeoutSec=60
StandardOutput=journal
StandardError=journal
SyslogIdentifier=cherrypick
Restart=always

[Install]
WantedBy=multi-user.target
```

保存して次を実行する。  
```bash
sudo systemctl enable cherrypick
```
```bash
sudo systemctl start cherrypick
```
実行して少し経ってから、次を実行してactiveならOK。
```bash
sudo systemctl status cherrypick
```

あとはURLにアクセスして初期設定を済ませる。

## 付録
### nanoの基本的な使い方
- 保存: `Ctrl + O`
- 退出: `Ctrl + X`

### viの基本的な使い方
- 入力モードに切り替え: `i`
- 閲覧モードに切り替え: `Esc`
- 保存: `:w`
- 退出: `:q`
- 保存して退出: `:wq`
- 強制退出: `:q!`
