---
title: "Anchor Positioningが全対応。HTML・CSSのポップオーバーが完全体に"
emoji: "🎯"
type: "tech"
topics: ["css", "html", "popover"]
published: true
publication_name: ubie_dev
---

HTMLのPopover APIを使えば、ESCキーで閉じる処理やフォーカス管理がJavaScriptなしで実装できます。しかし、ポップオーバーの**位置指定**には結局JavaScriptが必要でした。

2026年1月13日に、Firefox 147がリリースされ、「CSS Anchor Positioning」が全ブラウザ対応しました。CSS Anchor Positioningとは、要素の位置を別の要素の位置に合わせられるCSSの機能です。Chrome・Safari・Edgeでは先に対応していましたが、長らくFirefoxだけ対応していなかったのです。

本記事では、Popover APIとCSS Anchor Positioningを組み合わせて、ポップオーバーをJavaScriptなしで実装する方法を解説します。

実例として、タスク管理のサブメニューを右側に表示する表現や、ヘッダーのユーザーアイコン下にドロップダウンメニューを表示するデモを紹介します。

![タスク管理のサブメニューが右側に表示されている様子](/images/anchor-positioning-popover/demo2-submenu-task.png)
*タスク管理のサブメニュー*

![ユーザーメニューがアイコンの下に表示されている様子](/images/anchor-positioning-popover/demo3-user-menu.png)
*ユーザーアイコンの下にドロップダウンメニューが表示される*

# Popover APIとは

従来、ポップオーバーを実装するには、大量のイベントリスナー、複雑な位置計算のロジック、スクロールやリサイズへの対応が必要でした。

ポップオーバーの実装例です。

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

ボタンをクリックするとポップオーバーが開くシンプルな例です。

```html
<button popovertarget="my-popover">開く</button>

<div id="my-popover" popover>
  <p>ポップオーバーの内容</p>
</div>
```

これだけで、以下の機能が自動的に実装されます。JavaScriptは不要です。

- クリックで開閉
- ESCキーで閉じる
- 背景クリックで閉じる
- 適切なフォーカス管理

`popovertarget`属性で開閉ボタンを指定し、`popover`属性で要素をポップオーバーとして宣言します。

| 属性 | 役割 |
|------|------|
| `popover` | 要素をポップオーバーとして宣言 |
| `popovertarget` | クリック時に開閉するポップオーバーのIDを指定 |
| `popovertargetaction` | 動作を指定（`toggle` / `show` / `hide`） |

なお、`popovertarget`属性はHTML仕様上、`<button>`要素と`<input type="button">`でのみ使用できます。`<a>`要素では動作しません。

https://developer.mozilla.org/ja/docs/Web/API/Popover_API

# Popover APIがあっても、結局位置計算にはJavaScriptが必要

Popover APIは便利な機能ですが、**ポップオーバーの位置を指定する機能はありません**。ボタンの下や右側にポップオーバーを表示したい場合、従来はJavaScriptで位置を計算する必要がありました。

次の例では、ボタンの位置を取得して、ポップオーバーの位置を計算し、CSSで位置を指定しています。リサイズやスクロールのたびに再計算が必要なので、処理負荷も大きくなってしまいます。

```javascript
// ボタンの位置を取得して、ポップオーバーの位置を計算
const rect = button.getBoundingClientRect();
popover.style.top = `${rect.bottom + 8}px`;
popover.style.left = `${rect.left}px`;
```

# 全ブラウザ対応したCSS Anchor Positioningで位置指定をする

今回全ブラウザで対応したCSS Anchor Positioningを使うと、**CSSだけでポップオーバーの位置を指定できます**。

次のHTMLを例に解説します。

```html
<button class="button" popovertarget="my-popover">開く</button>
<div id="my-popover" popover class="popover">
  <p>ポップオーバーの内容</p>
</div>
```

次の3ステップで使用します。

**ステップ1**: ポップオーバーの基準となる要素を指定する

ポップオーバーの基準となる要素を、CSSの`anchor-name`で定義します。アンカー（anchor）とは船のいかりのことで、そこを基準にして位置を指定します。ちなみにHTMLの`<a>`要素も「アンカー」です。`anchor-name`には任意の名前を指定できます。

```css
.button {
  anchor-name: --my-anchor;
}
```

**ステップ2**: ポップオーバーをアンカー要素と紐づける

ポップオーバーをアンカー要素と紐づけるには、`position-anchor`プロパティを使用します。これで、CSSではアンカー要素を基準とする位置を指定できるようになります。

```css
.popover {
  position-anchor: --my-anchor;
}
```

**ステップ3**: ポップオーバーの位置を指定する

位置の指定方法は2種類あります。

### anchor()関数

`anchor()`関数を使うと、アンカー要素の特定の位置を基準にした配置ができます。

```css
.popover {
  top: anchor(bottom);   /* アンカーの下端 */
  left: anchor(left);    /* アンカーの左端 */
}
```

また、`calc()`と組み合わせて余白を調整することもできます。

