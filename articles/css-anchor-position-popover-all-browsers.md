---
title: "Popover API + CSS Anchor Positioningが全ブラウザ対応。ポップオーバーをJSなしで実装できて最高"
emoji: "🎯"
type: "tech"
topics: ["css", "html", "popover"]
published: false
---


HTMLにはポップオーバー APIという、その名の通りポップオーバーを表現できる便利なHTMLがあります。ESCの閉じる処理やフォーカス管理などが、JavaScriptなしで実装できます。2025年8月頃、全ブラウザで対応しました。
しかし、ポップオーバーの位置を決める際、結局は複雑なJavaScriptを記述する必要があり、HTML・CSSだけで手軽にポップオーバーを使えるというわけではありませんでした。

その問題、解決します。

2026年1月に、Firefox 147がリリースされ、「CSS Anchor Positioning」が全ブラウザ対応しました。CSS Anchor Positioningとは、要素の位置を別の要素の位置に合わせることができるCSSの機能です。Firefox以外のブラウザでは昔から対応していましたが、長らくFirefoxだけ対応していなかったのです。

本記事では、Popover APIとCSS Anchor Positioningを組み合わせて、ポップオーバーをJavaScriptなしで実装する方法を解説します。

これをずっとやりたかった…！

# Popover APIとは

従来、ポップオーバーを実装するには、大量のJavaScriptが必要でした。
大量のイベントリスナー、複雑な位置計算のロジック、スクロールやリサイズへの対応も必要でした。

次に示すのは、ポップオーバーの実装例です。

```javascript
// 従来のJavaScript実装
const button = document.querySelector('.trigger');
const popover = document.querySelector('.popover');

// 表示・非表示の制御
button.addEventListener('click', () => {
  popover.classList.toggle('open');
});

// ESCキーで閉じる
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    popover.classList.remove('open');
  }
});

// 背景クリックで閉じる
document.addEventListener('click', (e) => {
  if (!popover.contains(e.target) && !button.contains(e.target)) {
    popover.classList.remove('open');
  }
});

// さらに位置計算も必要...
function updatePosition() {
  const rect = button.getBoundingClientRect();
  popover.style.top = `${rect.bottom + 8}px`;
  popover.style.left = `${rect.left}px`;
}
```

## Popover APIで表示・非表示が簡単に

Popover APIを使うと、ポップオーバーの表示・非表示がHTML属性だけで実現できます。

次の例では、ボタン要素をクリックするとポップオーバーが開くシンプルな例です。

```html
<button popovertarget="my-popover">開く</button>

<div id="my-popover" popover>
  <p>ポップオーバーの内容</p>
</div>
```

たったこれだけで、以下の機能が自動的に実装されます。JavaScriptは一切不要です。

- クリックで開閉
- ESCキーで閉じる
- 背景クリックで閉じる
- 適切なフォーカス管理

## Popover APIの基本

`popovertarget`属性で開閉ボタンを指定し、`popover`属性で要素をポップオーバーとして宣言します。

| 属性 | 役割 |
|------|------|
| `popover` | 要素をポップオーバーとして宣言 |
| `popovertarget` | クリック時に開閉するポップオーバーのIDを指定 |
| `popovertargetaction` | 動作を指定（`toggle` / `show` / `hide`） |

https://developer.mozilla.org/ja/docs/Web/API/Popover_API

# Popover APIがあっても、結局位置計算にはJavaScriptが必要

Popover APIは素晴らしい機能ですが、**ポップオーバーの位置を指定する機能はありません**。ボタンの下や右側にポップオーバーを表示したい場合、従来はJavaScriptで位置を計算する必要がありました。

次の例では、ボタンの位置を取得して、ポップオーバーの位置を計算し、CSSで位置を指定しています。リサイズやスクロールのたびに再計算が必要なので、処理負荷も大きくなってしまいます。

