---
title: Asahi Linuxを入れて消す
published: 2024-06-14
description: 戻すのが面倒
tags: [Linux, mac]
category: 技術
draft: false
---

# 要約
入れるときは`curl`でシェルスクリプト、消すときは`diskutil`

# 本編

今年の4月くらいにM2 Macbook Airを買ったのだけども、こいつがなかなか使い勝手がいい。  
軽いしバッテリー持ちも前のWindowsラップトップより良く、画面もきれい。  

しかし問題がある。そう、ARMプロセッサなのである。  
Intel Macならインターネットから好きなisoをダウンロードして適当にインストールで済むのだけど、ARMだとそうもいかない。  

そこで、Asahi Linuxである。  
Asahi LinuxはApple Silicon MacにLinuxを移植することを目的としたプロジェクト及びコミュニティのことを指す。[参考](https://asahilinux.org/about/)  

非公式プロジェクトにUbuntu Asahi Linuxもあるが、今回は公式のFedora Asahi Linuxをインストールする。

# 必要なもの

- Apple Silicon Mac(M2 Macbook Airが一番サポートされている)
- Timemachineのバックアップ
- 度胸

# インストール

ホームページを開くと[コマンドが貼ってある](https://asahilinux.org)のでコピーしてターミナルにペーストする。  
  
```sh
curl https://alx.sh | sh
```  
  
そうするとまずsudoパスワードを聞かれるので入力する。  
後はプロンプトの指示に従ってインストールを進める。  
注意しなければならないのは、パーティションの割り当てを変更する際に入力する数字は、大きければ大きいほどmacOSに割り当てるパーティションが大きくなる。macOSを引き続き使い続けるかどうか考慮して進めよう。  

終わると再起動するよう求められるので長押しして再起動する。  
起動ディスク一覧にAsahi Linuxが追加されていれば成功だ。

# アンインストール

まずは長押ししながら再起動して、ディスクユーティリティを開く。  
First Aidを実行しておく。  

次に、MacintoshHDを起動ディスクに指定し、macOSを起動して、ターミナルを開く。  
`diskutil list`を叩き、Asahi Linux関連のパーティションとコンテナを確認する。 
`Linux File System`などとあったらそれを削除する。 

```sh
$ diskutil list

/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk4         346.1 GB   disk0s2
   3:                 Apple_APFS Container disk2         2.5 GB     disk0s3
   4:                        EFI EFI - FEDOR             524.3 MB   disk0s4
   5:           Linux Filesystem                         1.1 GB     disk0s5
   6:           Linux Filesystem                         24.3 GB    disk0s6
                    (free space)                         119.9 GB   -
   7:        Apple_APFS_Recovery Container disk3         5.4 GB     disk0s7
```

```sh
diskutil eraseVolume free free disk0s5
diskutil eraseVolume free free disk0s4
diskutil apfs deleteContainer disk0s3
diskutil eraseVolume free free disk0s3 //コンテナ消したらボリュームも消すこと
diskutil apfs resizeContainer disk0s2 0
```
  
最後に、もう一度`diskutil list`を叩いて以下のようになってればOK。
```sh
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk3         494.4 GB   disk0s2
   3:        Apple_APFS_Recovery Container disk2         5.4 GB     disk0s4
```  

# 反省点

**Timemachineで予めバックアップを取れ！**  
以上。