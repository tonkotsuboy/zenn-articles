---
title: "最短2行。上下左右中央揃えにはCSS Gridが一番ラク"
emoji: "⚽︎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
publication_name: moneyforward
---

CSSで要素を上下左右中央揃え（以下、中央揃え）する機会は多いです。要素は1つだけの場合もありますし、複数の要素の場合もあります。

▼ 中央揃えの4つの事例

![](/images/centering/goal-list.png)

昔は多くのコードを書いていましたが、CSS Gridを使えば最短2行のコードだけで実現できます。2020年から全ブラウザ（Chrome・Firefox・Safari・Edge）で対応済みです。

本記事では、単一・複数要素の中央揃えの方法、キーとなる`place-content`と`place-items`の使い方、従来のコードとの比較についてデモを交えて解説します。

# 結論

1つの要素の中央揃えは、次の2行のコードを使います。

```css
.container {
  display: grid;
  place-content: center;
}
```

![](/images/centering/single.png)

複数の要素を中央揃えしたければ、次の3行のコードを使います。

```css
.container {
  display: grid;
  place-content: center;
  place-items: center;
}
```

![](/images/centering/row.png)


# 従来の手法

従来の手法としては、例えば次のようなコードがありました。6行ものコードが必要ですし、中央揃えしたい要素の幅・高さが変わる度に、`margin-top`等の箇所を変える必要があります。

```css
.container {
  position: relative;
}

.container .box {
  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -100px;
  margin-left: -100px;
}
```

※ 幅200px、高さ200pxの要素の中央揃えの場合

`calc()`でまとめれば4行になりますが、中央揃えしたい要素の幅・高さに依存していることには変わりません。

```css
.container {
  position: relative;
}

.container .box {
  position: absolute;
  top: calc(50% - 100px);
  left: calc(50% - 100px);
}
```

次のようなコードを使えば配置したい要素の幅・高さは考慮しなくてよくなりますが、7行ものコードが必要です。

```css
.container {
  position: relative;
}

.box {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
}
```

Flexboxを使ったとき、1つの要素を中央揃えするのにも3行のコードが必要です。

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

CSS Gridを使うと最短2行のコードで実現できます。1つの要素・複数要素の中央揃えの方法とともに紹介します。

# 1つの要素の中央揃え

1つの画像要素を中央揃えにする方法です。

![](/images/centering/single.png)


HTMLは次のとおりです。`.container`という親要素の中に、画像要素が配置されています。

```html
<div class="container">
  <img src="画像パス" />
</div>
```

CSSは次の2行の指定をします。

```css
.container {
  display: grid;
  place-content: center;
}
```

コードと実行結果は次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/ZERmVON)

# `place-content`プロパティとは

`place-content`プロパティとは、行列自体の配置を制御するためのプロパティです。`place-content: center;`と指定すれば、行列自体の配置を中央にできます。

1つの要素の中央揃えの場合、1行×1列の配置を中央にしているということになります。

![](/images/centering/place-content.png)


`place-content`プロパティは、次の2つのプロパティのショートハンドです。

- `justify-content`プロパティ
  - 行列の横方向（インライン軸）の配置を制御
- `align-content`プロパティ
  - 行列の縦方向（ブロック軸）の配置を制御

値は、一例として次が使えます。

- `center`: 中央
- `start`: 先頭側
- `end`: 末尾側
- `left`: 左側
- `right`: 右側
- `space-between`: 均等配置
- `space-around`: 均等配置
- `space-evenly`: 均等配置

※ 各均等配置の値については、最初と最後のアイテムの余白が異なります（[place\-content \| MDN](https://developer.mozilla.org/ja/docs/Web/CSS/place-content)）


# 複数要素を中央揃えする場合

1つの要素の場合は、2行で中央揃えができましたが、複数要素の場合はどうでしょうか？

たとえば、次のように画像とテキストを中央揃えしてみましょう。

![](/images/centering/row.png)

`.container`という親要素の中に、画像要素とテキスト要素が配置されています。これらを縦に並べて中央揃えします。

```html
<div class="container">
  <img src="画像パス" />
  <p>にゃあと鳴く声。そこに彼はいた</p>
</div>
```

## 複数要素の場合は2行のコードだと中央揃えできない

冒頭の2行の上下中央揃えを使ってみましょう。

▼ NG例

```css
.container {
  display: grid;
  place-content: center;
}
```

すると、画像が左右揃えになりません。**`place-content`プロパティは行列全体をどの位置に配置するかだけを指定するため**、セル内でのアイテムの配置が中央揃えになっていないのです。

▼ NG例

![](/images/centering/row-ng.png)


## レイアウトのために`div`を増やしてはいけない

次のように、`.container`クラス内に`div`を増やせばいいと思われるかもしれません。

```html
<div class="container">
  <div class="inner">
    <img src="画像パス" />
    <p>にゃあと鳴く声。そこに彼はいた</p>
  </div>
</div>
```

しかし、レイアウトのためだけにHTMLを増やすのは好ましくありません。スタイルの制御はできるだけCSSだけで完結するべきです。

## `place-items`プロパティで複数要素の中央揃え

複数要素を中央揃えしたい場合は、`place-items`プロパティを使います。

```css
.container {
  display: grid;
  place-content: center;
  /* 追加 */
  place-items: center;
}
```

コードと実行結果は次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/jOKQejB)

