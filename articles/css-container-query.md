---
title: "コンテナクエリ @container が全ブラウザ対応。新時代のレスポンシブ対応を完全理解する"
emoji: "🎁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css" ]
published: true
publication_name: moneyforward
---

本日（2/14）、ついに Firefox でコンテナクエリに対応しました。Chrome・Edge・Safari はすでに現行ブラウザで対応済みのため、全ブラウザにてコンテナクエリが使えることになります💐

従来、レスポンシブ対応でレイアウトを変えるには `@media` を使ってウインドウサイズを基準にするかありませんでした。コンテナクエリ `@container`を使うと任意の要素を基準にできるので、**「A要素の横幅が `500px` 以下のとき、子要素のレイアウトを変える」などが手軽に実現できます**。


![](/images/container-query/basic_media.png)

![](/images/container-query/basic_container.png)

とくにコンポーネントベース開発が主流の現在においては、コンテナクエリをマスターすることで、より柔軟でラクなレスポンシブ対応が可能になります。

本記事では、コンテナクエリの基本、メリット、デモまでをできるだけ詳しく解説し、コンテナクエリを理解することを目標とします。

# 今回のデモ

次のようなSNS風のウェブページを作っているとします。アプリケーションはウェブページに対応しており、ナビゲーションが上・左と移動したり、右下部分が2カラムになったりします。

![](/images/container-query/sns_goal_large.png =1000x)
*画面サイズ：大*

![](/images/container-query/sns_goal_medium.png =600x)
*画面サイズ：中*

![](/images/container-query/sns_goal_small.png =400x)
*画面サイズ：小*

あなたはSNSのつぶやきの下にあるリアクション用バーを作ることになりました。 ただし、**配置される領域の横幅に応じて、テキスト（返信する・いいね・View数など）部分を非表示にする**必要があります。


![](/images/container-query/sns_reaction_large.png)
*配置される領域の横幅が大きいとき*

![](/images/container-query/sns_reaction_small.png =470x)
*配置される領域の横幅が小さいとき*


# 従来のレスポンシブレイアウトの課題


リアクション用バーをつくるとき、あなたならどうやってレスポンシブに対応しますか？ もし `@media` を使って実装しようとすると、大変なことになります。 ウインドウサイズによって、細かくレイアウトが変わってしまうからです。

- 画面幅 `1480px` 以上のとき
  - メインエリアのリアクションバー：大表示
  - カラムが切り替わるエリアのリアクションバー：大表示
- 画面幅 `1080px` 以上 `1480px` 未満のとき
  - メインエリアのリアクションバー：大表示
  - カラムが切り替わるのリアクションバー：小表示
- 画面幅 `859px` 以上 `1080px` 未満のとき
  - メインエリアのリアクションバー：大表示
  - カラムが切り替わるエリアのリアクションバー：大表示
  - ※ 右下は1カラム
- 画面幅 `825px` 以上 `859px` 未満のとき
  - メインエリアのリアクションバー：大表示
  - カラムが切り替わるエリアのリアクションバー：小表示
- 以下略

（※ サイズは適当です）

CSS で記述するとこうなります。

```css
@media (min-width: 1480px) {
  .reaction-bar-1 {
    /* 大レイアウト */
  }

  .reaction-bar-2 {
    /* 大レイアウト */
  }
}

@media (min-width: 1080px) and (max-width: 1480px) {
  .reaction-bar-1 {
    /* 大レイアウト */
  }

  .reaction-bar-2 {
    /* 小レイアウト */
  }
}

@media (min-width: 859px) and (max-width: 1080px) {
  .reaction-bar-1 {
    /* 大レイアウト */
  }

  .reaction-bar-2 {
    /*  大レイアウト  */
  }
}

/* 以下略 */
@media...
@media...
@media...
```

それだけではありません。ナビゲーションエリアのサイズは固定ではありません。ハンバーガーメニューにより開閉する機能を持つかもしれませんし、デザイン変更によりサイズが変わるかもしれません。それらに対応する `@media` を記述するのは、見通しの悪いコードになります。

# ウインドウではなく、任意の要素を基準にする

問題を解決するには、ウインドウサイズに応じたレイアウト変更ではなく、**リアクションバーを包含する先祖要素のサイズを基準としたレイアウト変更にする必要があります**。

すなわち、リアクションバーを配置している親要素の横幅を基準として次のようにすればよいのです。リアクションバーは、基準となる要素の横幅だけを気にすればいいので、**条件分岐がたった2つになります**。

- 親要素の横幅が大きいならテキストを表示
- 親要素の横幅が小さいならテキストを表示

これが実現できるようになったのが、2023/02/14に全ブラウザで対応となった「コンテナクエリ（ `@container` ）」です。

`@media` と `@container` の基準の違いは次のとおりです。

