---
title: "Sassなしで入れ子が可能に。CSSネストがブラウザにやってきた"
emoji: "🧑🏻‍🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
publication_name: moneyforward
---

先日リリースされたGoogle Chrome Canaryで、CSSでネスト（入れ子）ができるようになりました💐

次のようなコードが、「SassではなくCSSで」できるようになります。Sassを使わないCSSコーディングを大きく変えることでしょう。

```css
.container {
  .child {
    color: red;
  }
}
```

▼ ブラウザの実行結果（Google Chrome Canary）

![](/images/css-nesting-module/example.png)


機能はほぼSassのネストと同じですが、一部に違いがあります。

本記事では、CSSネストの基本から各シーンでのコード例までを、デモを交えて紹介します。

# ネストの基本

Sassのネストとほぼ同じです。

次のようなHTMLがあったとしてます。

```html
<div class="container">
  <div class="child">
    Child
  </div>
</div>
```

`.child` 要素の文字色を赤色にしたい場合、従来は次のようにしていました。

```css
.container .child {
  color: red;
}
```

CSSのネストに対応したブラウザでは、次のように記述できます。

▼ CSSのネスト記法①👩‍🎨

```css
.container {
  .child {
    color: red;
  }
}
```

もしくは、 `&`を使って次のように記述します。

▼ CSSのネスト記法②👩‍🎨

```css
.container {
  & .child {
    color: red;
  }
}
```


# `h1`や`p`といった要素名のセレクターの場合

次のようなHTMLがあったとしてます。

```html
<section>
  <h1>タイトル</h1>
</section>
```

`h1` 要素の文字色を赤色にしたい場合、**次のようにしてもスタイルが当たりません**。

▼ ネストがうまくいかない例

```css
section {
  h1 {
    color: red;
  }
}
```

`h1`や`p`といった要素名のセレクターの場合は、`&`を使う必要があります。

▼ CSSのネスト記法👩‍🎨

```css
section {
  & h1 {
    color: red;
  }
}
```

# 複数の親セレクターに対してネストはできるか？

可能です。

次のようなHTMLがあるとします。

```html
<div class="apple">
  <p class="text">りんご</p>
</div>
<div class="orange">
  <p class="text">みかん</p>
</div>
```

`.apple`要素、`.orange`要素のそれぞれの子要素に`.text`要素があります。`.text`要素の文字色を赤色にしたい場合、従来は次のようにしていました。

▼ 従来のCSS

```css
.apple .text,
.orange .text {
  color: red;
}
```

ネストを使えば、次のように記述できます。

▼ CSSのネスト👩‍🎨

```css
.apple,
.orange {
  .text {
    color: red;
  }
}
```

# 一つの親要素から複数の子のCSS指定をネストにできるか？

可能です。

次のようなHTMLがあるとします。

```html
<div class="container">
  <div class="child1">子供1</div>
  <div class="child2">子供2</div>
</div>
```

`.container`要素内に複数の子要素があります。それぞれの子要素の文字色を赤色にしたい場合、従来は次のようにしていました。

▼ 従来のCSS

```css
.container .child1,
.container .child2 {
  color: red;
}
```

ネストを使えば、次のように記述できます。

▼ CSSのネスト👩‍🎨

```css
.container {
  .child1,
  .child2 {
    color: red;
  }
}
```

# `&:hover` の記法はできるか？

可能です。

次のようなHTMLがあるとします。

```html
<a class="link" href="#">リンク</a>
```

`.link`要素で、`hover`時と`active`時に文字色を赤色にしたい場合、従来は次のようにしていました。

▼ 従来のCSS

```css
.link:hover,
.link:active {
  color: red;
}
```

ネストを使えば、次のように記述できます。

▼ CSSのネスト👩‍🎨

```css
.link {
  &:hover,
  &:active {
     color: red;
  }
}
```

# メディアクエリのネストはできるか？

可能です。

次のようなHTMLがあるとします。

```html
<div class="box"></div>
```

`.box`要素について、通常の文字色は青色、ウインドウサイズ800px以上のときのみ文字色を赤色にしたい場合、従来は次のようにしていました。

▼ 従来のCSS

```css
.box {
  color: blue;
}

@media (min-width: 800px) {
  .box {
    color: red;
  }
}
```

ネストを使えば、次のように記述できます。

▼ CSSのネスト👩‍🎨

```css
.box {
  color: blue;

  @media (min-width: 800px) {
    color: red;
  }
}
```

「`.box`要素についてのメディアクエリを設定したい」と関心事が分離されるので、コードの見通しがよくなって便利ですね。 Sassの経験がある方は、それと同じ挙動であると思えばよいです。

# セレクター名の一部を使った `&__bar` の記法はできるのか？

できません🙅‍


BEMなどを使っている際、次のような `foo__bar` というクラス名をHTMLで定義し・・・

```html
<div class="foo__bar"></div>
```

次のようにSassでスタイルをあてるケースがよくあります。

▼ Sass。CSSのネストではない

```scss
.foo {
  &__bar {
    color: red;
  }
}
```

こういった使い方はCSSネストではできません。

ちなみにこの記法は、ジャンプできない・リファクタリングできない・コード検索が手間という理由により、CSS被害者救済法という法律で禁止されました。

@[tweet](https://twitter.com/tonkotsuboy_com/status/1269848769488515073)

# CSSネストをブラウザで確認するには？

CSSネストの対応ブラウザは、現状Google Chrome Canaryのフラグつきのみです。

![](/images/css-nesting-module/caniuse.png)

Canaryのフラグを有効にするには、`chrome://flags/` にアクセスし、「Experimental Web Platform features」を「Enabled」に設定します。

![](/images/css-nesting-module/flags.png)

# デモのURL

各動作が確認できるURLは次のとおりです。Chrome Canaryでご確認ください。

@[codepen](https://codepen.io/tonkotsuboy/pen/ExRbPgV)
[https://codepen.io/tonkotsuboy/pen/ExRbPgV](https://codepen.io/tonkotsuboy/pen/ExRbPgV)

▼ Chrome Canaryでの実行結果

![](/images/css-nesting-module/canary-result.png)

# 今すぐCSSネストを使うには？

全ブラウザの対応はまだ先ですが、現状でも使いたければPostCSSのプラグインを使いましょう。

- [postcss\-plugins/plugins/postcss\-nesting](https://github.com/csstools/postcss-plugins/tree/main/plugins/postcss-nesting)

![](/images/css-nesting-module/postcss.png)

# 最後に

CSSネストができるようになると、CSSのコーディングがラクになります。使い方もSassと似ているので、学習コストも低いでしょう。

CSSネストを使いこなし、見通しがよく便利なコーディングをしましょう。

## 関連資料

- [CSS Nesting Module](https://www.w3.org/TR/css-nesting-1/)


# 11/25のCSS Niteでも紹介します

CSS NestなどのモダンCSSについては、 [Coder's High 2022 (Part 3)](https://cssnite.doorkeeper.jp/events/141697)でも紹介します。興味のある方はぜひご参加ください🧑🏻‍🎨

[![](/images/css-nesting-module/cssnite-20221125-CodersHigh.png)](https://cssnite.doorkeeper.jp/events/141697)
