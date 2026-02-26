---
title: "Claude Code Remote Controlが登場。ソファでも移動中でも、ローカルセッションをスマホから動かす"
emoji: "📲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["claudecode", "ai"]
published: true
publication_name: ubie_dev
---


旅行中にふと暇になった時間、子どもが寝静まるのを待っている時間、電車に乗っている時間。その隙間時間に、「ん゛あ゛あ゛あ゛あ゛！゛開゛発゛が゛し゛た゛く゛て゛た゛ま゛ら゛な゛い゛よ゛お゛お゛お゛！゛！゛！゛」と悶々とする時間がありませんか？ 私は、ありまぁす！

本日登場したClaude CodeのRemote Control機能を使えば、ローカルのパソコンで動いているClaude Codeのセッションを、スマホから確認したり、追加指示を出したりできるようになります。

![](/images/claude-code-remote-control/pc-iphone-side-by-side.jpg)

## 概要

Remote Controlは、ローカルPCで動いているClaude Codeのセッションを、Claudeモバイルアプリ（[iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) / [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)）やClaudeデスクトップアプリから操作できる機能です。

セッションはあくまでローカルのPC上で動き続けるため、ファイルシステムやMCPサーバー、プロジェクト設定はそのまま利用できます。PCでカリッカリにチューニングした設定や、いろんなところから集めた秘蔵のスキルがそのまま活用できるということです。
会話はすべてのデバイスでリアルタイムに同期され、ネットワーク切断やスリープ後も自動で再接続します。

たとえばこんな使い方ができます。

- デスクで始めた作業の進捗を、ソファや別の部屋からスマホで確認する
- 長時間かかるタスクを走らせたまま外出し、スマホから追加の指示を出す
- 電車の中や子どもの寝かしつけ中に、手元のスマホでコードの修正を依頼する


## 設定方法

公式ドキュメントがどこよりも詳しいので、公式ドキュメントを参照することをおすすめします。

https://code.claude.com/docs/en/remote-control

執筆時点（2025/02/25）ではresearch previewとして提供されており、Maxプランで利用できます。Team・Enterpriseプランでは利用できません。私は[会社ではClaude Code使い放題のAPIキー](https://x.com/tonkotsuboy_com/status/2014603660463104447)ですが、まだ使えなかったので、個人のMaxプランで試しました。

おすすめの設定はつぎのとおり。Claude Codeを起動後、`/config`コマンドを実行すると、「Enable Remote Control for all sessions」という項目があります。これを`true` にしておきます。こうすることで、今後の全セッションが、 スマートフォンにインストールしたClaudeアプリから確認や追加指示ができるようになります。

![](/images/claude-code-remote-control/config-enable-remote-control.png)

- 参考: https://code.claude.com/docs/en/remote-control#enable-remote-control-for-all-sessions

# 実際に動作している様子

## パソコンの作業をスマートフォンで確認する

次の例では、パソコンからあるリポジトリにおいてHTMLの修正を依頼している様子です。「メインのテキストエリアを中央に表示して。」と命令すると、その作業が行われます。

キャプチャーはPCです。

![](/images/claude-code-remote-control/pc-session.png)

iPhoneのClaudeアプリを確認すると、"ローカルの"パソコンのセッションの様子が確認できます。リモート環境ではなく、普段作業しているパソコンのセッションが見れていることがポイントです。

![](/images/claude-code-remote-control/iphone-session.png =320x)


パソコンとスマートフォンを同時に並べるとこんな感じです。

![](/images/claude-code-remote-control/pc-iphone-side-by-side.jpg)

## スマートフォンから追加の指示を出す

これが私の一番うれしいポイント。スマートフォンから作業中のセッションに対して追加の指示を出せます。iPhoneから「あと、背景明るくしてほしい」と依頼します。すると、同期されたPCのセッションで作業が行われます。すごすぎる。

![](/images/claude-code-remote-control/sp_1.png =320x)

![](/images/claude-code-remote-control/sp_2.png =320x)

# 接続の仕組みとセキュリティ

Remote Control利用時、ローカルのClaude Codeは**アウトバウンド（外向き）のHTTPSリクエストのみ**を発行します。つまり、PCから外部のサーバーへ通信するだけで、外部からPCへの接続（インバウンド）を受け付けるポートは一切開きません。自分のPCが直接インターネットに晒されることはありません。

スマホとPC間のメッセージは、AnthropicのAPIサーバーを中継してやりとりされます。通信はすべてHTTPS（TLS）で暗号化されており、通常のClaude利用時と同等のセキュリティです。認証に使われるトークンは用途ごとに分かれた短命なもので、一定時間で自動的に無効化されます。同時に接続できるリモートセッションは1つのみで、ターミナルを閉じたり`claude`プロセスを停止するとセッションが終了します。

- 参考: https://code.claude.com/docs/en/security


# さいごに

私は昔、[Devinを使ってソファーに座りながらSlack上からDevinに作業指示を出していました](https://zenn.dev/ubie_dev/articles/devin-for-test)。しかし、Claude Codeがカリカリに育った今、リモート環境ではなく育ちきったClaude Codeをローカルでそのまま動かしたいと考えていました。巷には、サードパーティーツールを使ったり、なにか難しい設定をしてClaude Codeをスマートフォンから操作できるようにする方法がありますが、私はどれもやってきませんでした。設定が面倒くさかったのと、どうせClaude Codeがそのうち公式で対応するだろうと思っていたからです。

今回のRemote Controlの登場で、ついに夢が叶いました。これからはスキマ時間に贔屓の野球チームの結果を見たりジャンププラスでチェンソーマンの続きを読む前に、ローカルのClaude Codeに仕事を依頼します。

@[tweet](https://x.com/twitter/status/2026461458960363976)

3/11に、Claude Codeのイベントを行います。ぜひご参加ください。

鹿野さんに聞く！AIは速いのに、人間が遅い？Claude Codeをさらに加速させる私の推しツール

https://findy.connpass.com/event/384179/