```javascript
// ボタンの位置を取得して、ポップオーバーの位置を計算
const rect = button.getBoundingClientRect();
popover.style.top = `${rect.bottom + 8}px`;
popover.style.left = `${rect.left}px`;
```

# 全ブラウザ対応したCSS Anchor Positioningで位置指定をする

CSS Anchor Positioningを使うと、**CSSだけでポップオーバーの位置を指定できます**。

次の例では、ボタンをアンカーとして定義し、ポップオーバーをアンカーに関連付けています。

```html
<button class="button" popovertarget="my-popover">開く</button>

<div id="my-popover" popover class="popover">
  ポップオーバーの内容
</div>
```

```css
/* ボタンをアンカーとして定義 */
.button {
  anchor-name: --my-anchor;
}

/* ポップオーバーをアンカーに関連付け */
.popover {
  position-anchor: --my-anchor;
  top: anchor(bottom);  /* アンカーの下端に配置 */
  left: anchor(left);   /* アンカーの左端に揃える */
}
```

- 👍 JavaScriptによる位置計算が不要
- 👍 リサイズやスクロールに自動追従
- 👍 宣言的で可読性が高い

## Anchor Positioningの基本

Anchor Positioningは3つのステップで使用します。

**ステップ1**: `anchor-name`でアンカーを定義します。

```css
.button {
  anchor-name: --my-anchor;
}
```

**ステップ2**: `position-anchor`でアンカーに関連付けます。

```css
.popover {
  position-anchor: --my-anchor;
}
```

**ステップ3**: `anchor()`関数で位置を指定します。

```css
.popover {
  top: anchor(bottom);   /* アンカーの下端 */
  left: anchor(left);    /* アンカーの左端 */
}
```

▼ 参考: MDN - CSS Anchor Positioning
https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning

# デモ1: さまざま位置のポップオーバー

`position-area`プロパティを使うと、ポップオーバーをアンカーの上下左右、さらに斜め方向にも配置できます。

```html
<!-- 上に表示 -->
<button id="anchor-top" popovertarget="popover-top">上に表示</button>
<div id="popover-top" popover class="popover-top">
  <p><code>position-area: block-start</code></p>
</div>

<!-- 右に表示 -->
<button id="anchor-right" popovertarget="popover-right">右に表示</button>
<div id="popover-right" popover class="popover-right">
  <p><code>position-area: inline-end</code></p>
</div>

<!-- 左下に表示 -->
<button id="anchor-bottom-left" popovertarget="popover-bottom-left">左下に表示</button>
<div id="popover-bottom-left" popover class="popover-bottom-left">
  <p><code>position-area: block-end inline-start</code></p>
</div>
```

```css
/* 各ボタンをアンカーとして定義 */
#anchor-top { anchor-name: --anchor-top; }
#anchor-right { anchor-name: --anchor-right; }
#anchor-bottom-left { anchor-name: --anchor-bottom-left; }

/* 上に表示 */
.popover-top {
  position-anchor: --anchor-top;
  position-area: block-start;
}

/* 右に表示 */
.popover-right {
  position-anchor: --anchor-right;
  position-area: inline-end;
}

/* 左下に表示 */
.popover-bottom-left {
  position-anchor: --anchor-bottom-left;
  position-area: block-end inline-start;
}
```

## position-areaの値

| 値 | 配置位置 |
|---|---------|
| `block-start` | 上 |
| `block-end` | 下 |
| `inline-start` | 左（LTRの場合） |
| `inline-end` | 右（LTRの場合） |

2つの値を組み合わせることで、斜め方向にも配置できます。

| 値 | 配置位置 |
|---|---------|
| `block-start inline-end` | 右上 |
| `block-end inline-start` | 左下 |

# デモ2: サブメニューナビゲーション

実践的な例として、サイドメニューのサブメニューをAnchor Positioningで実装します。

