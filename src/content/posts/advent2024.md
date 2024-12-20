---
title: 2024年の私とFediverse
published: 2024-12-12
description: お好み
tags: [Fediverse, notectl]
category: その他
draft: false
---

この記事は、[Fediverse Advent Calender 2024](https://adventar.org/calendars/10051)の12日目です。
2024年の私とFediverseということで、私とFediverseに関する話をいくつか。  

# 2024年ダイジェスト
- MisskeyとそのフォークにいくつかPR(MR)を出す
- 自分でフォークして色々改造した話
- OSCなどのイベント
- notectl作った

## MisskeyやSharkeyなどにPull Requestをだした
MisskeyやSharkeyなどにPull Requestを出しました。とりあえず出したやつはマージされています。  
Misskeyでは、HTMLメールにCSSが適用されていない問題を修正し、Sharkeyではi18nが不十分だった部分の修正をしました。  
CherryPickには、自分のフォークしていたものがcherry-pickされてコントリビュータという扱いになりました。
今後もぼちぼち手が空いている時間にPull Requestを投げていきたいです。

## 自分でフォークして色々改造した話
メインの自鯖はCherryPickを使っていたのですが、kokonect.linkだけで使える機能があり、羨ましいと思ったので自分でフォークして色々改造することにしました。  
主な変更点としては、`toot:indexable`や`fedibird:searchableBy`への対応、サイコロウィジェットや検索ウィジェットの追加、Elasticsearch対応の追加、パスワードのハッシュアルゴリズムをargon2に変更するなどです。  
サイコロウィジェットや検索ウィジェットはCherryPick 4.12.0にcherry-pickされているため、他のCherryPickサーバーでも使用可能です。  
個人的に改造した中で気に入ってるのは、オフラインスクリーンの配色変更です。CherryPick Darkというテーマをベースに変更しました。  
元々この改造はyojo-artでやる予定だったのですが、私がyojo-art開発を降りたので実装できた感じです。(なんで降りたかは本人に聞く度胸があれば聞いてください)  

## OSCなどのイベント
今年はOSC春/秋 東京、OSC京都、KOFに参加しました。    
このうちOSC春以外は出展者側です。  
FediLUGや分散型SNSユーザー有志、OSC秋ではFedify/Holloのブースに参加していました。  
出展者側は出展者側で色々と楽しくていいですね。  

## notectl作った
これはおまけみたいなものですが、Misskeyにはtootctlのような管理ツールがないのでメンテナンスが割と不便です。
なので外付けで使えるものを作りました。  
リポジトリは[こちら](https://github.com/1673beta/notectl)。  
機能はdocs/featureを見てください。  
developブランチではDocker対応もしています。  
CI/CDの整備などが追いついていないため、ぼちぼち改善予定です。  
ライセンスはMITとApache 2.0のデュアルライセンスです。  
自由に改造して遊んでください。  

# その他個人的なこと

## 体調が悪い1年
去年までの無理が祟った結果、見事に体調を崩しまくって心も崩しまくって病院がよいの1年となりました。前年比で医療費が+20万円! (2023年度: 0円)
最も、見てる額はマイナポータルのそれなので、実際は2ヶ月後にならないといくらかかったのかわからないという…

## 休学した
メンタル面がどうにも治らないし体調も悪いままなので休学しました。  
無理はするものじゃないですね。  
実質無職です。  
体調が悪かったりメンタル面が悪かったりで、外に出る時は病院か買い物に行くか特定個人に会いに行く時くらいです。  

## 2024年買ってよかったもの

1. 「良いウェブサービスを支える「利用規約」の作り方」  
本です。幼女.artの利用規約を書くときに役立ちました。(私が書きました)
2. Macbook Air M2 16GB
Windowsの使い勝手の悪さにキレて衝動買いしました。  
重量が軽いので持ち運びも楽ですしUNIX系なのでコマンドもLinuxとある程度共通しているので使いやすいです。  
3. Fitbit Sense 2
スマートウォッチです。体調管理のために買いました。  
5月にエアコンが壊れたのですが、その時に出たノジマのポイントで買いました。  
電池持ちもいいですし、自分の健康状態を把握するにはこれくらいの機能でちょうどいいなと思っています。  

# おわりに
来年はもう少し健康になりたいところです。  