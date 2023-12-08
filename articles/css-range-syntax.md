---
title: "ついにSafariも。 media queryの範囲指定をより直感的に書ける記法が全ブラウザ対応へ"
emoji: "🫨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css" ]
published: true
---

筆者が CSS を学び始めたとき、media query で画面サイズに応じてスタイルを変える方法が大変ニガテでした😭 `min`？ `max`？ 未満のときはどうするの？ `and` で繋げなきゃいけないの？ 長くない？ と疑問に思いながら、今日まで長い年月を過ごしてきました。

本日（2023/03/28）、Safari 16.4 がリリースされ、 media query のrange（範囲）記法に対応しました。

▼ 従来の media query の範囲指定

```css
@media (min-width: 600px) and (max-width: 799px) {
}
```

▼ 今後

```css
@media (600px <= width < 800px) {
}
```

本記事では、従来の記法の課題、range 記法の使い方、`@container` クエリとの組み合わせ方、今すぐ全ブラウザ向けに使う方法まで、デモを交えながら解説します。


# 従来の記法の課題

CSS Media Query は、CSS を特定の条件下で適用するための機能です。たとえば、画面の横幅が特定の大きさを超えた場合に、異なるスタイルを適用できます。

従来のMedia Queryで、画面の横幅に応じたスタイルを適用する場合、`min-**`・`max-**`・`and`といった記述を使う必要がありました。

```css
/* 画面サイズ 600px 未満 */
@media (max-width: 599px) {
  /* スタイルルール */
}

/* 画面サイズ 600px 以上 1200px 未満 */
@media (min-width: 601px) and (max-width: 1200px) {
  /* スタイルルール */
}

/* 画面サイズ 1200px 以上 */
@media (min-width: 1201px) {
  /* スタイルルール */
}
```

この方法にはいくつかの課題があります。

1つ目は、冗長で長いことです。プログラミング言語に慣れている方であれば、 `width <= 600px` のような表記をしたいと思うでしょう。

2つ目は、**「未満」や「より大きい」の表現ができないことです**。
`min-*`や`max-*`は、「○○px以上・○○px以下」にしか対応していません。したがって、たとえばブレークポイント `600px` でスタイルを分けようとすると、次のような書き方をする必要があります。

```css
@media (max-width: 599px) {
  /* 599px 以下のスタイル */
}

@media (min-width: 600px) {
  /* 600px 以上のスタイル */
}
```

この場合、小数点のビューポートが考慮されません。解決するのであれば、次のような面倒な書き方をする必要があります。

```css
@media (max-width: 599.99px) {
  /* 600px 未満のスタイル */
}

@media (min-width: 600px) {
  /* 600px 以上のスタイル */
}
```

参考： [Media Queries Level 4 - Using “min-” and “max-” Prefixes On Range Features](https://www.w3.org/TR/mediaqueries-4/#mq-min-max)


# 新しい Media Query の Range 記法

新しく使えるようになった Media Query の range （範囲）指定の方法では、次のように記述できます。

```css
/* 画面サイズ 600px 未満 */
@media (width < 600px) {
  /* スタイルルール */
}

/* 画面サイズ 600px 以上 1200px 未満 */
@media (600px <= width < 1200px) {
  /* スタイルルール */
}

/* 画面サイズ 1200px 以上 */
@media (1200px <= width) {
  /* スタイルルール */
}
```

従来の記法よりも、短く、直感的です。また、「以上なのか」「未満なのか」といったこともわかりやすくなります。

こちらの URL からデモを確認できます。

@[codepen](https://codepen.io/tonkotsuboy/pen/vYdgKqM)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/vYdgKqM)

▼ 動作している様子

https://twitter.com/tonkotsuboy_com/status/1640516966170644480


## 記号の確認

各記号の説明は次のとおりです。

| 記号 | 説明 |
| --- | --- |
| `<=` | 以下 |
| `<` | 未満 |
| `>=` | 以上 |
| `>` | より大きい |

筆者的には、`<=` と `<` だけを使い、「小さい数 < 大きい数」と並べて記述するのがわかりやすいと思います。ただの好みの問題ですので、ご自身のプロジェクトにあわせて使い分けましょう。

# コンテナクエリ `@container` にも使える

先日、全ブラウザでコンテナクエリ `@container` が使えるようになりました。詳しくは次の記事で解説しましたが、 ウインドウサイズではなく要素のサイズを基準にレスポンシブ対応できるものです。

https://zenn.dev/tonkotsuboy_com/articles/css-container-query

次の HTML にて、 `.container` 要素が `300px` 以上のときに文字を赤色にしてみます。 

```html
<div class="container">
  <p>リサイズしてください</p>
</div>
```

従来の記法であれば、次のように記述していました。

```css
.container {
  container-type: inline-size;
}

@container (min-width: 300px) {
  p {
    color: red;
  }
}
```

`@container` と range 記法を組み合わせると、次のように記述できます。

```css
.container {
  container-type: inline-size;
}

@container (300px <= width) {
  p {
    color: red;
  }
}
```

こちらの URL からデモを確認できます。

@[codepen](https://codepen.io/tonkotsuboy/pen/GRXexYY)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/GRXexYY)

# Safari 16.4の浸透を待たず、今すぐ全ブラウザ向けに使う方法

Safari 16.4 の登場で全ブラウザに range 記法が対応したとは言え、ユーザーがSafari 16.4 にバージョンアップするまでには時間がかかるでしょう。

今すぐ range 記法を使いたいのであれば、PostCSS とプラグインを使うとよいです。

range 記法を従来の記法に変換してくれます。

▼ 変換前

```css
@media screen and (width >= 500px) and (width <= 1200px) {
  .bar {
    display: block;
  }
}
```

▼ 変換後

```css
@media screen and (min-width: 500px) and (max-width: 1200px) {
  .bar {
    display: block;
  }
}
```


- [postcss/postcss\-media\-minmax: Writing simple and graceful Media Queries\!](https://github.com/postcss/postcss-media-minmax)


# 対応ブラウザ

range 記法の対応ブラウザは次のとおりです。本日（2023/03/28）に Safari 16.4 が対応したことにより、全ブラウザ対応となりました。

各ブラウザ対応開始バージョンは次のとおり。使用する際は、プロジェクトの対象ブラウザに気をつけましょう。

- Safari 16.4 で対応（NEW！）
- Chrome 104 で対応
- Edge 104 で対応
- Firefox 63 で対応

![](/images/range/caniuse.png)

- [Media Queries: Range Syntax \| Can I use](https://caniuse.com/css-media-range-syntax)


# 関連資料

- [Media Queries Level 4](https://www.w3.org/TR/mediaqueries-4/) 
- [WebKit Features in Safari 16\.4 \| WebKit](https://webkit.org/blog/13966/webkit-features-in-safari-16-4/)