```html
<nav class="menu">
  <ul class="menu-list">
    <li class="menu-item">
      <button class="menu-link" id="task-menu-anchor" popovertarget="task-submenu">
        ✅ タスク管理
        <span class="chevron">›</span>
      </button>
      <ul class="submenu" popover id="task-submenu">
        <li><a href="#">今日のタスク</a></li>
        <li><a href="#">週次タスク</a></li>
        <li><a href="#">アーカイブ</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

`popovertarget`属性は`<button>`要素で使用します。`<a>`要素では期待通りに動作しません。

```css
/* メニューリンクをアンカーとして定義 */
#task-menu-anchor {
  anchor-name: --task-menu;
}

/* サブメニューの位置指定 */
.submenu {
  position-anchor: --task-menu;
  top: anchor(--task-menu top);
  left: anchor(--task-menu right);
}
```

## :has()セレクターでシェブロンアイコンを自動表示

サブメニューを持つ項目には、自動的に矢印アイコンを表示できます。

```css
/* サブメニューがある項目に矢印アイコンを表示 */
.menu-item:has(.submenu) > .menu-link::after {
  content: "▶";
  margin-left: auto;
  color: #666;
}
```

`:has()`セレクターを使うことで、**サブメニューの有無をCSSだけで検出**し、自動的にアイコンを表示できます。

# デモ3: ヘッダーのユーザーメニュー

よくあるパターンとして、ヘッダーのユーザーアイコンからドロップダウンメニューを表示する例を紹介します。

```html
<header class="header">
  <button popovertarget="user-menu" class="user-icon">
    <img src="avatar.png" alt="ユーザー" />
  </button>
  <div id="user-menu" popover class="dropdown-menu">
    <a href="#">プロフィール</a>
    <a href="#">設定</a>
    <a href="#">ログアウト</a>
  </div>
</header>
```

```css
.user-icon {
  anchor-name: --user-icon;
}

.dropdown-menu {
  position-anchor: --user-icon;
  position-area: block-end inline-end;
  margin-top: 8px;
}
```

`position-area: block-end inline-end`を指定することで、ユーザーアイコンの**下かつ右端揃え**で配置されます。ヘッダー右端のアイコンからドロップダウンを表示するとき、メニューが画面外へはみ出さないための指定です。

# position-area vs anchor()関数

Anchor Positioningには2つの位置指定方法があります。

## position-area: シンプルな配置向け

```css
.popover {
  position-anchor: --my-anchor;
  position-area: block-end;  /* アンカーの下に配置 */
}
```

- 👍 直感的で書きやすい
- 👍 論理プロパティ対応（RTLでも正しく動作）

## anchor()関数: 精密な制御向け

```css
.popover {
  position-anchor: --my-anchor;
  top: calc(anchor(bottom) + 8px);
  left: anchor(center);
}
```

- 👍 `calc()`と組み合わせて余白を調整可能
- 👍 中央揃えなど細かい位置指定が可能

用途に応じて使い分けましょう。

# ブラウザ対応状況

CSS Anchor Positioningは、2026年1月に全主要ブラウザで対応しました。

| ブラウザ | 対応バージョン |
|---------|---------------|
| Chrome | 125以降 |
| Edge | 125以降 |
| Safari | 18.4以降 |
| Firefox | 147以降 🎉 |

▼ Can I use - CSS Anchor Positioning
https://caniuse.com/css-anchor-positioning

# まとめ

Popover API + CSS Anchor Positioningが全ブラウザ対応して最高です。

筆者は、モーダルやツールチップを実装する度に、位置計算のJavaScriptコードの複雑さに震えていました。リサイズ対応、スクロール対応、viewport外へのはみ出し対応...考えることが多すぎました。

CSS Anchor Positioningのおかげで、その複雑さがなくなるので一安心です。

今後のプロジェクトでは積極的に活用していきます。

# 参考リンク

- [MDN - Popover API](https://developer.mozilla.org/ja/docs/Web/API/Popover_API)
- [MDN - CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)
- [MDN - Using CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Anchor_positioning/Using)
- [Can I use - CSS Anchor Positioning](https://caniuse.com/css-anchor-positioning)
