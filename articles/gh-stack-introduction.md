---
title: "GitHubにスタック型プルリクエストが登場。gh stackでPRを分割して積み上げよう"
emoji: "🔗"
type: "tech"
topics: ["github", "git", "cli"]
published: true
publication_name: ubie_dev
---

GitHubでPRを細かく作り、前のブランチに対して数珠つなぎのようにPRを作る私が大歓喜！ GitHubに「スタック型プルリクエスト（Stacked pull requests）」が来ました💐

## 4行まとめ

- 大きな変更を順序付きの小さなPR群に分割して扱える「スタック型プルリクエスト」がGitHub公式で登場
- `gh extension install github/gh-stack`でCLIから使える
- `gh stack sync`でスタックのrebase, pushまで一発でできる
- AIエージェント用に`gh-stack`スキルがあるので自然言語でスタックPRの操作が可能

## 巨大な1つのプルリクエストは悪

AIエージェントでの開発が前提の現在、AIエージェントは素早く多くのコードを生み出します。気をつけないと、1つのプルリクエストに多くの開発が盛り込まれ、巨大なプルリクエストになってしまいます。1つの巨大なプルリクエストは、レビューをするのが難しく、また後からコード差分を追うときにもわかりづらいものです。確かにAIエージェント時代にはコードレビューをしないという考え方もありますが、だからと言ってPRを巨大にしていいという理由にはならないと私は考えています。

![数珠つなぎPRの課題](/images/gh-stack/huge-pr-problem.png)

### プルリクエストを細かめの粒度で分割し、数珠つなぎにする

私はこれを解決するために、プルリクエストが巨大になりそうであれば、意味のある細かめの粒度に分割し、数珠つなぎのようにプルリクエストをつなげていました。

例として、ユーザーのお気に入り機能を追加するAPIを実装するとき、次のように3つのPRへ分割します。

■ 1つめのPR: お気に入りを保存するテーブルを追加するマイグレーション
■ 2つめのPR: そのテーブルを操作するRepository（DBアクセスをまとめる層）の実装・テスト
■ 3つめのPR: Repositoryを呼び出す「お気に入り登録・解除」APIエンドポイントの実装・テスト

2つめのPRは1つめで追加したテーブルがないと実装できず、3つめのPRは2つめで実装したRepositoryがないと呼び出せません。1つめのPRのブランチに対して2つめのPRを出し、2つめのPRのブランチに対して3つめのPRを出す、というようにPRを数珠つなぎのようにつなげています。こう分割すると、レビュアーは「まずテーブル定義が妥当か」「次にRepositoryの実装が妥当か」「最後にAPIの実装が妥当か」と、層ごとに焦点を絞ってレビューできます。1つのPRに全部詰め込んでいたら、テーブル設計へのコメントとAPI実装へのコメントが入り混じり、収拾がつかなくなります。

![プルリクエストを細かく分割して数珠つなぎにする概念図](/images/gh-stack/stacked-pr-concept.png)

これは例なので、RepositoryとAPIの実装はコード量によっては1つのPRにまとめることもできるでしょうが、少なくとも1つ目のマイグレーションのようなPRは私はほとんど分割してPRを出しています。

ちなみに、「数珠つなぎ」は勝手に私が命名した概念で、みなさんがどう呼んでいるのかはわかりません笑

また、同様の理由で、コミットも乱雑に巨大な1コミットをするのではなく、意味のある細かい粒度で分割するべきだと考えており、細かい粒度でコミットするためのエージェントスキルを用いています。

## 手動の数珠つなぎのPRの辛さ

数珠つなぎのPRは良いのですが、いくつか問題があります。

■ 数珠つなぎになるようPRの向き先を設定するのが面倒

branch1 -> branch2 -> branch3のように、PRのマージ先を都度設定しないといけません。「あれ？今何個目のPRだっけ？」といちいち悩むのが面倒です。

■ どのPRが関係しているのか、自分でdescriptionに書く必要がある

数珠つなぎの情報を知っているのは私だけなので、「このPRは3つのPRのうちの1つめです。他にPR2とPR3があります」というように、自分でdescriptionに書く必要があります。

■ 前のPRの変更を、さらに上位のPRに反映するのが面倒

