---
title: "CSSのSubgridが全ブラウザ対応。Gridの入れ子の基本から応用までを完全理解する"
emoji: "🎃️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css" ]
published: true
---

2023/9/12にChrome 117、9/15にEdge 117がリリースされ、CSSのSubgridが全ブラウザに対応しました。

Subgridとは、CSS Gridで新しく使えるようになった機能の一つ。行列（グリッド）を入れ子にして、親行列の行や列に子行列を整列させることが可能です。

この記事では、Subgridの基本から応用までを具体的なデモを交えて詳しく解説します。CSS Gridが初めての人でもわかりやすいよう、CSS Grid自体の解説も盛り込んでいます。

# 前提知識: CSS Gridとは

CSS Gridとは、行と列を使ったレイアウトのことです。行・列とは、次の方向を指します。

![](/images/subgrid/grid_row.png)

![](/images/subgrid/grid_gap.png)

CSS Gridを使うと、次のようなことができます。

■ エリア名を指定して配置できる

![](/images/subgrid/area.png)

■ 行列を繰り返したり、隙間をつくる

![](/images/subgrid/repeat.png)

■ 行・列数の自動変更、敷き詰め

![](/images/subgrid/dense.png)

# 複雑な行列の入れ子と、subgridの登場

CSS Gridは柔軟な表現ができますが、表現の限界があります。 CSS Gridを使って、次のようなコンテンツを作るケースを考えてみましょう。

![](/images/subgrid/goal_pc.png)
*ウインドウサイズ：大*

![](/images/subgrid/goal_sp.png =400x)
*ウインドウサイズ：中*

タイトル・画像・文章・ラベルからなる「カード」要素が並んでいます。

![](/images/subgrid/card.png =400x)
*カード要素*

コンテンツにおいては、カード内のタイトル・画像・説明文・カテゴリの高さが、他のカードと一致しています。

タイトル・説明文に注目してください。テキスト1行分・テキスト2行分のタイトルがあったり、テキスト2行分・テキスト3行分の説明文があります。 それぞれ、横並びになったカード内で、最大のタイトル・説明分の高さになっています。

![](/images/subgrid/demo_1.png)
*ウインドウサイズ：大*

コンテンツはレスポンシブ対応するので、横並びになったカード内のタイトル・説明文の高さは変わります。ウインドウサイズが小さくなると、タイトルはテキスト1行分のみとなります。

![](/images/subgrid/demo_2.png)
*ウインドウサイズ：小*

## 従来はJavaScriptを使っていた

「ウインドウサイズが変わるごとに、各行の最大の高さにあわせる」ということは、従来のCSSだけでは実現不可能です。したがって、従来の開発現場では、JavaScriptを使って実現していました。

手順は次のとおりです。

- JavaScriptを使って、行ごとの最大の高さを計算する
- 各行の高さを指定する
- ウインドウサイズが変わるごとに（あるいはブレイクポイントを跨ぐごとに）高さを計算し直す

JavaScriptを使うため、処理の負荷が高く、パフォーマンスの悪化を引き起こしてしまいます。

## 行列の入れ子で解決する

目的のレイアウトは、行列を入れ子にすれば解決できます。具体的なコードの前に、まずは考え方を紹介します。

親行列を作ります。次の図のケースでは、8行3列のGridになっています。

![](/images/subgrid/parent_grid.png)
*8行3列の親行列*

カード要素は、子行列として扱います。4行1列のカードとして作ればOKです。

![](/images/subgrid/subgrid_example.png =300x)
*4行1列の子行列*

親行列に、子行列を配置します（入れ子にします）。その際、**親行列の4行分を、子行列の4行分と一致させます**。こうすることで、カード内の各行の高さが揃います。

![](/images/subgrid/set_subgrid.png)


ウインドウサイズが小さい時も、カードの各要素の高さが揃います。

![](/images/subgrid/grid_ex_small.png)
*ウインドウサイズ：小*

## Subgridの登場

従来のCSS Gridの仕様であるCSS Grid Layout Level 1では、行列の入れ子をすることができませんでした。親行列の中に子行列を入れたとしても、子行列を親行列の行に揃えることはできません。

![](/images/subgrid/css-grid-module-1.png)


新しい仕様であるCSS Grid Layout Level 2では、行列の入れ子が可能になります。それが「subgrid」です。親行列の中に子行列を入れたとき、子行列を親行列の行に揃えることが可能になります。