# `place-items`プロパティとは

`place-items`プロパティとは、行列内での**アイテムの配置を**制御するためのプロパティです。`place-items: center;`と指定すれば、セル内でアイテムを中央揃えにできます。

![](/images/centering/place-items.png)

`place-items`プロパティは、次の2つのプロパティのショートハンドです。

- `justify-items`プロパティ
  - 横方向（インライン軸）のセルの位置を制御
- `align-items`プロパティ
  - 縦方向（ブロック軸）のセルの位置を制御

値は、一例として次が使えます。

- `center`: 中央
- `start`: 先頭側
- `end`: 末尾側
- `left`: 左側
- `right`: 右側
- `baseline`: ベースライン
- `stretch`: 引き伸ばされる

`place-content`と`place-items`プロパティを同時に使うことで、**行列全体と行列内のアイテムを中央揃えできる**のです。

# 複数要素の中央揃え例

複数要素の中央揃えについて、いくつか例を見てみましょう。

## 横に並べた要素の中央揃え

画像とテキストが横に並んだコンテンツを、中央揃えしてみましょう。

![](/images/centering/column.png)

HTMLは次のとおりです。

```html
<div class="container">
  <img src="画像パス" />
  <p>八番隊隊長 鹿猫抜雲斎</p>
</div>
```

CSSは次のようにします。`grid-auto-flow: column;`で、行列の方向を列方向（左右方向）にします。

```css
.container {
  display: grid;
  place-content: center;
  place-items: center;
  /* 行列の自動配置方向を列方向にする */
  grid-auto-flow: column;
}
```

コードと実行結果は次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/qBKQJgJ)

ちなみにこれはFlexboxだと3行でできますね。筆者的には、「横方向だったらFlexbox、縦方向だったらCSS Gridを使う」といった使い分けが面倒なので、一律でCSS Gridを使っています。

```css
.container {
  display: flex;
  justify-content: center;  /* place-contentでもOK */
  align-items: center;  /* place-itemsでもOK */
}
```

## 複数行・複数列の中央揃え

2行×2列の要素を、中央揃えしてみましょう。

![](/images/centering/square.png)

HTMLは次のとおりです。

```html
<div class="container">
  <img src="画像パス" />
  <p>🐾</p>
  <p>🐾</p>
  <img src="画像パス" />
</div>
```

CSSは次のとおりです。


```css
.container {
  display: grid;
  place-items: center;
  place-content: center;
  /* 2行×2列にする */
  grid-template: repeat(2, auto) / repeat(2, auto);
}
```

コードと実行結果は次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/ExROOMB)

# 特定のアイテムの配置だけを変えたい場合は`place-self`を使う

行列全体ではなく、特定のアイテムだけの配置だけを変えたい場合は`place-self`を使います。値は`place-items`のものとほぼ同じです。

- [place\-self \| MDN](https://developer.mozilla.org/ja/docs/Web/CSS/place-self)


# `align-items`や`justify-contents`を個別に指定したほうがいいのか？

複数要素の中央揃えで実装した次のレイアウトについて。

![](/images/centering/row.png)

今回は次のようにCSSを実装しました。

```css
.container {
  display: grid;
  place-content: center;
  place-items: center;
}
```

次のように、`place-*`の一括指定を使わずに書いても同様に中央揃えになります。

```css
.container {
  display: grid;
  justify-content: center;
  align-items: center;
}
```

しかし、次のレイアウトでは`place-*`を使う必要があります。

![](/images/centering/square.png)

作りたいレイアウトに応じて使い分けてもいいですが、筆者はどんなレイアウトの中央揃えにも対応できるように、`place-*`を使うことが多いです。

# 今回の手法は2020年から使える

今回の便利な中央揃えが使えるようになったのは、2020年。Chromium版Edgeが登場してからです。他ブラウザではすでに`place-content`・`place-items`に対応していたため、Chromium版Edgeの登場により全ブラウザ（Chrome・Safari・Firefox・Edge）で使えるようになった形です。

[![](/images/centering/caniuse.png)](https://caniuse.com/mdn-css_properties_place-content)

- [CSS property: place\-content \| Can I use\.\.\.](https://caniuse.com/mdn-css_properties_place-content)


# 最後に

筆者はFlexboxの3行の中央揃えをずっと使ってきましたが、2020年に全ブラウザで`place-*`に対応して浸透した後は、ずっと本記事の手法を使っています。上下左右中央揃えをしないコンテンツはほぼないので、頻繁に使うCSS表現の一つです。

ぜひ今回の記事をとおして各種中央揃えの手法を身に着け、開発の現場に役立てていただければ幸いです。

この記事は、Money ForwardのEngineering Advent Calendar 2022の5日目の記事です。

[![](/images/centering/advent.png)](https://adventar.org/calendars/7397)

- [Money Forward Engineering 1 Advent Calendar 2022](https://adventar.org/calendars/7397) 


今回の猫のロゴ画像は、デザイナーの[松下 絵梨さん](https://twitter.com/matsu_eri)に作成いただきました。



関連リンク

- [松下 絵梨さん](https://twitter.com/matsu_eri)
- [place\-content \| MDN](https://developer.mozilla.org/ja/docs/Web/CSS/place-content)
- [place\-items \| MDN](https://developer.mozilla.org/ja/docs/Web/CSS/place-items)