![](/images/container-query/basic_media.png)

![](/images/container-query/basic_container.png)

React・Vue・Svelte・Astro などのコンポーネントをベースとした開発と特に相性がいいです。コンポーネントの開発者は、**そのコンポーネントがどこにどう配置されるかを気にする必要がなく、あくまで自分がコンテナサイズに対してどのような表示になるかだけの関心を持てばよい**からです。

# 簡単なデモでコンテナクエリの仕様を理解する

リアクションバーの実装をする前に、まずは 簡単なデモを作りながら、コンテナクエリの動作を理解しましょう。

次のように、`.container` 要素の横幅が `300px` 以上のときは、`p`要素の文字を赤色で大きなサイズにするデモを作ります。

![](/images/container-query/simple-goal.webp)

HTMLは次のとおりです。

```html

<div class="container">
  <p>リサイズしてください</p>
</div>
```

`.container` 要素の横幅が `300px` 以上のときは、`p`要素の文字を赤色で大きなサイズにします。

コンテナクエリを使うまでのステップは3つあります。

1. コンテナを決定する
2. コンテナの名前と方向を決める
3. クエリを書く

## ① 基準となるコンテナを決定する

コンテナクエリのポイントは、「どの要素をコンテナにするか」を指定することです。今回は、 `.container` 要素をコンテナにします。必ずしも親要素である必要はなく、レスポンシブ対応したい要素の祖先要素であれば大丈夫です。

## ② コンテナの名前と方向を決める

要素をコンテナにするには、「`container-type` プロパティ」を指定します。コンテナになった要素は「コンテナコンテキスト（container context または containment context）」と呼ばれます。

`container-type` には、次の値を指定できます。

- `size`: インライン方向・ブロック方向の両方
- `inline-size`: インライン方向のみ

ブロック方向だけを指定することは現状ではできません。

今回は、`.container` 要素の横幅を基準にしたいので、「`inline-size`」を指定します。

```css
.container {
  container-type: inline-size;
}
```

## ③ クエリを書く

コンテナが決まったので、そのコンテナを基準にしてどのようなスタイルをあてるかを記述します。

今回は「コンテナの横幅が `300px` 以上になったら」と書きたいので、次のようなクエリを記述します。 `@container` 内に記述されたスタイルは、最も近い祖先のコンテナテキストを探し、クエリに応じて適用されます。

```css
@container (min-width: 300px) {
  p {
    color: #f4481a;
    font-size: 26px;
    font-weight: bold;
  }
}
```

## `resize` プロパティで要素をリサイズ可能にしておくとデバッグしやすい

コンテナクエリを試す場合は、要素をリサイズ可能にしておくとデバッグしやすいです。CSS の `resize` プロパティを使うと要素をリサイズできるようになります。

```css
.container {
  resize: horizontal;
  overflow: auto;
}
```

`resize` プロパティは、次のような値でリサイズ方向を制御します。

- `both`: 垂直・水平方向
- `horizontal`: 水平方向
- `vertical`: 垂直方向

また、`overflow` プロパティを `visible` 以外にする必要があります。

