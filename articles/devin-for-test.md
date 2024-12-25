---
title: "Devin AIにテストを丸ごと書かかせてCIがパスするまで作業してもらう方法"
emoji: "💐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["LLM", "AI"]
published: true
---

DevinというAIチームメイトにテストを書いてもらったら、開発効率が大きく向上した話です。

https://devin.ai/

Devinは、ソフトウェア開発におけるタスクを自動化・効率化してくれるAIプラットフォームです。2024年12月に正式リリースされました。 私が所属しているUbieにも先日導入されました。様々な作業ができますが、あるリポジトリで不足しているテストを書いてもらったところ、その便利さに感動して椅子から転げ落ちました。

## テストの作成をSlackで依頼する

Slackで「これこれのテストを書いてほしい」と依頼すると、Devinくんがテストコードを生成し、GitHubに新しいPRを作ってくれます。

依頼例は次のとおりです。

```markdown
こんにちは、 @Devin 以下の仕事をして
- ubie-inc/リポジトリ名 repo にアクセスして
- （テスト対象のパス） のテストを書いて
- 次のテストの書き方を参考にして
  - foo/index.test.tsx
  - bar/index.test.tsx
- 実装追加した変更で PR を作って。
- PRのタイトルとサマリは日本語にして。
- ブランチ名は、testから始まるようにして
- コミットメッセージに Co-Authored-By: GitHubアカウント名 <メールアドレス> を含めて
- CIの「Lint and Test 」では、warningも出ないように修正して
- import '@testing-library/jest-native/extend-expect'; は含めないで
- import React from 'react'; は含めないで
- PRはdraftで作って
```

実際にSlackに依頼した例

![](/images/ai-for-test/request-in-slack.png)

### 依頼する際のポイント

プロジェクト内のお作法がわかるようなテスト事例をいくつか渡してあげるとスムーズです。事例が無い場合、ゼロから作ってもらうことも可能ですが、Devinとのやりとりに時間がかかってしまうので、最初の1, 2個目のテストは自前で作ってしまう方が早いです。

オススメはGitHub Copilot in VS Code、Cursor、JetBrains AI Assistantといった◯◯を使い、プロジェクト全体を解読してもらいながらテストを書いてもらう方法です。

## 2. 作業が開始するので更新を待つ

Devinの作業が開始します。更新をしばらくまちます。

![](/images/ai-for-test/creating.png)


## 3. CIのfailは自動で修正してくれる

個人的に激アツなのは、CIが失敗したときに成功するまで自動的に修正してくれるところです。これにより、わざわざ「CIが落ちているので修正して」といった追加の依頼をする必要はありません。

また、状況によっては失敗するCIをあえて無視したいケースもあるでしょう。そういった場合には、「◯◯のCIが落ちるのは無視して」とSlack上やDevin上でコメントすれば、該当のCIの失敗だけを無視してくれるようになります。


## 4. 追加の作業依頼を行う

作業が終わると実際にPRができますが、Devinの作業中やPRの作成後に追加の作業依頼をできます。依頼方法はいくつかあります。

### 方法① Slackのコメントから依頼する

最も手軽なのは、Slackのコメントから修正を依頼する方法です。例えば次の例では、不要な `import`文を削除してもらうよう依頼しています。Devinがコメントを拾って、作業を開始してくれます。

![](/images/ai-for-test/slack-request.png)

個人的にはこれがとても便利で、例えば私はリビングでテレビを見ながら、iPhoneのSlackでDevinに作業修正を依頼していました。

### 方法② GitHubのPR上から依頼する

GitHubのPR上から作業を依頼する方法です。PRの差分箇所に作業依頼をコメントすると、これもDevinが拾って作業をしてくれます。

![](/images/ai-for-test/pr-request.png)


### 方法③ DevinのUI上から依頼する

DevinのUI上から作業を依頼する方法です。Devinが作業を開始するとセッションが発行され、 `https://app.devin.ai/sessions/セッションID` でアクセスできます。Editorタブでエディターを開けるので、編集をしたい箇所をDevin上で参照して作業依頼を追加できます。次の例では、テストのモック方法の修正を依頼しています。

![](/images/ai-for-test/devin-request.png)


# フィードバックの知見が溜まる

大変便利なのが、各種作業依頼のフィードバックをDevinが学習して覚えてくれていることです。例えば、「モックの際は`jest.spyOn()`を優先的に使ってください」と指定していたとします。すると、その命令をDevinがリポジトリごとに「Knowledge」として保存され、次回の作業時に使ってくれるのです。

![img.png](/images/ai-for-test/knowledge.png)

Knowledgeは自由に追加・修正・削除が可能なので、毎回長ったらしいプロンプトを書く手間が省けますし、リポジトリごとのコードの方針も統一ができます。


https://docs.devin.ai/onboard-devin/knowledge


# もっと改善されると嬉しいところ

 テストの内容が正しいかどうか、ドメイン知識や最終チェックにはエンジニアがPRをチェックする必要があります。当初、非エンジニアに不足しているテストを追加・量産してもらい、リリースまで実施してもらおうとしていたのですが、成果物の内容的にはまだ厳しいと言わざるを得ない場面がありました。とは言え、うまく行ってないところを修正すればいいだけなので、ゼロからテストを書くよりは格段にラクです。


作業速度についても、もっと改善されると嬉しいなとは感じました。文中でも書いたように、ゼロから複雑な作業を依頼するよりは、ある程度の事例や指示を明確に出した方が、最終的な仕上がりは早くなる印象です。


# 面倒なテストはAIに任せよう

Slackから気軽に作業完結まで依頼できる点、CIがパスするまで修正を繰り返す点、 コードの一部を参照して修正依頼が可能な点、 フィードバックはKnowledgeに溜まっていく点など、Devinは私のテスト作成作業を大きく改善してくれました。

https://x.com/tonkotsuboy_com/status/1871777460330938846

なお、私の所属しているUbieは、Devinのような新しい生成AIを実務に取り入れています。社内LLM「dev爺」では各種最新LLMモデルが使い放題になっているため、金銭的な負担もなく好きなだけAIを試せます。

興味のある人は、ぜひUbieで一緒に楽しい開発をしていきましょう。

https://recruit.ubie.life/
