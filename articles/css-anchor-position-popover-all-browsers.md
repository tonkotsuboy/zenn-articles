---
title: "Popover API + CSS Anchor Positioningが全ブラウザ対応。ポップオーバーをJSなしで実装できて最高"
emoji: "🎯"
type: "tech"
topics: ["css", "html", "popover"]
published: false
---

開発者が長い間求めていた機能がついに実現されました。

2025年、Firefox 147のリリースにより、CSS Anchor Positioningが全ブラウザに対応しました。Popover APIと組み合わせることで、**JavaScriptなしでポップオーバーの表示・非表示・位置指定がすべて実現できます**。

従来、モーダルやツールチップの実装で苦労していた以下の処理が、HTMLとCSSだけで完結します。

- 表示・非表示の制御
- ESCキーでの閉じる処理
- 背景クリックでの閉じる処理
- フォーカス管理
- **位置計算**

これをずっとやりたかった…！

## 従来のポップオーバー実装の苦労

従来、ポップオーバーやモーダルを実装するには、大量のJavaScriptが必要でした。

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

- 👎 大量のイベントリスナーが必要
- 👎 位置計算のロジックが複雑
- 👎 スクロールやリサイズへの対応も必要

この問題は、UIが複雑になるほど深刻になります。

## Popover APIで表示・非表示が簡単に

Popover APIを使うと、ポップオーバーの表示・非表示がHTML属性だけで実現できます。

```html
<button popovertarget="my-popover">開く</button>

<div id="my-popover" popover>
  <p>ポップオーバーの内容</p>
</div>
```

たったこれだけで、以下の機能が自動的に実装されます。

- 👍 クリックで開閉
- 👍 ESCキーで閉じる
- 👍 背景クリックで閉じる
- 👍 適切なフォーカス管理

JavaScriptは一切不要です。

### Popover APIの基本

`popovertarget`属性で開閉ボタンを指定し、`popover`属性で要素をポップオーバーとして宣言します。

| 属性 | 役割 |
|------|------|
| `popover` | 要素をポップオーバーとして宣言 |
| `popovertarget` | クリック時に開閉するポップオーバーのIDを指定 |
| `popovertargetaction` | 動作を指定（`toggle` / `show` / `hide`） |

▼ 参考: MDN - Popover API
https://developer.mozilla.org/ja/docs/Web/API/Popover_API

## 課題: 位置指定にはJavaScriptが必要だった

Popover APIは素晴らしい機能ですが、**ポップオーバーの位置を指定する機能は含まれていません**。

ボタンの下や右側にポップオーバーを表示したい場合、従来はJavaScriptで位置を計算する必要がありました。

```javascript
// ボタンの位置を取得して、ポップオーバーの位置を計算
const rect = button.getBoundingClientRect();
popover.style.top = `${rect.bottom + 8}px`;
popover.style.left = `${rect.left}px`;
```

- 👎 リサイズやスクロールのたびに再計算が必要
- 👎 位置計算ライブラリ（Popper.jsなど）に依存しがち

## CSS Anchor Positioningで位置指定も解決

CSS Anchor Positioningを使うと、**CSSだけでポップオーバーの位置を指定できます**。

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

### Anchor Positioningの基本

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

## デモ1: シンプルなポップオーバー

Popover APIとAnchor Positioningを組み合わせた基本的な実装です。

```html
<button id="popover-anchor" popovertarget="anchor-popover">
  ポップオーバーを開く
</button>

<div id="anchor-popover" popover>
  <h2>アンカーされたポップオーバー</h2>
  <p>CSSのanchor-positionを使って、ボタンの横に表示されています。</p>
</div>
```

```css
/* ボタンをアンカーとして定義 */
#popover-anchor {
  anchor-name: --popover-anchor;
}

/* ポップオーバーの位置指定 */
[popover] {
  margin: 0;
  padding: 16px;
  border: none;
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
  width: min(300px, 90vw);

  /* Anchor Positioning */
  position-anchor: --popover-anchor;
  top: anchor(top);
  left: calc(anchor(right) + 16px);
}
```

