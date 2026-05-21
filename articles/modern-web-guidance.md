---
title: "GoogleのModern Web Guidanceスキル登場。AIが古いCSS・JSを書く問題を解決する"
emoji: "✅"
type: "tech"
topics: ["css", "javascript", "claudecode", "ai"]
published: true
publication_name: ubie_dev
---

AIにフロントエンドのコードを書かせると、一昔前の書き方をされて困ることがよくあります。

私のAIの使い方が悪いというのは前提にありつつ、最新の書き方を選んでくれないことが何度もあります。たとえばCSSではSubgridを使ってほしい場面でわざわざ古いGridのネストを書いてくる。JavaScriptでは`Object.groupBy`で済むグループ化を`reduce`で書いてくる。さらに当たり前品質として押さえるべきアクセシビリティが漏れたり、最新のセキュリティ対策が抜けたコードを書いてくることもあります。これはサービスの品質低下に直結します。

5/19にGoogle公式がGoogle I/Oで発表した「Modern Web Guidance」のスキルを使えば、Claude CodeやCodexといったAIエージェントが最新のWeb機能で書けるようになります。フロントエンド開発をするならぜひ導入しておきたいスキルです。

![Google I/O '26のDeveloper KeynoteでModern Web Guidanceが発表された様子](/images/modern-web-guidance/google-io-2026.jpg)
_Google I/O '26のDeveloper Keynoteで発表されたModern Web Guidance_

Modern Web Guidanceの中身と、実際に私のポートフォリオサイトへ導入した結果を紹介します。

## 4行まとめ

- Modern Web Guidanceとは、ここ数年で使えるようになったモダンなWeb機能の知識をAIエージェントに注入するGoogle公式ガイド
- ユーザーエクスペリエンス・CSSレイアウト・パフォーマンス・フォーム & UI・アクセシビリティ・ブラウザ内蔵AIの6分野をチェックできる
- ブラウザ対応状況（Baseline）に連動し、対応が揃っていない機能にはフォールバックを添えて提案してくれる
- `npx modern-web-guidance@latest install`で導入

## なぜAIは古いコードを書くのか

LLM（大規模言語モデル）は過去に書かれた膨大なコードを学習しています。Webの世界には古い書き方のコードが多く積み上がっているため、何も指定しないとAIは「多数派」のコードを元に一昔前の書き方を出すことがよくあります。冒頭で挙げたSubgridや`dialog`の例はまさにこれが原因です。

> Coding agents often default to older patterns because LLM training data contains vast amounts of legacy code.
> （意訳：コーディングエージェントは、学習データに大量のレガシーコードが含まれているため、つい古いパターンを選んでしまう）
> https://github.com/GoogleChrome/modern-web-guidance#-why

Modern Web Guidanceは最新のWebプラットフォームの知識をAIエージェントに注入し、この問題を解決するよう導きます。公式はClaude Opus 4.7に「画像をもっと速く読み込ませて」という曖昧な開発依頼を75個出しました。できあがったコードがブラウザ上で正しく動くか、モダンな書き方になっているかを採点しました。Modern Web Guidanceなしだと正しく書けたのはおよそ半分（52%）。ありだと85%まで上がりました。実験の詳細は公式リポジトリのEvalsセクションにまとまっています。

https://github.com/GoogleChrome/modern-web-guidance#-evals-to-prove-this-works-well-

## 何がチェックされるのか

Modern Web Guidanceには、6つの分野（Core Disciplines）に整理されたモダンなWeb機能やユースケースが収録されています。AIエージェントはこのユースケースから「やりたいこと」を逆引きします。モダンなWeb技術で実装したり、既存のコードがモダンな書き方になっているかを点検したりできます。

■ CSSレイアウト
コンテナクエリ、Subgrid、`oklch`などのモダンな色空間、`text-wrap`の調整、行の余白（leading）のトリミングなど

■ ユーザーエクスペリエンス
View Transitions、要素の表示・非表示アニメーション、パララックススクロール、CSSの`scrollbar-color`など

■ パフォーマンス
ページの先読み（instant preloading）、INP（Interaction to Next Paint）の診断、`scheduler.yield`によるタスク分割など

■ フォーム & UI
ツールチップ向けのAnchor Positioning、Popover API、`<dialog>`、`:user-invalid`によるバリデーション、入力欄の自動サイズ調整など

■ アクセシビリティ
アクセシブルなエラー通知や、キーボードのフォーカス管理など

■ ブラウザ内蔵AI
翻訳・要約・言語検出を端末内で行うローカルAIモデルなど

### ユースケースやWeb機能の例

たとえばCSSレイアウトの機能ガイドは、次のURLで確認できます。

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md

内容は次のとおり。FlexboxやGridの使い分けについてGoogle公式の判定があるのはありがたいですね。

- 「1次元の並びならFlexbox、孫要素を祖父グリッドの線に揃えたいならSubgrid、行と列の骨格から組むならGrid」といったレイアウトモードの選び方
- `flex-basis`と`width`を同時に指定してはいけない、`auto-fit`と`auto-fill`はどう違うのか、といったつまずきポイントのDo / Don't
- そのまま使えるコード例と、機能ごとのBaseline状況

別の例では、「Optimize image priority（画像の優先順位の最適化）」のユースケースを次のMarkdownで確認できます。

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/performance/optimize-image-priority.md

中身を見ると、[Largest Contentful Paint (LCP)](https://web.dev/articles/lcp)になる画像には`fetchpriority="high"`、重要でない画像には`fetchpriority="low"`を付ける、といった実装がコード例つきで書かれています。

```html
<img
  src="/images/hero-lcp.jpg"
  alt="Main Banner"
  fetchpriority="high"
  width="800"
  height="400"
/>
```

「ここ数年でWebに増えたモダンな機能を網羅的にチェックして自分のサイトへ取り入れたい」というとき、人間が一つひとつ思い出さなくてもAIエージェントが代わりにチェックして提案してくれるということです。

## 導入方法

次のコマンドで導入できます。

```bash
npx modern-web-guidance@latest install
```

実行すると対話形式のウィザードが立ち上がり、使っているエージェントに合わせて`SKILL.md`を適切な場所へ配置できます。

![導入先のエージェントを選ぶインストールウィザード。Amp・Antigravity・Codex・Cursor・Gemini CLI・GitHub Copilot・Claude Codeなどが並ぶ](/images/modern-web-guidance/install-agent-select.png)
_インストールしている様子_

## ブラウザ対応はBaselineに連動し、古くなりにくい

モダンな機能を使うときに気になるのがブラウザ対応です。Modern Web Guidanceは[Baseline（主要ブラウザでその機能が安全に使えるようになった状態を示す、Webの共通指標）](https://web.dev/baseline)のリアルタイムなデータに連動しています。これによりブラウザの対応状況に応じて使用する機能を取捨選択できます。

> We leverage real-time compatibility data from the Baseline project so agents can dynamically adapt to current browser support and any browser support preferences.
> （意訳：Baselineプロジェクトのリアルタイムな対応データを使っているため、エージェントは現在のブラウザ対応状況や、プロジェクトごとの対応方針に合わせて動的に判断できる）

## 仕組み

AIエージェントにフロントエンドのタスクを頼むと、スキルが自動で発動するようになっています。もし発動しない場合は明示的に「Modern Web Guidanceを使ってダイアログの背景をアニメーションさせて」などと指示します。

![「Modern Web Guidanceを使って」と指示すると、スキルが読み込まれ、検索からガイドの取得まで自動で実行される様子](/images/modern-web-guidance/skill-invocation.png)
_「Modern Web Guidanceを使って」と頼むと、スキルが読み込まれ`search`から`retrieve`まで自動で走る_

AIエージェントからの依頼を受けたときにmodern-web-guidanceでは`search`と`retrieve`の2ステップが動きます。

まず`search`にクエリを投げると、関連するガイドが類似度つきで返ってきます。実行結果は次のとおり。

```bash
$ npx modern-web-guidance@latest search "animate a dialog modal backdrop"
light-dismiss-a-dialog              0.71
animate-to-from-top-layer           0.68
declarative-dialog-popover-control  0.57
```

次に、目的に合うIDを`retrieve`に渡すと、実装手順とブラウザ対応状況が書かれた本文が返ってきます。

```bash
$ npx modern-web-guidance@latest retrieve "animate-to-from-top-layer"
Elements that render in the "top layer" (like <dialog>, popover, tooltips)
have historically been difficult to animate ... Modern CSS provides
@starting-style, transition-behavior: allow-discrete, and overlay ...
```

検索はオフラインのTensorFlow.js（ブラウザやNode.jsで動く機械学習ライブラリ）で動くため、ネットワーク通信もAPIキーも不要です。手元で完結するのでプライバシーや速度の面でも扱いですね。

## 実際に自分のポートフォリオへ使ってみた

私のポートフォリオサイト[kano.codes](https://kano.codes)を題材にModern Web Guidanceでチェックしてもらいました。

出した指示はシンプルです。「このリポジトリをCSS・JavaScript・HTMLからアクセシビリティ・パフォーマンス・セキュリティまで、モダンなWeb開発のベストプラクティスでチェックして、更新案を提案して」というものです。結果は見やすいようにHTMLで出力させました。

![Modern Web Guidanceがkano.codesを棚卸しして出力したレポート](/images/modern-web-guidance/audit-report.png)
_採用済み・追加候補・見送りに分けて整理されたレポート_

次のURLから確認できます。

https://kano.codes/modern-web-audit.html

次のような結果が出ました。

■ すでに採用済みの機能
`oklch()`による色指定、`:has()`、Subgrid、invoker commands（`command`・`commandfor`）など、最新の書き方がすでに入っていることを確認できました。

■ 改善案
テーマ切り替え時の一瞬のちらつきを抑える方法や、日本語フォントのレイアウトのずれを抑える`font-size-adjust`など、まだ手を入れられる余地が具体的に挙がってきました。セキュリティ面の指摘もしてくれました（レポートHTMLからは外してあります）。

■ 見送るべきもの
これは別のスキルによるものですが、過去に試して相性が悪く外した機能を参照しつつ「これは入れない方がよい」と理由つきで整理されました。新しい機能を闇雲に勧めるのではなく、見送る判断まで添えてくれるのが信頼できるところです。

## モダンなWeb機能を、AIにもちゃんと使ってもらおう

私はCSSやJSの最新情報を追うのが好きです。それでもAIエージェントにコードを書かせると、その知識がなかなか反映されず、もどかしい思いをしていました。「こう書くんだよ」と"教育的指導"を行ったりスキルを整えたりするのですが、当然私の知識も断片的で網羅できていたわけではありません。アクセシビリティやセキュリティの知識まではカバーできない辛さもありました。

Modern Web Guidanceはその悩みを解決してくれるものでした。「ここ数年でWebに増えたモダンな機能を網羅的にサイトへ取り入れたい」という、ぼくのほしかったさいきょうのスキルが手に入った感覚です。ぜひ`$ npx modern-web-guidance@latest install` しましょう。

https://github.com/GoogleChrome/modern-web-guidance

https://developer.chrome.com/docs/modern-web-guidance?hl=ja

https://developer.chrome.com/docs/modern-web-guidance/get-started?hl=ja

https://www.youtube.com/live/aqmpZocmR8o?si=INSdqS8hKXAZlu3S&t=2512


## 続編： Modern Web GuidanceからCSSのDos/Don'tsを学ぼう


記事内にも書いた通り、Modern Web GuidanceのリポジトリはGoogleが推奨するベストプラクティスの宝庫。その中でもCSSのDos/Don'tsにしぼり、次の記事で解説しました。

https://zenn.dev/ubie_dev/articles/modern-css-dos-donts