PR1でレビューが有り、コードを更新したとします。今度はその更新をPR2, PR3, ...と後続のPRに反映する必要があります。私はrebase派なのですが、都度rebaseし、force pushしていくのが面倒です。

これらを解決するのが、GitHub公式が公開した「スタック型プルリクエスト」です。

![数珠つなぎPRの課題](/images/gh-stack/pr-stack-problem.png)

## スタック型プルリクエストとは

スタック型プルリクエストは、1つの大きな変更を「土台のブランチ→その上に積んだブランチ→さらにその上のブランチ…」という順序で積み重ね、各層を1つのPRとして扱う仕組みです。私が勝手に「数珠つなぎプルリクエスト」とよんでいたものです。

```
main (trunk)
 └── PR #1 (base: main)
  └── PR #2 (base: PR #1のブランチ)
   └── PR #3 (base: PR #2のブランチ)
```

ポイントは、各PRのdiffが1つ下の層との差分だけになることです。PR #2を開いても、PR #1で入れた変更は混ざりません。レビュアーは層ごとの小さな差分だけを追えばよく、全体像はスタック表示で俯瞰できます。

GitHubの公式アナウンスでは、次のように説明されています。

> Stacked pull requests break large changes into small, reviewable pull requests. They're an ordered series of pull requests that each represent focused layers of your change.
>
> （意訳：スタック型プルリクエストは、大きな変更を小さくレビュー可能なプルリクエストに分割します。それぞれが変更の焦点を絞った層を表す、順序付きのプルリクエストのシリーズです）
>
> https://github.blog/changelog/2026-07-30-stacked-pull-requests-are-now-in-public-preview/

GitHubのWeb画面・CLI・モバイルアプリ・GitHub Copilotエージェントなどで利用できます。

## スタック型プルリクエストがGitHubのUIで動いている様子

スタック型プルリクエストで作成したPRは、次のように画面の上部に、複数のPRがスタックとして表示されます。どのPRがどの層にいるのかが一目瞭然です。

![](/images/gh-stack/pr-review.png)

これは画面下部のマージボタン付近にも表示されます。また、スタックされたPRを一括でマージするボタンも表示されます。

![](/images/gh-stack/pr-comment.png)

## 導入方法

CLIからは`gh`の`github/gh-stack`エクステンションをインストールすることで利用できます。

```bash
gh extension install github/gh-stack
```

確認コマンド

```bash
$ gh extension list
gh stack github/gh-stack v0.1.0
```

現場では、Claude Code、CodexなどのAIエージェントからスタックPRを操作することも多いでしょう。専用のスキルがあるのでインストールしておくことを推奨します。インストールコマンドは次のとおりです。

```bash
gh skill install github/gh-stack
```


確認コマンド

```bash
$ gh skill list | grep gh-stack
gh-stack  claude-code     user  github/gh-stack
gh-stack  codex           user  github/gh-stack
gh-stack  cursor          user  github/gh-stack
```

なお、`gh skill install` はGitHub公式のスキルインストール方法です。

https://zenn.dev/ubie_dev/articles/gh-skill-install-agent-skills

## 実際に使ってみる

基本的にはスキル経由でAIエージェントから操作することになると思いますが、せっかくなのでコマンドの挙動を見てみましょう。

手元のポートフォリオリポジトリで、検証用の軽量な変更を3層のスタックに分けて作ってみました。

### 1層目を作る

`gh stack init`にブランチ名を渡すと、スタックの起点となるブランチが作られます。

```bash
$ gh stack init demo/gh-stack-step-1
✓ Created stack: main ← demo/gh-stack-step-1
  You're on demo/gh-stack-step-1 (top of stack).
```

ここでファイルを1つ追加してコミットします。

### 2層目・3層目を積み上げる

`gh stack add`で2層目・3層目を追加します。

```bash
$ gh stack add demo/gh-stack-step-2
✓ Created and checked out branch "demo/gh-stack-step-2"

$ gh stack add demo/gh-stack-step-3
✓ Created and checked out branch "demo/gh-stack-step-3"
```

2層目・3層目でもそれぞれファイルを追加してコミットしておきます。

### PRをまとめて作成する

