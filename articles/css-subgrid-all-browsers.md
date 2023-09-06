---
title: "CSSのSubgridが全ブラウザ対応へ。Gridの入れ子を完全理解する"
emoji: "🎃️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css" ]
published: true
publication_name: moneyforward
---

本日、Chrome 117のリリースでCSSのSubgridに対応しました。

Subgridとは、CSS Gridの機能の一つで、親グリッドの行や列に子要素を整列させることが可能です。

この記事では、Subgridの基本から応用までを、具体的なデモを交えて詳しく解説します。また、CSS Gridが初めての人でもわかりやすいよう、CSS
Grid自体の解説も盛り込んでいます。

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

# 従来のCSS Gridの限界と、subgridの登場

CSS Gridは柔軟な表現ができますが、表現の限界があります。 CSS Gridを使って、次のようなコンテンツを作るケースを考えてみましょう。

![](/images/subgrid/goal_pc.png)
*ウインドウサイズ：大*

![](/images/subgrid/goal_sp.png =400x)
*ウインドウサイズ：中*

タイトル・画像・文章・ラベルからなる「カード」要素が並んでいます。

![](/images/subgrid/card.png =400x)
*カード要素*

コンテンツにおいては、カード内の各高さが、他のカードと一致しています。タイトル・説明文に注目してください。それぞれ、横並びになったカード内で、最大のタイトル・説明分の高さになっています。

![](/images/subgrid/demo_1.png)
*ウインドウサイズ：大*

コンテンツはレスポンシブ対応するので、横並びになったカード内のタイトル・説明文の高さは変わります。

![](/images/subgrid/demo_2.png)
*ウインドウサイズ：小*

## 従来はJavaScriptを使っていた

「ウインドウサイズが変わるごとンシブ対応するごとに、各行の最大の高さにあわせる」ということは、従来のCSSだけでは実現不可能です。

開発の現場では、JavaScriptを使って実現していました。

- JavaScriptを使って、行ごとの最大の高さを計算する
- 各行の高さを指定する
- ウインドウサイズが変わるごとに（あるいはブレイクポイントを跨ぐごとに）高さを計算し直す

JavaScriptを使うため、処理の負荷が高く、パフォーマンスの悪化を引き起こしてしまいます。無理やり

## Gridの入れ子で実現できる

目的のレイアウトは、Gridを入れ子にすれば解決できます。

まずは親Gridを作ります。次の図のケースでは、8行3列のGridになっています。

![](/images/subgrid/parent_grid.png)
*8行3列の親Grid*

カード要素は、子Gridとして扱います。4行1列のカードとして作ればOKです。

![](/images/subgrid/subgrid_example.png =300x)
*4行1列の子Grid*

親Gridに、子Gridを配置します（入れ子にします）。その際、**親Gridの4行分を、子Gridの4行分と一致させます**。こうすることで、カード内の各行の高さが揃います。

![](/images/subgrid/set_subgrid.png)


ウインドウサイズが小さい時も、カードの各要素の高さが揃います。

![](/images/subgrid/grid_ex_small.png)
*ウインドウサイズ：小*

従来のCSS Gridの仕様であるCSS Grid Layout Level 1では、そういった行列の入れ子をすることができませんでしたが、 新しい仕様であるCSS Grid Layout Level 2では、入れ子が可能になります。それが「subgrid」です。

subgridは、Firefox・Safariで対応済みでしたが、**本日（2023/09/06）リリースされたChrome 117でも対応しました**。Microsoft Edgeでは、9月14日週にリリースされる117で対応します。 これにより、**全ブラウザでsubgridが使えるようになります**。

# デモURL

こちらの URL からデモを確認できます。

