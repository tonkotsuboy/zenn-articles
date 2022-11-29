---
title: "svh・dvhがついに全ブラウザ対応へ。iOSの画面の高さいっぱいに要素を広げたいときの最適解"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
publication_name: moneyforward
---

iOS Safari の画面の高さいっぱいにヒーローイメージを表示するという表現は、よく見かけます。

![goal.png](/images/svh-dvh-lvh-for-all-browser/goal.png)

高さをいっぱいに広げるのに`100vh`を使うと、不要なスクロールが発生し、意図通りに表示されません。 この問題を解決するために、特殊なCSSを使ったりJSを使ったりと、開発の現場では多くの苦労がありました。

**本日リリースされたGoogle Chrome 108では対応した`svh`を使えば**、手軽に画面いっぱいのヒーローイメージを作れます。

```css
.hero-image {
  height: 100svh;
}
```

Safariではすでに対応済み、Chromeと中身が同じなEdgeは12/1リリースの108で対応するので、全ブラウザで使える時代が来ます。

本記事では、`svh`の使い方、同様に使えるようになった`dvh`や`svmax`などの違い、従来の手法のデメリットを、デモを交えて紹介します。

# `svh` を使ったデモ

次のようなコンテンツを作ってみましょう。ページにアクセス時、ヒーローイメージがブラウザの高さいっぱいに広がり、スクロールすると後続コンテンツが見えるようなものです。

![](/images/svh-dvh-lvh-for-all-browser/goal.webp =400x)

![](/images/svh-dvh-lvh-for-all-browser/goal-scroll.png)

HTML コードは次のとおりです。

```html
<main>
  <!-- ヒーローイメージ -->
  <div class="hero-image">
    <img src="ヒーローイメージ用画像" />
  </div>
  <section>
    <h1>ABOUT</h1>
    <!-- 以下、後続コンテンツ -->
  </section>
</main>
```

ヒーローイメージは`.hero-image`要素ですが、CSS で行うことは`height: 100svh;`を指定するだけです。

```css
.hero-image {
  height: 100svh;
}
```

iOS Safari、デスクトップの Chrome を始め、各環境で動作します。

▼ iOS Safari での動作結果

![](/images/svh-dvh-lvh-for-all-browser/goal.webp =400x)

▼ デスクトップの Chrome での動作結果

![](/images/svh-dvh-lvh-for-all-browser/goal-chrome.webp)

デモは次から確認できます。

https://codepen.io/pen/debug/NWzBBNB