### コードのポイント

| プロパティ | 役割 |
|-----------|------|
| `anchor-name: --popover-anchor` | ボタンに名前を付けてアンカーとして定義 |
| `position-anchor: --popover-anchor` | ポップオーバーをアンカーに関連付け |
| `top: anchor(top)` | ポップオーバーの上端をアンカーの上端に合わせる |
| `left: calc(anchor(right) + 16px)` | ポップオーバーをアンカーの右側に16pxの間隔で配置 |

`anchor()`関数は`calc()`と組み合わせられるので、余白の調整も簡単です。

## デモ2: サブメニューナビゲーション

実践的な例として、サイドメニューのサブメニューをAnchor Positioningで実装します。

```html
<nav class="menu">
  <ul class="menu-list">
    <li class="menu-item">
      <a class="menu-link" href="#" id="task-menu-anchor" popovertarget="task-submenu">
        タスク管理
      </a>
      <ul class="submenu" popover id="task-submenu">
        <li><a href="#">今日のタスク</a></li>
        <li><a href="#">週次タスク</a></li>
        <li><a href="#">アーカイブ</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

```css
/* メニューリンクをアンカーとして定義 */
#task-menu-anchor {
  anchor-name: --task-menu;
}

/* サブメニューの位置指定 */
.submenu {
  position: absolute;
  margin: 0;
  padding: 0.5rem;
  list-style: none;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);

  /* Anchor Positioning */
  position-anchor: --task-menu;
  top: anchor(--task-menu top);
  left: anchor(--task-menu right);
  margin-left: 0.75rem;
}
```

### :has()セレクターでシェブロンアイコンを自動表示

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

## デモ3: ヘッダーのユーザーメニュー

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
  position-area: block-end;
  margin: 0;
}
```

### position-areaでシンプルに配置

`position-area`プロパティを使うと、より直感的に位置を指定できます。

| 値 | 配置位置 |
|---|---------|
| `block-start` | 上 |
| `block-end` | 下 |
| `inline-start` | 左（LTRの場合） |
| `inline-end` | 右（LTRの場合） |

`anchor()`関数を使った細かい制御が不要な場合は、`position-area`がシンプルでおすすめです。

## position-area vs anchor()関数

Anchor Positioningには2つの位置指定方法があります。

### position-area: シンプルな配置向け

```css
.popover {
  position-anchor: --my-anchor;
  position-area: block-end;  /* アンカーの下に配置 */
}
```

- 👍 直感的で書きやすい
- 👍 論理プロパティ対応（RTLでも正しく動作）

### anchor()関数: 精密な制御向け

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

## ブラウザ対応状況

CSS Anchor Positioningは、2025年に全主要ブラウザで対応しました。

| ブラウザ | 対応バージョン |
|---------|---------------|
| Chrome | 125以降 |
| Edge | 125以降 |
| Safari | 18.4以降 |
| Firefox | 147以降 🎉 |

▼ Can I use - CSS Anchor Positioning
https://caniuse.com/css-anchor-positioning

## まとめ

Popover API + CSS Anchor Positioningが全ブラウザ対応して最高です。

筆者は、モーダルやツールチップを実装する度に、位置計算のJavaScriptコードの複雑さに震えていました。リサイズ対応、スクロール対応、viewport外へのはみ出し対応...考えることが多すぎました。

CSS Anchor Positioningのおかげで、その複雑さがなくなるので一安心です。

今後のプロジェクトでは積極的に活用していきます。

## 参考リンク

- [MDN - Popover API](https://developer.mozilla.org/ja/docs/Web/API/Popover_API)
- [MDN - CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)
- [MDN - Using CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Anchor_positioning/Using)
- [Can I use - CSS Anchor Positioning](https://caniuse.com/css-anchor-positioning)