@[codepen](https://codepen.io/tonkotsuboy/pen/GRXexYY)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/GRXexYY)

# 実践: subgrid を使ってカードレイアウトを作る

カードレイアウトを作りながら、subgridの挙動を理解しましょう。

## 1. HTMLのコーディング

HTMLは次のとおりです。

```html

<div class="card-container">
  <div class="item">
    <h1 class="item-title">Feeling Good</h1>
    <img
      class="image"
      src="https://assets.codepen.io/221808/gallery_1.jpg"
      width="360"
      height="240"
      alt=""
    />
    <p class="description">I AM A CAT. As yet I have no name.</p>
    <div class="label">
      <span class="material-symbols-outlined">pets</span>Goods
    </div>
  </div>
  <div class="item">
    <h1 class="item-title">What's going on here?</h1>
    <img
      class="image"
      src="https://assets.codepen.io/221808/gallery_2.jpg"
      width="360"
      height="240"
      alt=""
    />
    <!--  中略  -->
  </div>
  <div class="item">
    <!--  中略  -->
  </div>
  <div class="item">
    <!--  中略  -->
  </div>
  <div class="item">
    <!--  中略  -->
  </div>
  <div class="item">
    <!--  中略  -->
  </div>
</div>
```

親Grid・子Gridはそれぞれ次のとおりです。

- `.card-container`要素: 親Grid
- `.card`要素: 子Grid

## 2. 親要素をCSS Gridで配置する

親Gridである `.card-container` 要素を、CSS Gridで配置します。

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

ここからがsubgridの出番です。 子Gridである`.card`要素に、CSS Gridを指定しています。そして、子Gridの行の高さを、親Gridの行の高さに合わせるよう、指定してます。

```css
.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 4;
}
```

- `grid-template-rows` プロパティ
  - **行**方向のサイズを指定する
- `subgrid`
  - 子のグリッドサイズが、親Girdで定義されたグリッドサイズを使用します
- `grid-row: span 4;`
  - 自身の行について、4行分のサイズを使うようにする。subgridが指定されていることにより、親要素の4行分の高さになる

組み合わせると、次の結果になります。

- 子Gridである `.card` 要素内の各行が、親Girdの`.card-container`要素の行に行トラックにしたがう
- `.card` 要素の高さは、`.card-container`要素の4行分

ここまでの実行結果は次のとおりです。各カードの要素の高さが揃っていますね。

![](/images/subgrid/step3_1.png)
*ウインドウサイズ：大*

![](/images/subgrid/step3_2.png =400x)
*ウインドウサイズ：小*

## 4. 親Gridで要素間の隙間（`gap`）を指定する

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

CSS Gridでレイアウトする場合、ブラウザのデバッグツールを使うと便利です。たとえば、ChromeのDevToolを使い、Grid要素横の「Grid」をクリックすれば適用されているCSS Gridを明示できます。

![](/images/subgrid/step4_2.png)

確認すると、カード内にも隙間が指定されていることがわかります。子Gridである`.card`
要素が親要素の行に従っているため、親要素の行の隙間が、`.card`要素にも適用されているのです。

Firefox・Safari・Edgeの全ブラウザにて同様の機能があります。

## 5. 子Gridで要素間の隙間（`gap`）を指定する

Subgridの便利な点は、**子Gridにおいて要素の隙間を上書きできることです**。親Gridの隙間が`40px`
で大きすぎるので、子Gridは `12px` に指定しましょう。

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

親Gridと子Gridに、それぞれ`gap` が指定されていることがわかります。

![](/images/subgrid/step5_2.png)

ちなみに、親Grid同様、ChromeのDevToolでGrid要素横の「subgrid」をクリックすると、次のように適用されているGridを明示できます。

![](/images/subgrid/devtool_subgrid.png)

# 対応ブラウザ

subgridの対応ブラウザは次のとおりです。

本日（2023/09/06）にChrome 117が対応しました。Chromeと同じChromiumエンジンを使っているMicrosoft
Edgeも、[9月14日週にリリースされる](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-release-schedule)
117でsubgridに対応します。これで、全ブラウザでsubgridが使えることになります。

各ブラウザ対応開始バージョンは次のとおり。使用する際は、プロジェクトの対象ブラウザに気をつけましょう。

- Chrome 117 で対応（NEW！）
- Safari 16.0 で対応
- Firefox 71 で対応
- Edge 117 で対応予定（NEW！）

# subgridを待ちわびていた

CSS Gridが全ブラウザ対応したのは2017年頃でしたが、その頃から行列の入れ子をしたいとずっと思っていました。私はよく勉強会でCSS
Gridを取り上げますが、subgridを取り上げる度に「まだ対応しないのかな？」と首を長くして待っていました。

# CSS Gridを学ぶ関連資料

基本的な内容から応用的な内容まで、CSS Gridの情報を発信しています。本記事とあわせてご参照ください。

https://cssnite.jp/archives/post_3023.html

https://cssnite-osaka.com/vol52/followup/session01.html

https://speakerdeck.com/tonkotsuboy_com/css-gridflexboxno-zui-jin-nojin-hua-tomirai

# 関連資料

- [サブグリッド - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_grid_layout/Subgrid)
- [CSS Grid Layout Module Level 2 - subgrid](https://drafts.csswg.org/css-grid/#subgrids)

# スペシャルサンクス

デモのデザインには、デザイナーの松下絵梨さんにご協力いただきました。ありがとうございます。

https://twitter.com/matsu_eri


