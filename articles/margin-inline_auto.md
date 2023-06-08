---
title: "margin-inline:autoを使おう。margin-left:autoとmargin-right:autoを書くのが面倒なあなたへ"
emoji: "🫎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css" ]
published: true
publication_name: moneyforward
---

要素を左右中央揃えにしたいとき、よく使われるのが `margin-left: auto` と `margin-right: auto` です。

```css
.box {
  margin-left: auto;
  margin-right: auto;
}
```

このスタイルを適用すると、指定した要素の左右のマージンが同等になり、結果として要素はその親要素内で中央に配置されます。


しかし、最近のCSSを使うと、より簡単に要素を中央揃えにできます。それが `margin-inline: auto;` です。全ブラウザで使えます。

```css
.box {
  margin-inline: auto;
}
```

本記事では、`margin-inline: auto;` によるデモを紹介したあと、`margin-inline`とは何かについて解説します。

# 簡単なデモで動作を確認する

簡単なデモで動作を確認してみましょう。次のように、 `.container` 要素の中に `.box` 要素を配置します。

```html
<div class="container">
  <div class="box">
    BOX
  </div>
</div>
```

`.box` 要素を中央揃えしたい場合、`.box` 要素に対して `margin-inline: auto;` を指定するだけです。

```css
.box {
  margin-inline: auto;
}
```

実行結果は次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/LYXYjYx)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/LYXYjYx)

# `margin-inline` とは

`margin-inline` はCSSの「論理プロパティ」で、要素の「インライン方向」のマージンを設定するためのものです。

「論理プロパティ」とは、従来の上、下、左、右といった方向（物理的な向きと言います）ではなく、文字の記述される方向（書字方向）によって方向が変わるプロパティのことです。2つの方向があります。

- インライン方向
  - テキストの流れと同じ方向。英語や日本語横書きでは、横方向（水平）になります
- ブロック方向
  - テキストの流れと垂直の方向。英語や日本語横書きでは、縦方向（垂直）になります

![](/images/margin-inline/logical_css.png)

`margin` だけではなく、 `padding`・`border`・`size`など、さまざまなプロパティと組み合わせて使えます。

参考: [CSS 論理的プロパティと値](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_logical_properties_and_values)

`margin-inline`とは、`margin-iniline-start`（インライン先頭）と`margin-iniline-end`（インライン末尾）のショートハンドです。`margin-inline: auto;` を指定することで、インラインのマージンの先頭と末尾を `auto` として、中央揃えを実現しているというわけです。

# 実用的なデモ

`margin-inline: auto;` を使って、コンテンツを中央揃えするデモを作ってみましょう。

次のように、コンテンツを中央揃えにしたいとします。

![](/images/margin-inline/centering_demo.png)


HTMLは次のようになっています。

```html
<header>
  <h1>福岡</h1>
  <!-- 省略 -->
</header>
<main>
  <section id="top-section">
    <h2>西日本の魅力、福岡を堪能しよう</h2>
    <p>福岡は、九州最大の都市であり、（中略）</p>
  </section>
  <section>
    <h2>観光スポット</h2>
    <ul>
      <li>
        <img src="画像パス" />
        <h3>太宰府天満宮</h3>
        <p>太宰府天満宮は、（中略）</a>
      </li>
      <li><!-- 中略 --></li>
      <li><!-- 中略 --></li>
      <li><!-- 中略 --></li>
    </ul>
  </section>
</main>
```

この場合、CSSで `section` 要素に `margin-inline: auto;` を指定すれば、目的のレイアウトを達成できます。

```css
main section {
  margin-inline: auto;
  /* 最大幅800px */
  max-width: 800px;
}
```

こちらのURLからデモを確認できます。


[![](/images/margin-inline/centering_goal.png)](https://codepen.io/tonkotsuboy/pen/OJBKBKx)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/OJBKBKx)


# `margin: auto;` ではだめなのか？

筆者的には避けたいです。

`margin: auto;` は、次の指定のショートハンドです。

- `margin-top: auto;`
- `margin-right: auto;`
- `margin-bottom: auto;`
- `margin-left: auto;`

インライン方向の中央揃えを指定したいだけなのに、`margin-top`と`margin-bottom`も指定してしまうと、予期せぬスタイルの上書きを考慮しないとなりません。


# `margin-inline`は書字方向によって変わることに注意

`margin-inline`は、書字方向によって向きが変わることには注意が必要です。 筆者は、英語・日本語横書きのコンテンツを作ることがほとんどなので気にしたことはありませんが、人によっては縦書きのコンテンツを作ることがあるかもしれません。 その場合、`margin-inline: auto;` は縦方向の中央揃えになるので注意をしましょう。


# CSS Gridでも中央揃えは可能

「実用的なデモ」の中央揃えは、CSS Gridを使っても実現できます。

```css
main {
  display: grid;
  justify-content: center;
}

main section {
  max-width: 800px;
  justify-self: center;
}
```

状況に応じて、CSS Gridや `margin` を使い分けるとよいでしょう。

CSS Gridによる中央揃えについては、次の記事で詳しく解説しています。

https://zenn.dev/moneyforward/articles/css-grid-centering

# 対応ブラウザ

`margin-inline`は全ブラウザで対応済みです。Safariも14.1と古いバージョンから対応しているので、安心して使えると言っていいでしょう。

![](/images/margin-inline/caniuse.png)

- [CSS property: margin-inline | Can I use](https://caniuse.com/mdn-css_properties_margin-inline)



# 6/10（土）に大阪でCSSの勉強会を実施します

6/10（土）に、大阪でCSSの勉強会をします。

「Modern Coding 2023」と題し、開発現場で使える新しいCSSを紹介。ライブコーディング、セミハンズオン、質疑応答を交えながら進めます。アーカイブのみの参加も可能です。

ぜひご参加ください💪

https://cssniteosaka.doorkeeper.jp/events/157336