- [resize \- CSS: カスケーディングスタイルシート \| MDN](https://developer.mozilla.org/ja/docs/Web/CSS/resize)

## 実行結果

実行結果は次のとおりです。

![](/images/container-query/simple-demo-small.png)
*`.container` 幅が `240px` のとき*

![](/images/container-query/simple-demo-large.png)
*`.container` 幅が `540px` のとき*


![](/images/container-query/simple-goal.webp)

こちらのURLからデモを確認できます。

@[codepen](https://codepen.io/tonkotsuboy/pen/OJwKdeo)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/OJwKdeo)


# リアクションバーのレスポンシブ対応

冒頭のSNSウェブサイトの、リアクションバーのレスポンシブ対応をします。

## HTML 構造

全体のHTMLは次のような構成になっています。

3つのリアクションバー `.reaction-bar` 要素があります。

```html
<div class="wrapper">
  <nav>ナビゲーションエリア</nav>
  <main class="main">
    <h2>メインエリア</h2>
    <div class="tweet">
      <p>よく晴れた気持ちのいい朝...（略）</p>
      <div class="reaction-bar">
        <!-- リアクションバー -->
      </div>
    </div>
    <section>
      <h3>1カラム・2カラムが切り替わるエリア</h3>
      <div class="section-inner">
        <div class="tweet">
          <p>「やれやれ…」私は立ち上がり、（略）</p>
          <div class="reaction-bar">
            <!-- リアクションバー -->
          </div>
        </div>
        <div class="tweet">
          <p>その創造物は想像を超えていたーーー。（略）</p>
          <div class="reaction-bar">
            <!-- リアクションバー -->
          </div>
        </div>
      </div>
    </section>
  </main>
</div>
```

リアクションバーのHTMLは次のようになっています。

```html
<div class="tweet">
  <p>つぶやき内容</p>
  <div class="reaction-bar">
    <div class="reaction">
      <button>返信ボタン</button>
      <span class="text">返信する</span>
      <span class="count">60</span>
    </div>
    <div class="reaction">
      <button>いいねボタン</button>
      <span class="text">いいね</span>
      <span class="count">120</span>
    </div>
    <!-- 略 -->
  </div>
```

リアクションバー内のテキスト `.text` 要素を、そのコンテナの横幅が小さくなったときに非表示にすれば、目的の表現を達成できます。

## コンテナを指定する

コンテナは何でもいいのですが、今回は `.reaction-bar` 要素が妥当でしょう。

`.reaction-bar` 要素をコンテナに指定するために、 `container-type` プロパティを指定します。今回はインライン方向を取り扱いたいので、 `inline-size` を指定します。

```css
.reaction-bar {
  container-type: inline-size;
}
```

## クエリを指定する

今回は、コンテナ `.reaction-bar` 要素の横幅が `560px` 以下になったときに、テキスト `.text` 要素を非表示にします。CSSは次の様に指定します。

```css
@container (max-width: 560px) {
  .text {
    display: none;
  }
}
```

## 挙動の確認

実行結果は次のとおりです。 **画面横幅によらず、コンテナの横幅に応じて**、テキスト `.text` 要素が非表示になります。

![](/images/container-query/sns_goal_large.png =1000x)

![](/images/container-query/sns_goal_movie.webp)


こちらのURLからデモを確認できます。別ウインドウで開いたほうが確認しやすいです。

@[codepen](https://codepen.io/tonkotsuboy/pen/PoBMMZw)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/PoBMMZw)


## コンテナには名前も指定できる

コンテナ要素（コンテナテキスト）には、`container-name` プロパティを使って名前も指定できます。複数のコンテナを指定するとき、どのコンテナを使うのか明示したいときに便利です。

```css
.container {
  container-type: inline-size;
  /** コンテナの名前 */
  container-name: my-container;
}
```

`container-type` プロパティと `container-name` プロパティは、 `container` プロパティでまとめて記述できます。 `コンテナ名 / コンテナタイプ` のように、順番や `/` に気をつけてください。

```css
.container {
  container: my-container / inline-size;
}
```

コンテナ名を `@container` で使うときは、`@container` のあとに名前を指定します。

```css
@container my-container (min-width: 300px) {
  p {
    color: #f4481a;
    font-size: 26px;
    font-weight: bold;
  }
}
```

こちらのURLからデモを確認できます。

@[codepen](https://codepen.io/tonkotsuboy/pen/bGxbXOB)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/bGxbXOB)

## ブラウザでデバッグする

Chrome の DevTools を使うと、コンテナクエリの確認がラクになります。

次の図は、コンテナサイズが小さくなったときに非表示にしたテキストにカーソルをあわせたものです。

![](/images/container-query/container-query-debug.png)

- 何がコンテナテキストになっているのか
- コンテナの現在のサイズ・type
- 現在適用されているクエリ

などが、視覚的にわかります。

Microsoft Edge・Firefox・Safariでも似たような機能が使えます。

# コンテナクエリに応じた単位

コンテナクエリとともに、コンテナクエリ用単位が使えるようになりました。

コンテナクエリ単位は、コンテナのサイズに対する割合のサイズを指定できるので、たとえば次のようなことができます。

- コンテナ横幅いっぱいに要素を広げる
- コンテナ高さ `50%` の高さを指定する

主に使うのは次の単位です。

- `cqw`: コンテナ横幅の `1%`
- `cqh`: コンテナ高さの `1%`

一見地味な機能ですが、利用シーンが多くあります。近日公開する別記事にてデモとともに詳しく解説します。

# 対応ブラウザ

コンテナクエリの対応ブラウザは次のとおりです。本日（米国時間2023/2/14）にFirefoxが対応したことにより、全ブラウザ対応となりました。

![](/images/container-query/firefox.png)
*Firefoxでの動作の様子*

各ブラウザ対応開始バージョンは次のとおり。使用する際は、プロジェクトの対象ブラウザに気をつけましょう。

- Chrome 105で対応（Android・デスクトップ共に）
- Edge 105で対応
- Safari 16.0で対応（iOS・デスクトップ共に）
- Firefox 110で対応（NEW！）

[![](/images/container-query/caniuse.png)](https://caniuse.com/css-container-query-units)

- [CSS Container Query Units \| Can I use](https://caniuse.com/css-container-query-units)

# 関連資料

- [CSS Container Queries \- CSS: Cascading Style Sheets \| MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries)
- [CSS Containment Module Level 3](https://www.w3.org/TR/css-contain-3/)
