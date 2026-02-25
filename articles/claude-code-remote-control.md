---
title: "Claude Code Remote Controlが登場。旅行先や子育てしながらスマホでローカルコードの開発をしよう"
emoji: "👴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:  ["typescript", "javascript", "tsgo"]
published: true
publication_name: ubie_dev
---


旅行中にふと暇になった時間、子どもが寝静まるのを待っている時間、電車に乗っている時間。その隙間時間に、「ん゛あ゛あ゛あ゛あ゛！゛開゛発゛が゛し゛た゛く゛て゛た゛ま゛ら゛な゛い゛よ゛お゛お゛お゛！゛！゛！゛」と悶々とする時間がありませんか？ 私は、ありまぁす！

本日登場したClaude CodeのRemote Control機能を使えば、ローカルのパソコンで動いているClaude Codeのセッションを、スマホから確認したり、追加指示を出したりできるようになります。


## 利用シーン

## 概要


## 設定方法

公式ドキュメントがどこよりも詳しいので、公式ドキュメントを参照することをおすすめします。

https://code.claude.com/docs/en/remote-control

筆者は、全セッションでリモートコントロールを有効にしたいので、次の手順を使いました。
Claude Codeを起動後、`/config`コマンドを実行を実行すると、「Enable Remote Control for all sessions」という項目があります。これを`true` にしておきます。


![](img.png)


- 参考: https://code.claude.com/docs/en/remote-control#enable-remote-control-for-all-sessions


スマートフォンにインストールしたClaudeアプリから、該当セッションを確認できるようになります。


# 実際に動作している様子


## パソコンの作業をスマートフォンで確認する

次の例では、パソコンからあるリポジトリにおいてHTMLの修正を依頼している様子です。「メインのテキストエリアを中央に表示して。」と命令すると、その作業が行われます。


キャプチャーはPCです。

![img_2.png](img_2.png)

iPhoneのClaudeアプリを確認すると、"ローカルの"パソコンのセッションの様子が確認できます。リモート環境ではなく、普段作業しているパソコンのセッションが見れていることがポイントです。

![iphone_session.png](iphone_session.png)


パソコンとスマートフォンを同時に並べるとこんな感じです。

![img_1.png](img_1.png)


## スマートフォンから追加の指示を出す

これが筆者の一番うれしいポイント。スマートフォンから作業中のセッションに対して追加の指示を出せます。iPhoneから「あと、背景明るくしてほしい」と依頼します。すると、同期されたPCのセッションで作業が行われます。すごすぎる。

（動画）









