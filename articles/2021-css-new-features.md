---
title: "2021年から開発の現場で使える3つの便利CSS - aspect-ratio, gap, is()"
emoji: "🌸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
---

この記事は[マネーフォワードアドベントカレンダー2021](https://adventar.org/calendars/6182)の13日目の記事です。

ブラウザは日々進化しています。2021年もブラウザには多くの新機能が追加されました。

私はCSSの新機能を1年かけてチェックしてきましたが、その中でもとりわけ便利だと思った3つの機能を紹介します。いずれも、2021年に全モダンブラウザ（Chrome、Firefox、Microsoft Edge、Safari）で使えるようになったもので、日々の開発をラクにしてくれることでしょう。

# 動画や画像のアスペクト比を指定できる`aspect-ratio`プロパティ

`aspect-ratio`プロパティとは、ボックスのアスペクト比（幅と高さの比率）を指定するプロパティです。

| 構文                      |
|:------------------------|
| `aspect-ratio: アスペクト比率` |

▼ 簡単な例

```css
/* .box要素のアスペクト比を4:3にする */
.box {
  aspect-ratio: 4 / 3;
}
````

## 説明

次のように、YouTube動画のアスペクト比率を16:9に固定する例を考えます。

![](/images/2021-css-new-features/aspect-ratio.jpg)

動画部分は、`iframe`で表示されています。

```html
<iframe src="動画URL"></iframe>
```

## 昔のやりかた: `padding-top`と絶対配置

昔は、`iframe`のラッパー要素を作り、`padding-top: 56.25%;`（もしくは`padding-top: calc(100% * 16 / 9);` を指定し、`iframe`を`absolute`で絶対配置する必要がありました。

```html
<div class="wrapper">
  <iframe
    class="video"
    src="動画URL"
  ></iframe>
</div>
```

```css
.wrapper {
  position: relative;
  padding-top: 56.25%;
}

iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
```

## `aspect-ratio`を使ったやり方

`aspect-ratio`プロパティを使うと、ラッパー要素や絶対配置は不要で、`iframe`に`aspect-ratio`プロパティを指定するだけで、アスペクト比が固定されます。

```css
iframe {
  aspect-ratio: 16/9;
  width: 100%;
  height: 100%;
}
```


私は**ウェブ業界に入った頃から前述の`padding-top`によるハックを使っていましたが、なんでこんな面倒くさいことをしなければならないんだろう？🤔**とずっと疑問に思っていました。`aspect-ratio`の登場で、直感的にアスペクト比を指定できるようになり助かっています。

デモはつぎのとおりです。新しいタブでデモを開くとわかりやすいのですが、YouTubeの動画が画面サイズがウインドウいっぱいに広がりつつ、アスペクト比が16:9に保たれています。

@[codepen](https://codepen.io/tonkotsuboy/pen/wvrogEw)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/wvrogEw)

## `aspect-ratio`の対応ブラウザ

`aspect-ratio`プロパティは、2021年9月に登場したSafari 15が対応したことで、全モダンブラウザで使えるようになりました。

[![](/images/2021-css-new-features/caniuse_aspect-ratio.png)](https://caniuse.com/mdn-css_properties_aspect-ratio)

- [Can I use - aspect-ratio](https://caniuse.com/mdn-css_properties_aspect-ratio)

# Flexboxで`gap`プロパティを使う

`gap`プロパティとは、行や列の隙間を指定するためのプロパティです。昔はCSS Gridのアイテムに対してのみ有効でしたが、2021年からはFlexboxのアイテムに対しても使えるようになりました。

| 構文        |
|:----------|
| `gap: 間隔` |


▼ 簡単な例

```css
/* .container要素内のアイテムの隙間を10pxにする */
.container {
  display: flex;
  gap: 10px;
}
````

## 説明

次のように、Flexboxで配置したボックスに対して、`16px`の隙間を空けることを考えましょう。

![](/images/2021-css-new-features/gap.png)

HTMLは次のようになっています。

```html
<div class="container">
  <div class="box">臨</div>
  <div class="box">兵</div>
  <div class="box">闘</div>
  <div class="box">者</div>
  <div class="box">皆</div>
  <!-- 中略 -->
</div>
```

従来であれば、`margin`を使って次のように指定していました。最初や最後のアイテムの`margin`を`0`にするなどして調整する必要があります。また、複数行になった場合には、最初の行や最後の行での`margin`の調整も必要で、煩雑でした。

```css
.container {
  display: flex;
  margin: 0 16px;
}

.box:first-child {
  margin-left: 0;
}
```

Flexboxの`gap`プロパティを使えば、次のように記述できます。スタイルの指定はコンテナーに対してだけであり、`margin`のときのようにアイテムに個別のスタイルを指定する必要はありません。複数行のときも然りです。

```css
.container {
  display: flex;
  gap: 16px;
}
```

私は単一行レイアウトではFlexbox、複数行レイアウトではCSS Gridという使い分けをしているのです。しかし、昔はアイテムの隙間を空けたいケースにおいてはFlexboxではなくCSS Gridを使っていました。前述のように、`margin`や`padding`で隙間を空けようとすると、最初の要素や最後の要素の調整が煩雑だったためです。Flexbox用の`gap`プロパティが登場したことにより、単一行レイアウトでも気兼ねなくFlexboxが使えるようになり、助かっています。

デモはこちら。ボックス間の隙間が`16px`になっています。

@[codepen](https://codepen.io/tonkotsuboy/pen/eYGBywQ)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/eYGBywQ)

## Flexbox用`gap`の対応ブラウザ

Flexbox用`gap`プロパティは、2021年4月に登場したSafari 14が対応したことで、全モダンブラウザで使えるようになりました。

[![](/images/2021-css-new-features/caniuse_gap.png)](https://caniuse.com/flexbox-gap)

- [Can I use - gap property for Flexbox](https://caniuse.com/flexbox-gap)

# セレクターリストをまとめてスタイルをあてる `:is()`擬似クラス

`is`擬似クラスとは、引数のセレクタリストに合致する要素を選択できる擬似クラスです。

| 構文                      |
|:------------------------|
| `.foo:is(.bar, .baz) {}` |

▼ 簡単な例

```css
/* section要素、nav要素、main要素内のh1のテキスト色を赤色にする */
:is(section, nav, main) h1 {
  color: red;
}
```

## 説明

次のように、ホバー時とカレント時に、文字色を`white`、背景色を`#333333`にすることを考えてみましょう。

https://www.youtube.com/watch?v=j5VG43nNq_k

リンク部分のHTMLは次のようになっています。

```html
<ul class="link-list">
  <li>
    <a class="link current" href="#">
     ホーム
   </a>
  </li>
  <li>
    <a class="link" href="#">探索</a>
  </li>
  <!-- 中略 -->
</ul>  
```

JavaScriptでは、カレント状態の`a.link`要素に対して、`.current` クラスが付与されるようになっています。

## 昔のやり方

`.link-list`内の`a`の、`:hover`と`.current`に対して同じスタイルを指定したい場合、従来は次のように指定していました。

```css
.link-list .link:hover,
.link-list .link.current {
  color: white;
  background-color: #333333;
}
```

## `:is()`を使ってセレクタリストをまとめる

`:is()`を使うと、次のようにセレクタリストをまとめられます。

```css
.link-list .link:is(:hover, .current) {
  color: white;
  background-color: #333333;
}
```

デモはこちら。リンクを押すと`.link`のカレント状態が変わり、カレント状態のスタイルはホバー時のスタイルと同じになります。

@[codepen](https://codepen.io/tonkotsuboy/pen/XWeNpbK)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/XWeNpbK)

## 詳細度が常に`0`の`:where()`

`:is()`と似た疑似クラスで、`:where()`があります。`:is()`は`()`内の詳細度になるのに対し、`:where()`の詳細度はつねに`0`です。

次の`:is()`の例では、`div a`は`red`になります。

```css
:is(div.foo a) {
  color: red;
}

div a {
  color: blue;
}

```

次の`:where()`の例では、`div a`は`blue`になります。

```css
:where(div.foo a) {
  color: red;
}

div a {
  color: blue;
}
```

## `:not()`でもセレクタリストが指定できるように

`:not()`でもセレクタリストが指定できるようになりました。

個人的にはこの機能がありがたいです。`.foo`や`.bar`**ではない要素**に対してスタイルを設定するというケースは多かったものの、昔は`:not()`の引数としてセレクタリストを取れずに不便に感じていました。現在はラクです。

```css
/* ホバーされておらず、.currentも付与されていないa要素に対してスタイルを指定 */
a:not(:hover, .current) {
  color: red;
}
```

## `:is()`・`where()`・セレクタリストをとる`not()`の対応ブラウザ

`:is()`・`where()`・セレクタリストをとる`not()`は、2021年10月に登場したChrome 88が対応したことで、全モダンブラウザで使えるようになりました。

[![](/images/2021-css-new-features/caniuse_is-pseudo.png)](https://caniuse.com/css-matches-pseudo)

- [Can I use - :is() CSS pseudo-class](https://caniuse.com/css-matches-pseudo)
- [Can I use - CSS selector: :where()](https://caniuse.com/mdn-css_selectors_where)
- [Can I use - selector list argument of :not()](https://caniuse.com/css-not-sel-list)

細かいことを言うと、`:is()`や`:where()`はSafari 15以前から対応していましたが、[Forgiving Selector](https://developer.mozilla.org/ja/docs/Web/CSS/:is#forgiving_selector_parsing)に対応したのは15からです。

# 新しいCSSで便利に開発しよう

`aspect-ratio`プロパティ、Flexbox用`gap`プロパティ、`:is()`擬似クラスは、いずれも昔の煩雑なCSSの記述をラクにしてくれるものです。シンプルなコードになれば、バグの発見がしやすくなり、他の開発者も読みやすくなることでしょう。

今回紹介したCSSの機能以外にも、便利なCSSはどんどん追加されていきます。当たり前だと思っていたCSSの知識を見直し、新しくて便利なCSSをキャッチアップしていきましょう。