```css
.popover {
  position-anchor: --my-anchor;
  top: calc(anchor(bottom) + 8px);
  left: anchor(center);
}
```

| 値 | 説明 |
|---|---|
| `top` | アンカーの上端 |
| `bottom` | アンカーの下端 |
| `left` | アンカーの左端 |
| `right` | アンカーの右端 |
| `center` | アンカーの中央 |

https://developer.mozilla.org/ja/docs/Web/CSS/Reference/Values/anchor

### position-areaプロパティ

`position-area`プロパティを使うと、アンカー周囲のエリアを指定して配置できます。

```css
.popover {
  position-area: block-end;
}
```

| 値 | 配置位置 |
|---|---|
| `block-start` | 上 |
| `block-end` | 下 |
| `inline-start` | 書字方向の開始位置（LTRなら左） |
| `inline-end` | 書字方向の終了位置（LTRなら右） |

2つの値を組み合わせることで、斜め方向にも配置できます。

| 値 | 配置位置 |
|---|---------|
| `block-start inline-end` | 右上 |
| `block-end inline-start` | 左下 |

![ポップオーバーが上側に表示されている様子](/images/anchor-positioning-popover/demo1-popover-top.png)
**ポップオーバーが上側に表示されている様子**

![ポップオーバーが右側に表示されている様子](/images/anchor-positioning-popover/demo1-popover-right.png)
**ポップオーバーが右側に表示されている様子**

@[codepen](https://codepen.io/tonkotsuboy/pen/yyJgQLY)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/yyJgQLY)

▼ 参考: MDN - position-area
https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/position-area_value

# 実例

## サブメニューナビゲーション

私はanchor positioningが出たなら、絶対にこれをCSSだけで表現したいと思っていました。メニューのサブメニューの位置をAnchor Positioningで実装します。

![タスク管理のサブメニューが右側に表示されている様子](/images/anchor-positioning-popover/demo2-submenu-task.png)
*タスク管理のサブメニュー*

![設定のサブメニューが右側に表示されている様子](/images/anchor-positioning-popover/demo2-submenu-settings.png)
*設定のサブメニュー*

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

前述のとおり、`popovertarget`属性は`<button>`要素で使用します（`<a>`要素では動作しません）。

```css
/* メニューリンクをアンカーとして定義 */
#task-menu-anchor {
  anchor-name: --task-menu;
}

/* サブメニューの位置指定 */
.submenu {
  position-anchor: --task-menu;
  top: anchor(top);
  left: anchor(right);
}
```

@[codepen](https://codepen.io/tonkotsuboy/pen/qENRQBP)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/qENRQBP)

## ヘッダーのユーザーメニュー

このパターンもよく使いますね。ヘッダーのユーザーアイコン下に、ドロップダウンメニューを配置する例です。

![ユーザーメニューがアイコンの下に表示されている様子](/images/anchor-positioning-popover/demo3-user-menu.png)
*ユーザーアイコンの下にドロップダウンメニューが表示される*

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
  top: anchor(bottom);
  right: anchor(right);
  margin-top: 8px;
}
```

`right: anchor(right)`を指定することで、ドロップダウンの**右端**とアイコンの**右端**が揃います。ヘッダー右端のアイコンからドロップダウンを表示するとき、メニューが画面外へはみ出さないための指定です。

なお、`position-area: block-end inline-end`だと「アンカーの右下エリアに配置」となり、ドロップダウンの左端がアンカー付近に来るため、右側にはみ出してしまいます。右端を揃えたい場合は`anchor()`関数を使いましょう。

@[codepen](https://codepen.io/tonkotsuboy/pen/myERQdG)

- [別ウインドウで開く](https://codepen.io/tonkotsuboy/pen/myERQdG)

# ブラウザ対応状況

CSS Anchor Positioningは、2026年1月13日に全主要ブラウザで対応しました。

| ブラウザ | 対応バージョン |
|---------|---------------|
| Chrome | 125以降 |
| Edge | 125以降 |
| Safari | 26.0以降 |
| Firefox | 147以降 🎉 |

▼ Can I use - CSS Anchor Positioning
https://caniuse.com/css-anchor-positioning

# まとめ

Popover APIが登場したとき、ポップオーバーがJSなしで実装できて感動しました。しかし位置指定にはCSS Anchor Positioningが必要で、Firefoxが対応するまで実務では使いづらい状況が続いていました。

Safari 26、そしてFirefox 147で全ブラウザ対応したことで、ようやくHTML・CSSだけでポップオーバーを完結できるようになりました。サブメニューやドロップダウンメニューは頻出するUIなので、すぐに使える場面は多いはずです。

# 参考リンク

- [MDN - Popover API](https://developer.mozilla.org/ja/docs/Web/API/Popover_API)
- [MDN - CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning)
- [MDN - Using CSS Anchor Positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Anchor_positioning/Using)
- [Can I use - CSS Anchor Positioning](https://caniuse.com/css-anchor-positioning)