![](/images/subgrid/css-grid-module-2.png)


subgridは、Firefox・Safariで対応済みでしたが、9/12にリリースされたChrome 117、9/15にリリースされたEdge 117でも対応しました**。これにより、**全ブラウザでsubgridが使えるようになりました**。

# 実践: subgrid を使ってカードレイアウトを作る

カードレイアウトを作りながら、subgridの挙動を理解しましょう。

## 1. HTMLのコーディング

HTMLは次のとおりです。

```html
<div class="card-container">
  <div class="card">
    <h1 class="card-title">Feeling Good</h1>
    <img
      class="image"
      src="https://assets.codepen.io/221808/gallery_1.jpg"
    />
    <p class="description">I AM A CAT. As yet I have no name.</p>
    <div class="label">Goods</div>
  </div>
  <div class="card">
    <h1 class="card-title">What's going on here?</h1>
    <img
      class="image"
      src="https://assets.codepen.io/221808/gallery_2.jpg"
    />
    <!-- 中略 -->
  </div>
  <div class="card"><!-- 中略 --></div>
  <div class="card"><!-- 中略 --></div>
  <div class="card"><!-- 中略 --></div>
  <div class="card"><!-- 中略 --></div>
</div>
```

親行列・子行列はそれぞれ次のとおりです。

- `.card-container`要素: 親行列
- `.card`要素: 子行列

## 2. 親要素をCSS Gridで配置する

親行列である `.card-container` 要素を、CSS Gridで配置します。

```css
.card-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
}
```

- `display: grid;`
  - `.card-container`要素を、CSS Gridで配置する
- `grid-template-columns` プロパティ
  - 列方向のサイズを指定する
- `repeat()`
  - 列や行を繰り返す
- `auto-fit`
  - 画面サイズに合わせて、列や行の数を自動で変更する
- `minmax(350px, 1fr)`
  - 列や行の最小値と最大値を指定する。この場合は`350px`以上、`1fr`以下になる
- `repeat(auto-fit, minmax(350px, 1fr))`
  - 画面サイズに合わせて、`350px`以上、`1fr`以下の列を自動で作成する

ここまでのレイアウトを確認すると、次のとおりになります。

![](/images/subgrid/step2_1.png)
*ウインドウサイズ: 大*

![](/images/subgrid/step2_2.png =400x)
*ウインドウサイズ: 小*

ウインドウサイズに応じて行列数が変わりますが、カード内の高さが揃っていませんし、カード間の隙間も空いていません。

## 3. 子要素をCSS Gridで配置する

ここからがsubgridの出番です。次のように指定します。

```css
.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 4;
}
```

- `display: grid;`
  - `.card`要素を、CSS Gridで配置する
- `grid-template-rows` プロパティ
  - **行**方向のサイズを指定する
- `subgrid`
  - 子行列のサイズは、親行列で定義された行や列のサイズを使用します
- `grid-row: span 4;`
  - 自身の行について、4行分のサイズを使うようにする。subgridが指定されていることにより、親要素の4行分の高さになる

`grid-template-rows: subgrid;` の部分が、今回新たに使えるようになったsubgridの機能（CSS Grid Layout Level 2）です。本記事内の他の技術は、全て従来のCSS Gridの機能（CSS Grid Layout Level 1）です。

組み合わせると、次の結果になります。

- 子行列である `.card` 要素内の各行が、親行列の`.card-container`要素の行トラックにしたがう
- `.card` 要素の高さは、`.card-container`要素の4行分

ここまでの実行結果は次のとおりです。各カードの要素の高さが揃っていますね。

![](/images/subgrid/step3_1.png)
*ウインドウサイズ：大*

![](/images/subgrid/step3_2.png =400x)
*ウインドウサイズ：小*

## 4. 親行列で要素間の隙間（`gap`）を指定する

仕上げに、要素間の隙間を指定しましょう。

親要素である `.card-container` 要素に対して、`gap` プロパティを指定します。

```css
.card-container {
  gap: 40px;
}
```

- `gap: 40px`
  - 行・列の隙間を `40px` にする

実行結果は次のとおり。要素間に隙間が空いているのがわかります。

![](/images/subgrid/step4_1.png)

しかし、カード内の余白が大きくなっているように見えますね。どうしてこうなったか調べてみましょう。