- [ソースコード](https://codepen.io/tonkotsuboy/pen/NWzBBNB)

## `svh`とは

従来使われてきた`vh`や`vw`とは、ビューポートに対する高さや幅を指します。

デスクトップブラウザのようなビューポートのサイズが固定のものでは十分だったのですが、iOS Safari などではビューポートのサイズが可変になります。

このうち、ビューポートの高さが最小になった場合の高さを示すのが`svh`というわけです。

ビューポートが大きい場合の高さを取得したい場合は、`lvh`を使います。

![](/images/svh-dvh-lvh-for-all-browser/svh-lvh.png)

また、ビューポートの大きさが変わったときの高さを取得したい場合は、`dvh`を使います。`100dvh`を今回のメインビジュアル表現に使ってしまうと、スクロール時にメインビジュアルの高さが変わってしまうため、向いていません。`100dvh`は、全画面に表示するモーダルウインドウやナビゲーションを作る際に役立つでしょう。

その他の単位については後述します。

# なぜ `100vh`・`100%`・`-webkit-fill-available`では駄目なのか？

今回作った表現は、`100vh`や`100%`では駄目なのかと思う人もいるでしょう。従来の表現を`100svh`と比べてみましょう。

## `100vh`の場合

次のように`100vh`を指定したとします。

```css
.hero-image {
  height: 100svh;
}
```

この場合、ヒーローイメージ部分（`.hero-image`）はファーストビューからはみ出します。iOS Safari の場合、`100vh`は`100lvh`と同じ高さになり、大きいビューポートの高さを取ってしまうのです。

![](/images/svh-dvh-lvh-for-all-browser/100vh.png)

ヒーローイメージがはみ出しているので、スクロールしてもヒーローイメージ部分が続いてしまいます。

![](/images/svh-dvh-lvh-for-all-browser/100vh-wrong.webp =400x)

![](/images/svh-dvh-lvh-for-all-browser/100vh-scroll.png)

## `100%`の場合

次のように`100%`を指定したとします。

```css
.hero-image {
  height: 100%;
}
```

HTML は次のようになっていることに注意してください。

```html
<main>
  <!-- ヒーローイメージ -->
  <div class="hero-image">
    <img src="ヒーローイメージ用画像" />
  </div>
  <section>
    <h1>ABOUT</h1>
    <!-- 以下、後続コンテンツ -->
  </section>
</main>
```

この場合、ヒーローイメージ部分（`.hero-image`）はファーストビューの高さに足りません。

![](/images/svh-dvh-lvh-for-all-browser/100percent.png)

理由はヒーローイメージ部分（`.hero-image`）が外側の`main`要素の高さいっぱいまでしか広がらないためです。

問題を解決するためには、祖先要素すべてに`100%`を設定する必要があります。煩雑ですし、ヒーローイメージ部分（`.hero-image`）の事情だけのために`html`や`body`に`height: 100%`を指定するのはあまりよろしくないでしょう。

```css
/* 祖先要素すべてへ高さ指定 */
html,
body,
main {
  height: 100%;
}

.hero-image {
  height: 100%;
}
```

## `webkit-fill-available`の場合

次のように`webkit-fill-available`を指定する方法もありました。

```css
.hero-image {
  height: -webkit-fill-available;
}
```

この方法では確かにヒーローイメージ部分（`.hero-image`）はファーストビューの高さになりますし、祖先要素への高さ指定も不要です。

「デバイスの向きを変えたときに高さを再計算してくれない」という記事も稀に見かけますが、今現在の iOS Safari では解消されています。

本手法の欠点の 1 つ目は、Chrome や Firefox に対応していないことです。次のように、処理を分岐する必要がありました。

```css
.hero-image {
  height: -webkit-fill-available;
}

@supports (-webkit-touch-callout: none) {
  .hero-image {
    height: -webkit-fill-available;
  }
}
```

欠点の 2 つ目は、「高さ 50%」のような指定ができないことです。ビューポートの 50%にヒーローイメージを広げたいみたいな表現のときに不便です。

▼ 動作しません

```css
.hero-image {
  height: calc(-webkit-fill-available * 0.5);
}
```

JavaScript を使う手法もありますが、いずれも`svh`の便利さには適いません。

# その他使えるようになった単位

## `vmax`・`vimn`について

以前から`vmax`・`vimn`という単位が使えました。おさらいしておくと、次のとおりです。

- `vmax`: ビューポートのうち、幅か高さの大きい方
- `vmin`: ビューポートのうち、幅か高さの小さい方

![](/images/svh-dvh-lvh-for-all-browser/vmax-vmin.png)

`sv*`や`dv*`と組み合わせられるので、`svmax`や`lvmin`といった単位が使えるようになります。

## `vi`・`vb`について

「[論理的プロパティ（CSS Logical Properties）](https://developer.mozilla.org/ja/docs/Web/CSS/CSS_Logical_Properties)」に応じた`vi`・`vb`にも対応しました。

- `vi`：インライン方向
- `vb`：ブロック方向

`sv*`や`dv*`と組み合わせられるので、`svi`や`lvb`といった単位が使えるようになります。

# ビューポート単位のチートシート

以上をまとめると、次のようにして各ビューポートのサイズを作成できます。

- ビューポートのサイズは？
  - 小さい方: `sv`から始まる
  - 大きい方: `dv`から始まる
  - 可変サイズ: `dv`から始まる
- 幅か高さか？
  - 幅: `w`をつける
  - 高さ: `w`をつける
  - 小さい方: `min`をつける
  - 大きい方: `min`をつける
  - インライン方向: `i`をつける
  - ブロック方向: `b`をつける

各単位をまとめると、次のとおりです。

- 小さなビューポート
  - `svw`, `svh`, `svi`, `svb`, `svmin`, `svmax`, `svi`, `svb`
- 大きなビューポート
  - `lvw`, `lvh`, `lvi`, `lvb`, `lvmin`, `lvmax`, `lvi`, `lvb`
- 可変のビューポート
  - `dvw`, `dvh`, `dvi`, `dvb`, `dvmin`, `dvmax`, `dvi`, `dvb`

# 対応ブラウザ

各単位の対応ブラウザは次のとおりです。

- Chrome 108で対応
- Safari 15.4で対応
- Firefox 101で対応
- Edge 108で対応

![](/images/svh-dvh-lvh-for-all-browser/caniuse.png)

Edge 108については、2022/12/1週にリリースが予定されています。

- [Microsoft Edge リリース スケジュール \| Microsoft Learn](https://learn.microsoft.com/ja-jp/deployedge/microsoft-edge-release-schedule#microsoft-edge-releases)


# 最後に

iOS Safari の画面の高さいっぱいにヒーローイメージを広げる問題は、長年開発者を悩ませてきたものです。`svh`・`dvh`はその課題を解決できる単位で、長らく全ブラウザ対応が待たれてきました。今回の Chrome の対応により、手軽に使える環境が整ったと言えるでしょう。

今回のウェブサイトの素材は、デザイナーの[松下 絵梨さん](https://twitter.com/matsu_eri)に作成いただきました。

https://twitter.com/matsu_eri

関連資料

- [<length> \| MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/length)