`gh stack submit`で、3つのブランチをまとめてpushし、PRを一括作成できます。`--auto`を付けるとPRタイトル入力用のエディタが開かず、タイトルを自動生成してくれます。なお作成されるPRはドラフト状態です。`--open`を付けるとready for reviewの状態で作成できます。実行結果は次のとおりです。

```bash
$ gh stack submit --auto
Checking stack state...
Pushing to origin...
✓ Created PR #452 for demo/gh-stack-step-1
✓ Created PR #453 for demo/gh-stack-step-2
✓ Created PR #454 for demo/gh-stack-step-3
✓ Stack created on GitHub with 3 PRs (stack #455)
✓ Pushed and synced 3 branches
```

3回`git push`して3回PRを作る手間が1コマンドで済むのは嬉しいです。実際に作成されたPRは次のとおりです。

https://github.com/tonkotsuboy/kano-portfolio/pull/452

https://github.com/tonkotsuboy/kano-portfolio/pull/453

https://github.com/tonkotsuboy/kano-portfolio/pull/454

![GitHub上でPR #452〜454がスタックとして表示されている画面](/images/gh-stack/pr-3.png)
_GitHub上でPR #452〜454がスタックとして表示されている画面_

## スタック状態を確認する（`gh stack view`）

`gh stack view`で、スタック全体の状態を確認できます。矢印でスタックPR間を選択し、Enterで該当PRのブランチにチェックアウトすることもできます。

![](/images/gh-stack/gh-stack-view.png)

`gh stack up` / `gh stack down` / `gh stack top` / `gh stack bottom` などでも移動できますが、私は一度リスト表示して移動するのが好きでした。

## スタックPRをまとめてpush（`gh stack push`）

`gh stack push`をすると、スタックしたPRをまとめてプッシュできます。

![gh stack pushで3つのブランチがまとめてpushされた実行結果](/images/gh-stack/gh-stack-push.png)

## スタックPRを勝手にrebaseしてくれる（`gh stack sync`）

私が思わず唸った機能です。PR1の変更をした後、その変更をスタックされたPR2, PR3に反映したいケースは頻出します。手動の場合は、都度rebaseしてforce pushしたのですが、スタック型プルリクエストなら一発です。

`gh stack sync` を実行すると、次の操作が一括で行われます。

- origin/mainをfetchし、手元のmainブランチを最新化してくれます
- スタック全体を下から順にrebaseしてくれます。言い換えれば、1つめのPRに変更を加えた場合、2つめ、3つめのPRにも自動で反映されます
- スタックPRをすべてpushし、GitHub上のPR情報も同期します

さらに`--prune`オプションを付けると、マージ済みPRのローカルブランチを自動で削除してくれます。

![gh stack syncでfetch・rebase・pushが一括実行された実行結果](/images/gh-stack/gh-stack-sync-prune.png)

大丈夫ですかこの神機能・・・合法ですか！？

## AIエージェントで自然言語で操作する

実際の現場では、Claude CodeやCodex経由で操作することが多いでしょう。前述のgh-stackスキル（`gh skill install github/gh-stack`）を使えば、細かいコマンドを覚えず、自然言語で操作できます。

- スタックPRを作って
- スタックPRを全部rebaseして

といった命令で、gh-stackのコマンドが実行されます。うまくいかないときは「gh-stackスキルを使って」と冒頭につけるとよいでしょう。私はh雨庵なので基本的に「gh-stackスキルを使って・・・」と言っています。

## 最後に

私はPRやコミットを細かくするのが好きです。一方で、PRのスタックにはdescriptionの記載、rebase、targetの指定など、面倒くさい作業が多くありました。GitHubがスタック型プルリクエストとしてこの課題を解決してくれたとき、「この操作、私以外にもやってたのか！」と正直おどろきました。AIエージェント時代に大量のPRが作られる現代において、スタック型プルリクエストは必須の機能。ぜひ皆さんもgh stackコマンドをインストールして使いましょう！

![結論](/images/gh-stack/conclusion.png)

@[tweet](https://x.com/tonkotsuboy_com/status/2083007010933756349)


https://github.blog/changelog/2026-07-30-stacked-pull-requests-are-now-in-public-preview/