CSS Gridでレイアウトする場合、ブラウザのデバッグツールを使うと便利です。たとえば、ChromeのDevToolを使い、行列要素横の「Grid」をクリックすれば適用されているCSS Gridを明示できます。

![](/images/subgrid/step4_2.png)

確認すると、カード内にも隙間が指定されていることがわかります。子行列である`.card`
要素が親要素の行に従っているため、親要素の行の隙間が、`.card`要素にも適用されているのです。

Firefox・Safari・Edgeの全ブラウザにて同様の機能があります。

## 5. 子行列で要素間の隙間（`gap`）を指定する

Subgridの便利な点は、**子行列において要素の隙間を上書きできることです**。親行列の隙間が`40px`で大きすぎるので、子行列は`12px`に指定しましょう。

```css
/* 親のgap */
.card-container {
  gap: 40px;
}

/* 子のgap */
.card {
  row-gap: 12px;
}
```

- `row-gap: 12px`
  - 行だけの隙間を `12px` にする
  - 列だけの隙間を指定する場合は `column-gap`

実行結果は次のとおり。

![](/images/subgrid/step5_1.png)

親行列と子行列に、それぞれ`gap` が指定されていることがわかります。

![](/images/subgrid/step5_2.png)

ちなみに、親行列同様、ChromeのDevToolで行列要素横の「subgrid」をクリックすると、次のように適用されているGridを明示できます。

![](/images/subgrid/devtool_subgrid.png)

# デモのURL

今回のデモは、こちらのURLから確認できます。Subgridに対応しているブラウザでご確認ください。

@[codepen](https://codepen.io/tonkotsuboy/pen/VwqmzeJ)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/VwqmzeJ)

![demo-goal.webp](/images/subgrid/demo-goal.webp)

# 対応ブラウザ

Subgridの対応ブラウザは次のとおりです。

|                    | 対応開始バージョン |
|--------------------|----------|
| Chrome             | 117 🆕   |
| Safari             | 16.0     |
| Firefox            | 71       |
| Edge               | 117 🆕    |
| Chrome for Android | 117 🆕    |
| Safari on iOS      | 16.0     |

- 参考: [Can I use](https://caniuse.com/css-subgrid)

本日（2023/09/12）にChrome 117が対応しました。Chromeと同じChromiumエンジンを使っているMicrosoft Edgeも、9/15リリースの117でsubgridに対応しました。これで全ブラウザでsubgridが使えることになりました。

# SubgridはGridの表現を大きく変える待望の機能

CSS Gridが全ブラウザ対応したのは2017年頃でしたが、その頃から行列の入れ子をしたいとずっと思っていました。私は定期的に勉強会で登壇するのですが、[初めてsubgridを取り上げたのは2018年](https://cssnite.jp/archives/post_3023.html)で、当時はどのブラウザも対応していませんでした。2019年にFirefoxが71でいち早く対応し、2022年のSafari 16.0が続いた形です。

私は、開発現場では複雑な入れ子のレイアウトをしたいケースが間々ありますが、今回のようなレイアウトではJavaScriptを使うしかありませんでした。subgridの登場により、CSSだけで入れ子が実現できるので、Gridの表現力が大きくアップすると期待しています。

# CSS Gridを学ぶ関連資料

基本的な内容から応用的な内容まで、CSS Gridの情報を発信しています。本記事とあわせてご参照ください。

https://zenn.dev/moneyforward/articles/css-grid-centering

https://speakerdeck.com/tonkotsuboy_com/css-gridflexboxno-zui-jin-nojin-hua-tomirai

https://cssnite-osaka.com/vol52/followup/session01.html

https://cssnite.jp/archives/post_3023.html


# 参考資料

- [サブグリッド - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_grid_layout/Subgrid)
- [CSS Grid Layout Module Level 2 - subgrid](https://drafts.csswg.org/css-grid/#subgrids)
- [Hello Subgrid! CSSConf.eu](https://noti.st/rachelandrew/i6gUcF/hello-subgrid)

# スペシャルサンクス

デモのデザインには、デザイナーの松下絵梨さんにご協力いただきました。

- [@matsu_eri](https://twitter.com/matsu_eri)

[行列画像](https://zenn.dev/moneyforward/articles/css-subgrid-all-browsers#%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AD%98%3A-css-grid%E3%81%A8%E3%81%AF)の作成には、鷹野さんにご協力いただきました。

- [@swwwitch](https://twitter.com/swwwitch)


