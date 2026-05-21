---
title: "GoogleのModern Web Guidanceに学ぶ、モダンCSSのDos / Don'ts大全"
emoji: "🧭"
type: "tech"
topics: ["css", "frontend", "webdev"]
published: true
publication_name: ubie_dev
---


![](/images/modern-web-guidance/css-dosdonts.png)

Googleがリリースした「Modern Web Guidance」スキルを使うと、AIエージェントが最新のWeb機能でコードを書けるようになります。

https://zenn.dev/ubie_dev/articles/modern-web-guidance

スキルが参照するガイドラインはMarkdownで書かれており、Googleが推奨するWeb開発のDos / Don'ts、つまり「やるべきこと・やってはいけないこと」がまとまっています。Googleお墨付きの、モダンなフロントエンド開発の知見が詰まっています。AIエージェントを使う・使わないにかかわらず、ウェブ技術の勉強に使ったり、実装に迷ったときに「Googleはどう言っているのかな？」の判断基準にしたりできます。

たとえばCSSのレイアウトのドキュメントは次のURLで確認できます。

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md

本記事は、Google Chromeチームが公開するModern Web Guidance（Apache License 2.0）の各ガイドをもとに、次の方針でまとめました。

- 複数のガイドに散らばったCSSの指針を横断し、現時点で全ブラウザ対応した機能だけを抽出
- Dos / Don'tsがはっきり示されている項目に絞って再構成
- 原文の訳に加えて、初心者向けの補足・最新のブラウザ対応状況・関連する筆者の記事や登壇資料を添えた

## 基礎方針（Foundations）

### 重複を避け、変数より組み込みの仕組みを優先する

同じ値をCSSのあちこちに書くと変更時に直し漏れが起きます。まず変数（カスタムプロパティ）で重複を避けます。さらにブラウザ標準の仕組みで表現できるなら変数より優先します。

■ やるべきこと（Dos）

- 色をそろえたいときは変数ではなく`currentColor`を使う
- 親子で同じ値を使いたいときは変数ではなく`inherit`キーワードを使う
- `font-size: var(--size)`の繰り返しではなく`em`単位を使う
- ボックスモデルの値の繰り返しではなくコンテナ単位の`cqw` / `cqh`（論理版は`cqi` / `cqb`）を使う

https://caniuse.com/css-variables

### 物理プロパティより論理プロパティを優先する

書字方向が変わっても破綻しないよう、物理プロパティより論理プロパティを優先します。論理プロパティは書字方向に応じて向きが変わるプロパティです。RTL（右横書き）だけでなく、日本語の縦書き（`writing-mode: vertical-rl`）でも向きが自動で切り替わります。横書きのみのサイトでは見た目は変わりませんが、外部の翻訳ツールが訳文を差し込む場合などの保険になります。

■ やるべきこと（Dos）

- `margin-left`ではなく`margin-inline-start`のように論理プロパティを使う

```css
/* RTL では自動的に右側の余白として扱われる */
.item {
  margin-inline-start: 16px;
}
```

■ やってはいけないこと（Don'ts）

- 論理プロパティを無条件に使わない。「これはRTLで反転してほしいか」と自問し、答えが「いいえ」なら物理プロパティを使う。たとえば言語に関係なく常に同じ向きに出したい影やアイコンの位置などは、物理プロパティのままにする

https://caniuse.com/css-logical-props

> 原典: [Modern Web Guidance / CSS — 1. Foundations](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#1-foundations)

## レイアウトの基礎（Fundamentals）

レイアウトはブラウザのレイアウトエンジンに任せるほど速くなります。固定の幅や高さ、込み入ったメディアクエリに頼る前に、intrinsic sizing・論理プロパティ・`aspect-ratio`を先に検討します。intrinsic sizingとは、要素の中身に応じてサイズを決める仕組みです。

### どのレイアウトを使うかの判断基準

どのレイアウト（FlexboxやGrid等）を使うかは、上から順に見て最初に当てはまったもので決めます。

|用途|使うもの|
| --- | --- |
|要素を1方向（行か列）に並べるだけ| Flexbox（1次元・コンテンツ主導）|
|入れ子の要素を親グリッドのトラックに揃えたい| subgrid（2次元・親トラックを継承）|
|行と列の両方を持つ複雑なページ・コンポーネント構造| Grid（2次元・骨格を先に定義）|
|長い文章を新聞のような段組みに分けたい| multi-column（1次元のフロー）|
|高さがバラバラな要素を隙間なく詰めたい| `grid-auto-flow: dense`のGridで代用する|
|要素をページの上に浮かせ、DOMやスタッキングコンテキスト（重なり順を決めるまとまり）を越えてトリガーに追従させたい| anchor positioning（トリガーに`anchor-name`、オーバーレイに`position-anchor`）|

### 固定値よりintrinsic sizingと論理プロパティを使う

サイズや余白は、できるだけ固定の`width`/`height`ではなくintrinsic sizingと伸縮するトラックで指定します。メディアクエリが減り、壊れにくいレイアウトになります。

■ やるべきこと（Dos）

- レイアウトの寸法や余白には論理プロパティ（`inline-size`・`block-size`・`margin-inline`・`padding-block`・`inset-inline-start`）を使う
- コンテンツ主導ならFlexbox、骨格を先に決めるならGridという考え方で選ぶ
- 両軸の揃えは`place-content`・`place-items`・`place-self`の一括指定でまとめる
- 固定値の前にintrinsic sizing（`min-content`・`max-content`・`fit-content()`）と伸縮するトラック（`fr`・`minmax()`）を使う
- メディア要素には`aspect-ratio`で先に場所を確保し、読み込み前のレイアウトシフトを防ぐ

```css
.sidebar      { inline-size: max-content; }    /* 折り返せない最長の語に合わせる */
.main-content { inline-size: fit-content; }    /* 使える幅まで伸び、それ以上は伸びない */
.media        { aspect-ratio: 16 / 9; inline-size: 100%; block-size: auto; }
body.centered { display: grid; place-content: center; min-block-size: 100dvb; }
```

https://caniuse.com/intrinsic-width

> 原典: [Modern Web Guidance / CSS Layout — 1. Fundamentals](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#1-fundamentals)

## Flexbox

Flexboxは1次元のレイアウトです。要素は主軸（main）に沿って流れ、交差軸（cross）で揃えます。ナビゲーション・ツールバー・項目の横並びなど、1方向の並びに向きます。

### 1方向の並びはFlexboxを使う

`display: flex`で文脈を作り、`flex-direction`（デフォルトは`row`）で主軸を決めます。

■ やるべきこと（Dos）

- 折り返しの可能性があれば`flex-wrap: wrap`を付ける。`nowrap`のまま`overflow: auto`/`hidden`もないと、狭い画面ではみ出す
- 子要素の伸縮は`flex-grow`/`flex-shrink`/`flex-basis`を個別に書かず、`flex`一括指定（例: `flex: 1 1 250px`）で書く
- 要素間の余白は子の`margin`ではなく`gap`（または`row-gap`/`column-gap`）で取る
- 位置揃えのキーワードには`safe`を付ける（例: `align-items: safe center`）。コンテナが中身より狭くてもフォーカス対象が見切れない
- 1つの要素だけ主軸の端に寄せたいときは`margin-inline-start: auto`（または`margin-block-start: auto`）を使う。これが定石
- 要素ごとの交差軸の揃えは`align-self`で上書きする
- 全要素を交差軸で中央寄せするなら`align-items`。1つの要素を両軸で中央に置くなら`margin: auto`。`align-content`は折り返して行に余りがあるときだけ使う
- URLやコードなど折り返せない長い中身を持つflex要素には`min-inline-size: 0`（または`min-width: 0`）を付ける。flex要素はデフォルトで中身より小さくならず、はみ出すため

```css
.card-grid        { display: flex; flex-flow: row wrap; gap: 1rem; }
.card-item        { flex: 1 1 250px; }                  /* 伸び・縮み・基準サイズ */
.card-item-action { margin-inline-start: auto; }        /* 主軸の端へ寄せる */
.toolbar          { display: flex; align-items: safe center; }
```

■ やってはいけないこと（Don'ts）

- flex要素に`justify-self`を使わない。Grid・block・絶対配置でしか効かない。代わりにautoマージンを使う
- 操作できるコンテンツの並べ替えに`order`や`flex-direction: *-reverse`を使わない。見た目の順序だけが変わり、DOM順は変わらない。キーボードのフォーカス順が見た目と食い違う
- `space-around`（両端は半分の余白）と`space-evenly`（前・間・後ろが等間隔）を混同しない
- 軸の入れ替わりを忘れない。`flex-direction: column`のとき、`justify-content`はブロック軸、`align-items`はインライン軸を揃える。デフォルトとは逆
- コンテナと子を互いに埋め合うサイズにしない。はみ出しや予想外の結果につながる。どちらか一方に確定したサイズを与える
- 同じ要素に`flex-basis`と`width`/`inline-size`を両方指定しない。flex文脈では`flex-basis`が優先され`width`は無視される

```css
/* どちらも避ける書き方 */
.flex-item {
  justify-self: end; /* flex では効かない。auto マージンを使う */
  flex-basis: 200px;
  width: 300px;      /* flex-basis が優先され、これは無視される */
}
```

https://caniuse.com/flexbox

> 原典: [Modern Web Guidance / CSS Layout — 2. Flexbox](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#2-flexbox)

https://speakerdeck.com/tonkotsuboy_com/css-gridflexboxno-zui-jin-nojin-hua-tomirai

## グリッドとサブグリッド（Grid and subgrid）

Gridは2次元のレイアウトです。行と列を明示的に定義するか、ブラウザに推測させます。subgridは、入れ子のグリッドが親のグリッド線をそのまま使える機能です。これで子孫要素を兄弟どうしで揃えられます。

### 行と列を持つ構造は Grid を使う

■ 列数が決まっているか

- 決まっている → 明示的なトラック（`grid-template-columns: 200px 1fr`、`repeat(3, 1fr)`など）を使う
  - 列ごとにサイズが違う（サイドバー＋メイン、全幅のヘッダーなど） → `grid-template-areas`で名前付きの読みやすい領域にする
  - 全列が同じ、または線番号だけで配置する → `repeat(N, ...)`や名前付き線を使う
- 決まっていない（レスポンシブ、項目数不明） → `repeat(auto-fit, minmax(min, 1fr))`を使う
  - 最終行の項目を伸ばして余白を埋めたい → `auto-fit`
  - 空の最終行トラックを最小サイズで残したい → `auto-fill`

■ 特定の場所に置くか

- 置く → `grid-column: <start> / <end>`か`grid-area: <name>`
- ただ複数トラックにまたがるだけ（位置は問わない） → `grid-column: span <n>`

■ 子が親のトラックサイズを継承するか（兄弟どうしで端を揃えるか）

- する → 該当する軸にsubgridを使う
  - セルごとの子の数が可変 → 片方の軸だけsubgrid。もう一方は`grid-auto-rows`/`grid-auto-columns`
  - 子の数が固定 → 両軸のsubgridでよい
- しない → 通常のGridでよい

■ やるべきこと（Dos）

- ページ全体の複雑なレイアウトには`grid-template-areas`を使う。領域名がそのまま説明になり、宣言を行と列に揃えて書くと一目で構造が分かる
- レスポンシブなカードグリッドには`repeat(auto-fit, minmax(200px, 1fr))`。空トラックを最小サイズで残すなら`auto-fill`
- 比率で分配するなら`fr`、伸縮するが上限のあるトラックには`minmax(min, max)`
- 配置は`grid-column: span <n>`（トラックをまたぐ）・`grid-column: <start> / <end>`（特定の線に置く）・`grid-area: <name>`（名前付き領域）で指定する
- カードリストの「端がそろわない」問題はsubgrid（`grid-template-columns: subgrid`か`grid-template-rows: subgrid`）で解決する。たとえばブログカードのタイトル・メタ情報・CTAが兄弟どうしで揃う
- subgrid宣言の直前に`grid-template-rows`/`-columns`の明示宣言を置く。同じカスケード内で古いブラウザのフォールバックになる


■ やってはいけないこと（Don'ts）

- `auto-fit`/`auto-fill`のトラックサイズが項目の中身から決まると思わない。`repeat()`のサイズ引数から決まる
- 操作できるコンテンツに`grid-auto-flow: dense`を使わない。詰め込みは効率的だが見た目の順序が変わり、DOM順のキーボードフォーカスが崩れる
- 子の数が可変のとき両軸にsubgridを使わない。余りは最終トラックに入る。暗黙の軸は`grid-auto-rows`/`grid-auto-columns`に任せる
- `justify-items`/`align-items`（トラック内で項目の中身を揃える）と`justify-content`/`align-content`（コンテナ内でトラックを揃える）を混同しない。取り違えても何も起きず、原因に気づきにくい
- コンテナに確定した`inline-size`がないまま`repeat(auto-fit/auto-fill, ...)`を使わない。`display: inline-grid`やサイズ未指定のflex要素の中では分割する幅がなく、トラック数が安定しない


```css
/* inline-grid では分割する幅がなく、トラック数が安定しない */
.grid {
  display: inline-grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
```


https://caniuse.com/css-grid

https://caniuse.com/css-subgrid

> 原典: [Modern Web Guidance / CSS Layout — 3. Grid and subgrid](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#3-grid-and-subgrid)

https://zenn.dev/tonkotsuboy_com/articles/css-subgrid-all-browsers


## コンテナクエリ（Container queries）

コンテナクエリは、画面幅ではなく親要素（コンテナ）の幅に応じてスタイルを変える仕組みです。コンテナクエリはコンポーネントの文脈、メディアクエリはページ全体のレイアウトやユーザー設定（`prefers-color-scheme`・`prefers-reduced-motion`）に使う、と覚えます。

### 親要素の幅に応じた切り替えはコンテナクエリを使う

子孫をクエリする前に、ラッパーに抑制（containment。中身のサイズ計算を外と切り離す指定）の文脈を作ります。幅だけなら`container-type: inline-size`、両軸なら`container-type: size`です。

■ やるべきこと（Dos）

- ラッパーに`container-type: inline-size`（幅のみ）か`container-type: size`（両軸）を付けて抑制の文脈を作る
- 入れ子の文脈が衝突しそうなら`container-name`（または一括指定`container: inline-size card`）で名前を付ける
- 流動的なフォントサイズや余白の計算にはコンテナクエリ単位を使う。`cqi`/`cqb`（論理のインライン/ブロック）・`cqw`/`cqh`（物理）・`cqmin`/`cqmax`
- `container-type: size`を使うときはコンテナに確定した`block-size`を与える。ないと抑制により中身が無視され、子孫がつぶれる

```css
.card-wrapper {
  container: inline-size / card; /* container-type + container-name の一括指定 */
}

@container card (inline-size > 400px) {
  .content {
    display: flex;
    gap: 2rem;
  }
}

.title {
  /* ビューポートでなくコンテナ幅に連動した流動的なフォントサイズ */
  font-size: clamp(1rem, 4cqi, 2rem);
}
```

■ やってはいけないこと（Don'ts）

- `container-type`の値に`block-size`を使わない。無効。両軸は`size`

```css
/* block-size は container-type の値として無効 */
.wrapper {
  container-type: block-size;
}
```

- `container-type`を宣言した後で子のintrinsic sizeがコンテナに影響すると思わない。抑制が効くと、コンテナは子要素が無いものとして扱われる
- 条件を満たさない祖先の子孫でコンテナクエリ単位に頼らない。小ビューポート（`svw`/`svh`）にフォールバックする


https://caniuse.com/css-container-queries

> 原典: [Modern Web Guidance / CSS Layout — 4. Container queries](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#4-container-queries)

https://zenn.dev/tonkotsuboy_com/articles/css-container-query

## ネイティブのオーバーレイとアンカーポジショニング（Native overlays & anchor positioning）

オーバーレイの基本要素は用途で使い分けます。`<dialog>`・popover・anchor positioningはいずれも主要ブラウザで使えるようになりました。

### オーバーレイは用途で使い分ける

■ やるべきこと（Dos）

- 一時的で非モーダルなUI（ポップアップ・トースト・ツールチップ）には`popover`を使う。トップレイヤー（ページ最前面の特別な層）に乗るため`z-index`の管理が要らない
- フォーカストラップと操作不可の背景が必要なモーダルには`<dialog>`を`.showModal()`で開く

■ やってはいけないこと（Don'ts）

- 同じ要素に`popover`と`.showModal()`を併用しない。実行時には排他的で、両立しない

https://caniuse.com/dialog

https://caniuse.com/mdn-html_global_attributes_popover

### アンカーポジショニングは機能検出してフォールバックを用意する

anchor positioningは、トリガー要素にオーバーレイを空間的に紐付けて配置する機能です。DOMやスタッキングコンテキストを越えても追従します。主要ブラウザで使えるようになりました。古いバージョンも考慮するなら、機能検出とフォールバックを用意しておくと安全です。

■ やるべきこと（Dos）

- 配置とサイズは`position-area`（またはインセットに`anchor()`）と`anchor-size()`でトリガー基準に指定する
- オーバーレイがビューポートをはみ出すときは`position-try-fallbacks: flip-block`（または`flip-inline`）でブラウザに再配置させる
- `@supports (anchor-name: --x)`で機能検出し、絶対配置のフォールバックを用意する

■ やってはいけないこと（Don'ts）

- 1つの`position-area`の値に物理キーワードと論理キーワードを混ぜない。座標系はどちらか一方に統一する


https://caniuse.com/css-anchor-positioning

> 原典: [Modern Web Guidance / CSS Layout — 5. Native overlays, anchor positioning & stacking contexts](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#5-native-overlays-anchor-positioning-and-stacking-contexts)

https://zenn.dev/ubie_dev/articles/anchor-positioning-popover


## オーバーフローとレイアウトの安定（Overflow tracking and layout stability）

スクロール、はみ出し、切り抜きを予測しやすく扱います。

### スクロールバーは`overflow: auto`で必要なときだけ出す

■ やるべきこと（Dos）

- スクロールバーは、中身が実際にはみ出したときだけ出す。`overflow: auto`を使う

```css
.box {
  overflow: auto;
}
```

■ やってはいけないこと（Don'ts）

- `overflow: scroll`を避ける。スクロールするものが無くてもスクロールバーを強制表示する

```css
.box {
  overflow: scroll;
}
```

https://caniuse.com/css-overflow

### 切り抜きだけしたいなら`overflow: clip`を使う

■ やるべきこと（Dos）

- スクロールコンテナを作らずに中身を切り抜くだけなら`overflow: clip`。あえてはみ出させたい分は`overflow-clip-margin`で許可する

■ やってはいけないこと（Don'ts）

- 切り抜きたいだけのときに`overflow: hidden`を使わない。`hidden`はスクロールコンテナを作り、プログラムからスクロールできてしまう

```css
/* 切り抜きたいだけなら clip。hidden はスクロールコンテナを作る */
.box {
  overflow: hidden;
}
```

https://caniuse.com/css-overflow

### `scrollbar-gutter: stable`でスクロールバーの領域を確保する

■ やるべきこと（Dos）

- 中身が増えたときのレイアウトシフトを防ぐため、`scrollbar-gutter: stable`でスクロールバーの領域をあらかじめ確保する。スクロールバーが出た瞬間に中身の幅が変わるのを防げる

https://caniuse.com/mdn-css_properties_scrollbar-gutter

### スクロールの連鎖を`overscroll-behavior`で止める

■ やるべきこと（Dos）

- スクロール可能なコンテナでは`overscroll-behavior: contain`（または`none`）を使う。コンテナの端までスクロールしたとき、その先のスクロールが親やページ本体へ伝わるのを止められる

https://speakerdeck.com/tonkotsuboy_com/2022nian-falsemodancssgai

https://caniuse.com/css-overscroll-behavior

### 複数行の省略は`-webkit-line-clamp`の三点セットを使う

■ やるべきこと（Dos）

- 複数行のテキストを「…」で省略するには`-webkit-line-clamp`・`display: -webkit-box`・`-webkit-box-orient: vertical`の3つを組み合わせる。`-webkit-`接頭辞が付くが、この書き方は仕様で正式に定義済みで非推奨ではない
- 三点セットと並べて`line-clamp`ショートハンドも書いておく。未対応のブラウザは無視するだけで、将来対応したブラウザでそのまま効く

```css
.scrollable-list {
  max-block-size: 400px;
  overflow-y: auto;
  scrollbar-gutter: stable; /* スクロールバーの領域を確保する */
  overscroll-behavior: contain; /* ページ本体へスクロールを連鎖させない */
}

.snippet {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  line-clamp: 3; /* 未対応のブラウザでは無視される */
  overflow: clip;
}
```

https://caniuse.com/css-line-clamp

> 原典: [Modern Web Guidance / CSS Layout — 6. Overflow tracking and layout stability](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#6-overflow-tracking-and-layout-stability)

## ビューポートの扱い（Viewport mechanics）

### モバイルの全画面コンテナには`dvh`・`dvw`を使う

■ やるべきこと（Dos）

- ブラウザUIの伸縮に追従させたいコンテナには`dvh`や`dvw`を使う。スマホではスクロールに応じてアドレスバーが伸縮し、その分だけ表示領域の高さが変わる。`dvh`は、ブラウザUIの伸縮を反映した「動的な」ビューポートの高さの単位

https://caniuse.com/viewport-unit-variants

### 全幅レイアウトに`100vw`を使わない

■ やるべきこと（Dos）

- 全幅のレイアウトには`100%`・`100dvw`・`100svw`を使う

```css
.full-width {
  width: 100%; /* もしくは 100dvw / 100svw */
}
```

■ やってはいけないこと（Don'ts）

- 全幅のレイアウトに`100vw`を使わない。`100vw`はスクロールバーの幅を含まず、横方向のはみ出しが起きる

```css
.full-width {
  width: 100vw;
}
```

https://caniuse.com/viewport-unit-variants

> 原典: [Modern Web Guidance / CSS Layout — 7. Viewport mechanics and track distribution](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css-layout/css-layout.md#7-viewport-mechanics-and-track-distribution)

## 継承とカスケード

カスケード（cascade）は、複数のスタイルが競合したときにどれを適用するかを決める仕組みです。継承（inheritance）は、親要素の値が子要素に引き継がれる仕組みです。

### 詳細度はカスケードレイヤーと`:where()`で管理する

■ やるべきこと（Dos）

- 詳細度の管理にはカスケードレイヤー（`@layer`）や`:where()`を使う。カスケードの挙動が予測しやすくなり、書き手の意図どおりに振る舞う

■ やってはいけないこと（Don'ts）

- 詳細度を管理するためにBEMなどの命名規則を導入しない

https://caniuse.com/css-cascade-layers

### カスケードレイヤーで優先順位を明示する

`@layer`で優先順位のゾーン（`reset`・`base`・`theme`・`components`・`utilities`など）を定義します。順序は先頭でまとめて宣言します。

■ やるべきこと（Dos）

- レイヤー順を先頭でまとめて宣言する

```css
/* 後に書いたレイヤーほど優先される */
@layer reset, base, theme, components, utilities;
```

- 各レイヤーの内部では`:where()`を使う。`:where()`は詳細度を0にして包む機能で、意味のあるシグナルだけで競合し、簡単に上書きできるデフォルト値を作れる

https://caniuse.com/css-cascade-layers

### 明示的な値ではなくグローバルキーワードで意図を表す

具体的な値ではなく`inherit`・`initial`・`unset`・`revert`を使うと意図が明確になります。

■ やるべきこと（Dos）

- 親と同じトランジションを子に持たせたいときは、書き直さず`transition: inherit`を使う
- プロパティを初期値に戻したいときは値を書かず`initial`を使う

```css
.child {
  /* 親と同じトランジションをそのまま受け継ぐ */
  transition: inherit;
}
```

> 原典: [Modern Web Guidance / CSS — 2. Inheritance and the cascade](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#2-inheritance-and-the-cascade)

## セレクタとスコープ（Selectors and scoping）

ブラウザ標準のモダンなセレクタを使えば、プリプロセッサやJavaScriptによる複雑な状態管理を減らせます。

### 子の状態で親をスタイルするには`:has()`を使う

子要素の状態に応じた親のスタイルは、JavaScriptでクラスを付け外しせずCSSだけで表現できます。`label.has-checked`のようなクラスを手で管理する代わりに`label:has(:checked)`と書けます。

■ やるべきこと（Dos）

- 子の状態で親を狙うときは`:has()`を使う。状態管理がCSSに寄り、JavaScriptが減って表示と状態のズレも起きない

```css
/* チェックされた子を持つ label を狙う */
label:has(:checked) {
  background: var(--color-accent);
}
```

■ やってはいけないこと（Don'ts）

- 親の見た目を変えるためにJavaScriptでクラスを付け外ししない

```js
// チェック状態が変わるたびにクラスを付け外しする
checkbox.addEventListener("change", () => {
  label.classList.toggle("has-checked", checkbox.checked);
});
```

- `:has()`を入れ子にしたり、内部で疑似要素を使ったりしない（ブラウザAPIの制限）

https://speakerdeck.com/tonkotsuboy_com/2023nian-modancssnozui-xin-torendo

https://caniuse.com/css-has

### n番目の特定要素を狙うには`:nth-child(... of ...)`を使う

ある条件を満たす要素のうちn番目をスタイルしたいときは`:nth-child(<An+B> of <selector>)`を使います。

■ やるべきこと（Dos）

- 条件付きのn番目には`:nth-child(... of ...)`を使う

```css
/* 見つかった中で最初の「開いている」details を狙う */
details:nth-child(1 of [open]) {
  outline: 2px solid var(--color-accent);
}
```

■ やってはいけないこと（Don'ts）

- 同じ意図で`details[open]:first-child`と書かない。これは「最初の子であり、かつそれが開いている場合」にだけ当たり、意図が変わる

```css
/* 最初の子が開いているときだけ当たる。意図がずれる */
details[open]:first-child {
  outline: 2px solid var(--color-accent);
}
```

https://caniuse.com/css-nth-child-of

### フォールバックにはルール複製ではなく`:is()`/`:where()`を使う

サポートされていないかもしれない疑似クラスのフォールバックは、CSSルールを複製せず`:is()`や`:where()`の寛容なパース（forgiving parsing。対応していないセレクタが交じっても全体を無効にしない解釈）で1つにまとめます。

■ やるべきこと（Dos）

- フォールバックは`:where()`（または`:is()`）で1つのルールにまとめる。これらは未対応のセレクタが交じってもルール全体を無効にしないため、複製せずに済む

```css
[popover]:where(:popover-open, .\:popover-open) {
  /* 同じスタイルを1つのルールで */
}
```

`.\:popover-open`は、ポリフィルが付けるコロン入りのクラス名`:popover-open`を狙うセレクタです。クラス名に含まれるコロンは、疑似クラスと区別するため`\`でエスケープします。

■ やってはいけないこと（Don'ts）

- 疑似クラスのフォールバックのためにCSSルールを複製しない

```css
/* `:where()` を使わずにルールを重複させている */
[popover]:popover-open {
  /* ネイティブ popover 向けのスタイル */
}
[popover].\:popover-open {
  /* ポリフィル版のために同じスタイルをもう一度 */
}
```

- 疑似要素にこの手法を使わない。`:is()` / `:where()`は疑似要素に対応していない

https://caniuse.com/css-matches-pseudo

### 無関係な状態・対象の除外には上書きではなく`:not()`を使う

本質的に無関係な状態や要素を除外したいときは、後から上書きするのではなく`:not()`で最初から除外します。

■ やるべきこと（Dos）

- 「最後ではない`li`にだけ下線を引く」のように、除外したい対象は`:not()`で最初から外す

```css
.fancy-list li:not(:last-child) {
  border-bottom: 1px solid silver;
}
```

- 並べ替えても結果が変わらないよう`button:hover:not(:disabled)`と書く

```css
button:hover:not(:disabled) {
  background: var(--color-blue);
}

button:disabled {
  background: var(--color-neutral);
}
```

■ やってはいけないこと（Don'ts）

- 望ましい値を後から打ち消す書き方をしない。別ルールで設定した`border-bottom`まで意図せず消すおそれがある

```css
.fancy-list li {
  border-bottom: 1px solid silver;
}

.fancy-list li:last-child {
  border-bottom: none;
}
```

- `button:hover`と`button:disabled`を別々に書かない。並べ替えると無効化されたボタンにもhoverの背景色が当たる

```css
button:hover {
  background: var(--color-blue);
}

button:disabled {
  background: var(--color-neutral);
}
```

https://caniuse.com/css-not-sel-list

### 特化のための上書きは問題ない

いっぽうで、対象を特化するための上書きは問題ありません。「ボタンは基本ニュートラル、主要ボタンだけ青」のように、どちらのルールも正当な意図を表しています。

■ やるべきこと（Dos）

- 特化（specialization）のための上書きは使ってよい

```css
button {
  background: var(--color-neutral);
}

button.primary {
  background: var(--color-blue);
}
```

### 深くネストしたサブツリーの除外には`:not()`より`@scope`を使う

`:not()`と子孫セレクタの組み合わせでもサブツリー（ある要素より下の枝。子孫のまとまり）を除外できます。ただし深くネストした構造ではうまく機能しません。`@scope`は階層上の近さを考慮するため、この問題を解決します。

■ やるべきこと（Dos）

- 深くネストしたサブツリーの除外には`@scope`で範囲を区切る

```css
@scope (.card) to (.content) {
  /* .card の中にあり、かつ .content の中にはない要素向けのスタイル */
}
```

これは入れ子のカードでも期待どおりに動作します。

■ やってはいけないこと（Don'ts）

- 深い入れ子の除外を`:not()`と子孫セレクタだけで済ませない。たとえば`.card :not(.content *)`は入れ子のカードで期待どおりに動かない

```css
/* 入れ子のカードでは期待どおりに動かない */
.card :not(.content *) {
  /* … */
}
```

https://caniuse.com/css-cascade-scope

### ネスト（nesting）はスコープと使い分ける

ネイティブのCSSネストは、関連するスタイルをまとめて読みやすくする範囲で使います。ただし「詳細度」より「近さ」を優先したいときは、ネストより`@scope`が向きます。テーマ用のクラスのように、どんな順でネストされても一番近い対象を勝たせたい場合です。

■ やるべきこと（Dos）

- 可読性と保守性が上がる範囲でネイティブのCSSネストを使う
- 近さを優先したいときはネストより`@scope`を使う

```css
@scope (.dark) {
  .invert { color-scheme: light; }
}

@scope (.light) {
  .invert { color-scheme: dark; }
}
```

■ やってはいけないこと（Don'ts）

- 近さが重要な場面でネスト（や子孫セレクタ）に頼らない。次は`.invert`が`.dark`と`.light`の両方にネストされると、詳細度が同じため常にダークに解決される

```css
.dark .invert { color-scheme: light; }
.light .invert { color-scheme: dark; }
```

https://zenn.dev/tonkotsuboy_com/articles/css-nesting-without-sass

https://caniuse.com/css-nesting

### グローバルリセットは使わない

■ やるべきこと（Dos）

- リセットスタイルは特定の要素種別や条件に対して適用する

■ やってはいけないこと（Don'ts）

- `*`に対するスタイル（グローバルリセット）を使わない。Webコンポーネントや優先順位の低いカスケードレイヤーから（`!important`なしには）上書きできない

```css
/* グローバルリセットは後から上書きしづらい */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

> 原典: [Modern Web Guidance / CSS — 3. Selectors and scoping](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#3-selectors-and-scoping)

## インタラクティブ性（Interactivity）

### フォーカスリングは`:focus`ではなく`:focus-visible`で定義する

`:focus`でフォーカスリングを上書きすると、マウスでクリックしたときにも常にリングが出ます。キーボード操作時だけリングを出したいときは`:focus-visible`を使います。

■ やるべきこと（Dos）

- フォーカスリングは`:focus-visible`で定義する
- リングには`box-shadow`より`outline`を使い、`outline-offset`で要素との間に余白を取る

```css
.button:focus-visible {
  /* リングと要素の間に余白を取る */
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}
```

- `box-shadow`に頼る場合は`forced-colors`メディアクエリで`outline`ベースのフォールバックを用意する。`box-shadow`は強制カラーモードで消えるため

■ やってはいけないこと（Don'ts）

- `:focus`でフォーカスリングを定義しない。マウスでクリックしたときにも常にリングが出る
- `outline: none`でデフォルトのリングを消したまま、代替の可視フォーカスを用意しない。キーボード利用者を締め出す

```css
/* 代替を用意せず既定のフォーカスを消すとキーボード利用者が困る */
.button:focus {
  outline: none;
}
```

https://caniuse.com/css-focus-visible

### タッチターゲットは`min-block-size`/`min-inline-size`で確保する

操作要素は最低24×24 CSSピクセルを確保します（[WCAG 2.5.8 Target Size Minimum（AA）](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum)）。

■ やるべきこと（Dos）

- 大きさは`width`/`height`ではなく`min-block-size` / `min-inline-size`やパディングで確保する。中身で大きくはできても、小さくはできなくなる

```css
.icon-button {
  /* 中身で広がるのは許容し、24px より小さくはしない */
  min-block-size: 24px;
  min-inline-size: 24px;
}
```

- タッチなど粗いポインタ環境では`@media (pointer: coarse)`でターゲットを大きくする

■ やってはいけないこと（Don'ts）

- 固定の`width`/`height`でタッチターゲットを決めない。中身が増えても広がらず、はみ出すおそれがある

```css
/* 中身が増えても広がらず、はみ出すおそれがある */
.icon-button {
  width: 24px;
  height: 24px;
}
```

https://caniuse.com/css-logical-props

### カスタムジェスチャーで`touch-action: none`を使わない

■ やるべきこと（Dos）

- 必要な軸だけに絞る。横スワイプなら`pan-y`（縦スクロールは残る）、縦スワイプなら`pan-x`

```css
.carousel {
  /* 横スワイプを扱いつつ縦のページスクロールは残す */
  touch-action: pan-y;
}
```

- `none`は描画キャンバスのようにネイティブのタッチ挙動が不要な要素だけに使う

■ やってはいけないこと（Don'ts）

- カスタムジェスチャーのために`touch-action: none`を使わない。その要素を通したページのスクロールができなくなる

```css
/* この要素の上ではページがスクロールできなくなる */
.carousel {
  touch-action: none;
}
```

https://caniuse.com/css-touch-action

> 原典: [Modern Web Guidance / CSS — 4. Interactivity](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#4-interactivity)

## デザイントークンとテーマ（Design Tokens and Theming）

### デザイントークンは`:root`のカスタムプロパティで定義する

色・フォント・サイズなどの中核のデザイン変数は`:root`のカスタムプロパティとして定義します。デザイン全体で一貫性を保て、チームをまたいでUIをスケールできます。

■ やるべきこと（Dos）

- 中核のデザイン変数は`:root`のカスタムプロパティで定義する
- トークンは前の層を踏まえた階層で構成する。スコープが小さいほど層は少なくてよく、簡単なデモなら1層で十分
  - 階層1:プリミティブトークン（`--color-blue-10`・`--color-gray-90`・`--font-sans-serif`・`--size-xl`など）。
  - 階層2:セマンティックトークン（`--color-accent`・`--color-neutral`・`--font-body`・`--font-heading`など）。
  - 階層3:汎用UIのトークン（`--ui-border`・`--surface-bg-subtle`など）。
  - 階層4:コンポーネント固有のトークン（`--button-bg-primary-hover`など）。
- 自前の流儀を作る前に既存の命名規則を確認する

■ やってはいけないこと（Don'ts）

- ささいでないスタイリング値をインラインで指定しない。`background: transparent`や`padding: 0`は問題ないが、`background: #f06`や`padding: .3em`はいけない（例外はテストケースなどコードを小さく保つことが最優先の場面だけ）

```css
/* 値を直接書くと変更時に直し漏れる */
.button {
  background: #f06;
  padding: .3em;
}
```

- トークンの階層を過剰に設計しない

https://caniuse.com/css-variables

### ダークモードは`color-scheme: light dark`でシステム設定に追従させる

■ やるべきこと（Dos）

- `:root`に`color-scheme: light dark`を指定してシステム設定に自動追従させる。個別の要素に指定すれば、そのサブツリーだけ別の値を強制できる

```css
:root {
  /* システムのライト/ダーク設定に自動追従する */
  color-scheme: light dark;
}
```

https://caniuse.com/mdn-css_properties_color-scheme

### `light-dark()`で配色を自動解決する

`light-dark()`を使うと、要素の`color-scheme`に応じて第1引数（ライト）か第2引数（ダーク）が選ばれます。通常は階層2か階層3のトークンで使います。

■ やるべきこと（Dos）

- 配色は`light-dark()`で自動解決する

```css
:root {
  /* color-scheme に応じてライトかダークの値が選ばれる */
  --color-surface: light-dark(#ffffff, #1a1a1a);
  --color-text: light-dark(#1a1a1a, #f5f5f5);
}
```

■ やってはいけないこと（Don'ts）

- 継承される`<color>`プロパティで`light-dark()`を使うとき、子孫の`color-scheme`上書きに追従すると思い込まない。その要素の`color-scheme`で具体的な色に解決され、子孫には解決済みの色が継承される。動的に保ちたいときは、未登録のカスタムプロパティとして値を渡すだけにとどめる

https://caniuse.com/mdn-css_types_color_light-dark

### 強制カラーモードにフォールバックを用意する

強制カラーモード（Windowsのハイコントラスト）では、ブラウザが作者の色をシステムキーワードで上書きします。`background-image`・`box-shadow`・`border-image`は取り除かれます。

■ やるべきこと（Dos）

- 色トークンには`@media (forced-colors: active)`でシステムカラーのフォールバックを定義する

```css
@media (forced-colors: active) {
  .button {
    /* システムキーワードへフォールバックする */
    border: 1px solid ButtonText;
    color: ButtonText;
  }
}
```

- 色が情報そのものを担う場面（シンタックスハイライター、カラーピッカーの見本色など）では`forced-color-adjust: none`を使う

■ やってはいけないこと（Don'ts）

- 境界線・区切り・状態を`background-image`・`box-shadow`・`border-image`だけで表現しない。強制カラーモードで消える（印刷でも消えがち）。使うならシステムカラーのキーワード（`CanvasText`・`LinkText`・`ButtonText`・`Highlight`・`GrayText`など）の`outline`や`border`で代替を用意する

```css
/* 強制カラーモードでは影が消え、境界が見えなくなる */
.card {
  box-shadow: 0 0 0 1px gray;
}
```

- 見た目を保ちたいだけの理由で`forced-color-adjust: none`を使わない

https://caniuse.com/mdn-css_at-rules_media_forced-colors

### 濃淡（ティント）は明度チャンネルだけで作らない

色の濃淡を動的に作る前に、まず既存のデザイントークンが使えないか確認します。そのほうがデザイナーの制御が効きます。

■ やるべきこと（Dos）

- 動的に作るなら`color-mix()`で白や黒と混ぜる（できれば`oklab`空間で）。色を安全に色域へ収められる。ただし彩度が落ちて色あせがち

```css
:root {
  /* oklab 空間で白と 30% 混ぜて明るい濃淡を作る */
  --color-primary-tint: color-mix(in oklab, var(--primary), white 30%);
}
```

- 明度調整を他の手法と組み合わせてもよい。ただし明度調整は30％を超えない

■ やってはいけないこと（Don'ts）

- 明度チャンネルだけを単純に調整しない。理論上は正しい方法だが、ブラウザはまだ色域マッピング（色を表示可能な範囲に収める処理）を実装しておらず、結果の色が予測できない

```css
/* 明度チャンネルだけ調整。結果の色が予測できない */
.tint {
  background: oklab(from var(--primary) 0.9 a b);
}
```

https://caniuse.com/mdn-css_types_color_color-mix

### ブラウザ生成UIはまずCSSでテーマを当てる

ブラウザが生成するUIの多くはCSSでカスタマイズできます。モダンな機能でも、古いブラウザで大きく崩れずに表示されます。フォールバックが要らないことも多いです。作り直す前に2点を確認します。

1.モダンなCSSでも必要なだけカスタマイズできないか
2.作り直すトレードオフに見合うほど重要か。作り直すとネイティブUIが無償で提供するアクセシビリティ・キーボード操作・IME・支援技術との連携を失う

■ やるべきこと（Dos）

- `::selection`で選択範囲の色を変える
- `accent-color`でチェックボックスなどのブラウザ生成UIにページのアクセントカラーを反映する
- `color-scheme`でブラウザUIをライト/ダークに追従させる
- スクロールバーは`scrollbar-color`（つまみとトラックはコントラスト比3:1以上）と`scrollbar-width`で調整する
- 入力の妥当性は`:user-invalid` / `:user-valid`でスタイリングする。ユーザーが操作したあとだけマッチし、読み込み時点で空の必須欄をエラー表示しない
- `<input>`・`<textarea>`・`<button>`は色・ボーダー・背景・タイポグラフィの目的で通常要素として扱える

```css
::selection {
  background: var(--color-accent);
  color: var(--color-on-accent);
}

.panel {
  /* 1つ目がつまみ、2つ目がトラックの色 */
  scrollbar-color: var(--color-accent) var(--color-track);
  scrollbar-width: thin;
}

input:user-invalid {
  border-color: var(--color-error);
}
```

■ やってはいけないこと（Don'ts）

- 本文テキストに`user-select: none`を使わない。コピー&ペーストや翻訳ツール、支援技術の読み上げを壊す。ドラッグハンドルやツールバーなどのUI部品だけに限る

```css
/* 本文の選択・コピー・読み上げを妨げる */
article {
  user-select: none;
}
```

- スクロール可能な領域に`scrollbar-width: none`を設定しない。`none`はスクロールを別の手段で完全に置き換えた場合だけに使う
- 妥当性のスタイルに`:invalid` / `:valid`を使わない。読み込み直後に、空の必須フィールドをエラー表示してしまう

```css
/* 読み込んだ瞬間、空の必須欄が赤くなる */
input:invalid {
  border-color: var(--color-error);
}
```

https://caniuse.com/css-appearance

#### テキスト系フィールド（`<input>`・`<textarea>`）

色・ボーダー・背景・タイポグラフィの目的では、通常のテキストコンテナとして扱えます。

■ やるべきこと（Dos）

- プレースホルダーは`:placeholder-shown`と`::placeholder`でスタイリングする
- `<textarea>`は`resize: vertical`で横方向のリサイズを無効化、`resize: none`で全リサイズを無効化する

```css
input::placeholder {
  color: var(--color-text-subtle);
}

textarea {
  /* 縦方向のリサイズだけ許可する */
  resize: vertical;
}
```

#### 複数選択コントロール（ラジオ・チェックボックス）

■ やるべきこと（Dos）

- ページ内にインラインで並べて選ばせる:各選択肢を`<label>`で包んだ`<input type=checkbox>`か`<input type=radio>`を使い`label:has(:checked)`でスタイリングする
- チェックボックス・ラジオ・スイッチは`appearance: none`と生成コンテンツ（`::before`/`::after`）か背景画像でチェック状態を描く

```css
/* チェックされた選択肢のラベルを強調する */
label:has(:checked) {
  border-color: var(--color-accent);
}

.custom-checkbox {
  /* 既定の見た目を消し、生成コンテンツで描き直す */
  appearance: none;
}
```

#### テキスト系でない `<input>`（ボタン・スライダー・ファイル入力など）

■ やるべきこと（Dos）

- ファイル入力: `::file-selector-button`でボタンをスタイリングする
- スライダー: `appearance: none`と、つまみ用（`::-webkit-slider-thumb`・`::-moz-range-thumb`など）・トラック用（`::-webkit-slider-runnable-track`・`::-moz-range-track`など）の擬似要素で制御する

```css
input[type="file"]::file-selector-button {
  background: var(--color-accent);
  color: var(--color-on-accent);
}

input[type="range"] {
  appearance: none;
}
input[type="range"]::-webkit-slider-thumb {
  /* つまみの見た目を独自に描く */
  appearance: none;
}
```

■ やってはいけないこと（Don'ts）

- `type`が`button`・`submit`・`reset`の`<input>`を使わない。`<button>`を使い通常の要素としてスタイリングする

```html
<!-- 避ける -->
<input type="button" value="送信">

<!-- 代わりに button を使う -->
<button type="submit">送信</button>
```

> 原典: [Modern Web Guidance / CSS — 5. Design tokens and theming](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#5-design-tokens-and-theming)

### 高コントラスト設定には`prefers-contrast`で対応する

アクセントを控えめな色（薄いグレーのスクロールバーなど）で作っている場合は、高コントラストを好むユーザー向けに濃い色の上書きを用意します。すでに十分なコントラストがあるなら不要です。

■ やるべきこと（Dos）

- 低コントラストの装飾を使うときだけ、`@media (prefers-contrast: more)`で黒×白のようなはっきりした色を上書きする

```css
@media (prefers-contrast: more) {
  .scroller {
    --scrollbar-thumb: #000000;
    --scrollbar-track: #ffffff;
  }
}
```

https://caniuse.com/mdn-css_at-rules_media_prefers-contrast

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/user-experience/adapt-scrollbar-to-contrast-preferences.md

## フォーム部品をCSSでテーマする（Form controls）

フォーム部品の見た目もCSSだけで整えられます。`forms`ガイドから、JavaScriptに頼らずCSSで完結する指針を補足します。

### 自動入力された欄は`:autofill`でハイライトする

`:autofill`は、ブラウザが自動入力し、ユーザーがまだ編集していないフィールドにマッチするCSS疑似クラスです。`<input>`・`<select>`・`<textarea>`で使えます。入力済みの欄を目立たせ、フォーム完了まで迷わせないために使います。なお自動入力された欄では`background-color`を直接上書きできません。背景は`box-shadow`のインセットで作ります。

■ やるべきこと（Dos）

- `:autofill`で自動入力済みの欄に独自の枠線や背景を当てる。背景は`box-shadow`のインセットで作る
- 状態は色だけに頼らず、枠線の太さや背景の濃淡など複数の手がかりで示す
- 自動入力欄をスタイルするときも、`:focus-visible`で明示的な高コントラストのフォーカス表示を必ず用意する

```css
input:autofill,
input:-webkit-autofill {
  /* 色だけに頼らず、枠線と背景の両方で状態を示す */
  border: 2px solid #2e7d32;
  box-shadow: 0 0 0 100vmax #e8f5e9 inset;
}

/* 必須: 自動入力欄をスタイルするときも明示的なフォーカス表示を用意する */
input:autofill:focus-visible,
input:-webkit-autofill:focus-visible {
  outline: 3px solid #000;
  outline-offset: 2px;
}
```

■ やってはいけないこと（Don'ts）

- 自動入力済みの状態を枠線の色だけで示さない。色覚特性によっては気づけない
- フォーカスの輪郭（`outline: none`）を、代替の高コントラスト表示なしに消さない
- 疑似クラス名を`:auto-fill`と書かない。正しくは`:autofill`

`:autofill`はプログレッシブエンハンスメントです。非対応のブラウザ（Firefoxなど）でもフォームは普通に機能し、ハイライトが付かないだけです。JavaScriptのフォールバックは不要です。

https://caniuse.com/mdn-css_selectors_autofill

> 原典: [Modern Web Guidance / Forms — Autofill highlight inputs](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/forms/autofill-highlight-inputs.md#how-to-implement)

## レスポンシブデザイン（Responsive design）

### ビューポートではなくコンテナの幅に合わせるなら`@container`を使う

`@container`クエリは、要素を親コンテナの大きさに応じて切り替える仕組みです。画面幅を基準にするメディアクエリと違い、置かれた場所のコンテナ幅でレイアウトが変わります。

■ やるべきこと（Dos）

- 要素を親コンテナの大きさに応じて切り替えるなら`@container`を使う。同じコンポーネントをサイドバーにもメインにも置ける。コンテナ幅を基準にしたサイズ単位`cqi`などとあわせて使う

なお`dvh` / `dvw`といったビューポート単位や`100vw`の扱いは別のセクションで扱います。

https://caniuse.com/css-container-queries

### メディア要素には`aspect-ratio`を指定する

■ やるべきこと（Dos）

- `<img>`や`<video>`には`aspect-ratio`で縦横比を指定し、読み込み中に表示領域を確保する。要素が後から表示されてレイアウトがずれるCumulative Layout Shift（CLS）を防げる

https://caniuse.com/mdn-css_properties_aspect-ratio

### レスポンシブなフォントサイズには`clamp()`を使う

ビューポート相対の単位とフォント相対の単位を`clamp()`の中で組み合わせます。画面幅に応じてフォントサイズが伸縮しつつ、上限と下限の範囲に収まります。

■ やるべきこと（Dos）

- ビューポート相対とフォント相対の単位を`clamp()`で組み合わせる。割合を変えると、フォントサイズが変化する速さを調整できる

```css
.title {
  font-size: clamp(2rem, 1rem + 5vw, 4rem); /* 下限 2rem、上限 4rem の範囲で可変 */
}
```

■ やってはいけないこと（Don'ts）

- `vw`だけで`font-size`を指定しない。`clamp()`がないと、極端に大きい画面や小さい画面で文字が大きくなりすぎたり小さくなりすぎたりする

```css
/* 極端な画面幅で文字が破綻する */
.title {
  font-size: 5vw;
}
```

https://caniuse.com/css-math-functions

> 原典: [Modern Web Guidance / CSS — 6. Responsive design](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#6-responsive-design)

## タイポグラフィ（Typography）

### `line-height`には単位なしの数値を使う

■ やるべきこと（Dos）

- `line-height`には`1.5`のように単位なしの数値を指定する。フォントサイズを継承するとき、その倍率で行の高さも一緒に拡大縮小される

### 長いURLには`overflow-wrap: break-word`を使う

■ やるべきこと（Dos）

- 長いURLのはみ出しは`overflow-wrap: break-word`で防ぐ。どうしても収まらないときは`anywhere`も使える

https://caniuse.com/wordwrap

### `font-size`に`px`を使わない

■ やるべきこと（Dos）

- `font-size`には`rem`を使い、ユーザーがブラウザで設定したフォントサイズ（ルートのフォントサイズ）を尊重する。文脈に応じたサイズには`em`を使う

■ やってはいけないこと（Don'ts）

- `font-size`に`px`を使わない。ユーザーのフォントサイズ設定を無視してしまう

```css
/* ユーザーのフォントサイズ設定を無視する */
body {
  font-size: 16px;
}
```

### 見出しには`text-wrap: balance`、本文には`text-wrap: pretty`を使う

`text-wrap`は行の折り返し方を制御するプロパティです。

■ やるべきこと（Dos）

- 見出しや見出しに準ずる要素（`<th>`など）には`text-wrap: balance`。各行の長さが揃う
- 段落や引用などの長い本文には`text-wrap: pretty`。「1単語だけ次の行」を防ぐ

```css
.heading { text-wrap: balance; } /* 見出し向け。各行の長さを均す */
.prose   { text-wrap: pretty; }  /* 本文向け。孤立行を防ぐ */
```

■ やってはいけないこと（Don'ts）

- `balance`や`pretty`を`*`（全要素）に当てない。パフォーマンスのコストがある

```css
/* 全要素に当てるとパフォーマンスのコストが大きい */
* {
  text-wrap: pretty;
}
```

- 背景・枠線・影などで囲まれた要素では`text-wrap: balance`を避ける。`balance`はコンテナの幅を変えず、その幅の中での折り返し方だけを変えるため、右端に余白が残りやすい

https://caniuse.com/css-text-wrap-balance

> 原典: [Modern Web Guidance / CSS — 7. Typography](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#7-typography)

### 折り返しを止めるなら`text-wrap: nowrap`を使う

ナビゲーションタブや横スクロールのチップなど、折り返すと崩れる要素では折り返しを止めます。`white-space: nowrap`より意味の伝わる`text-wrap: nowrap`を使います。

■ やるべきこと（Dos）

- 折り返しを止めるには`text-wrap: nowrap`を使う
- はみ出しは`overflow`で処理する。省略記号`text-overflow: ellipsis`を出すなら`overflow: hidden`を併せる

```css
.no-wrap {
  text-wrap: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

https://caniuse.com/mdn-css_properties_text-wrap

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/user-experience/prevent-text-wrapping.md

### Webフォントのズレは`font-size-adjust`で抑える

Webフォントが読み込まれると、同じ`font-size`でも代替フォントと文字の大きさが変わり、レイアウトシフト（CLS）や読みにくさが起きます。`font-size-adjust`は、フォントのx-heightなどを基準に大きさをそろえ、どのフォントでも見た目の大きさを保ちます。

■ やるべきこと（Dos）

- `font-size-adjust: from-font`で主フォントのx-heightを基準に代替フォントを正規化する。読み込み中や失敗時のCLSを防ぐ
- 大文字主体の見出しは`cap-height from-font`、等幅は`ch-width`など基準を選べる

```css
.text-content {
  font-family: "MyWebFont", "Arial", sans-serif;
  /* 主フォントの x-height に代替フォントを合わせ、CLSを防ぐ */
  font-size-adjust: from-font;
}
```

https://caniuse.com/mdn-css_properties_font-size-adjust

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/user-experience/visually-stable-font-fallbacks.md

## 視覚効果（Visual effects）

### 影を重ねて奥行きを表現する

■ やるべきこと（Dos）

- 複数の影を重ねて奥行きを出す
- 非矩形の図形や透過PNGには`box-shadow`ではなく`filter: drop-shadow()`を使う。要素の不透明な形に沿って影が付く

https://caniuse.com/css-boxshadow

### 光のオーバーレイには`mix-blend-mode`/`background-blend-mode`を使う

ブレンドモードは、重なった色をどう混ぜて描画するかを指定する機能です。

■ やるべきこと（Dos）

- 光が当たったようなオーバーレイには`mix-blend-mode`と`background-blend-mode`を使う。効果の及ぶ範囲は`isolation: isolate`で限定する

```css
.hero {
  background-image: url('texture.png'), linear-gradient(to bottom, #fff, #eee);
  background-blend-mode: soft-light; /* テクスチャと背景を柔らかい光として合成 */
}
```

https://caniuse.com/css-mixblendmode

### 比率の取れたカーブには楕円形の`border-radius`を使う

■ やるべきこと（Dos）

- 要素を増やさず比率の取れたカーブには楕円形の`border-radius`（例`10px / 20px`）を使う

### グラデーションの補間には`oklch`か`oklab`を指定する

`oklch`や`oklab`は、色を人の見た目に近い形で扱う新しい色空間です。

■ やるべきこと（Dos）

- グラデーションや`color-mix()`では、補間する色空間を`in oklch`か`in oklab`で明示する
  - `in oklch`:彩度（chroma）をよく保つ。ただし色の差が大きいとデバイスの色域からはみ出しやすい
  - `in oklab`:色域に収まりやすい。ただし中間でくすんだ色になりやすい（特に正反対の色相を補間するとき）

■ やってはいけないこと（Don'ts）

- 特別な理由がない限り`in srgb`を使わない。srgbで補間する必要があるカラーピッカーを作る場合などが例外

https://speakerdeck.com/tonkotsuboy_com/lu-ye-sanniwen-ku-cssnozui-xin-torendo-ver-dot-2026

https://caniuse.com/mdn-css_types_color_oklch

#### 古いブラウザにはフォールバックを用意する

2024年より前の一部ブラウザは、グラデーションの補間色空間に対応していません。

■ やるべきこと（Dos）

- 変数を定義し、安全なときだけトークンを使う

```css
:root {
  --in-oklab: ;
  --in-oklch: ;
}

@supports (linear-gradient(in oklab, white, black)) {
  :root {
    --in-oklab: in oklab;
    --in-oklch: in oklch;
  }
}
```

使うときは次のようにします。

```css
.card {
  background: linear-gradient(to bottom var(--in-oklab), var(--accent-color), var(--darker));
}
```

■ やってはいけないこと（Don'ts）

- この手法を使うとき、変数の前に置く固定の記述（例: `to bottom`）を省かない。これがないと、古いブラウザで変数が空になったときに構文エラーになる

なお`color-mix()`にはこの手法は不要です。`color-mix()`に対応するブラウザは、その`in <色空間>`引数にも対応しています。

### 模様はCSSグラデーションとハードストップで描く

多くの模様は、CSSグラデーションとハードストップ（色を急に切り替える指定）で作れます。SVGや外部画像より軽いことがあります。周囲の文脈からCSS変数や長さを参照できるためです。位置を2回繰り返す必要はありません。`0`または`0%`だけ書けば、グラデーションの補正で自動調整されます。

■ やるべきこと（Dos）

- 模様はCSSグラデーションとハードストップで描く

▼ それぞれ幅`1em`の縦ストライプ

```css
.box {
  background: linear-gradient(to right, var(--color-1) 50%, var(--color-2) 0) 0 / 2em;
}
```

▼ それぞれ幅`1em`の斜めストライプ

```css
.box {
  background: repeating-linear-gradient(-45deg, var(--color-1) 0 1em, var(--color-2) 0 2em);
}
```

▼ `1em`の正方形によるチェッカーボード

```css
.box {
  background: repeating-conic-gradient(var(--color-1) 0 25%, var(--color-2) 0 50%) 0 / 2em 2em;
}
```

▼ 半径`.5em`の点を`2em`間隔で並べた水玉

```css
.box {
  --distance: 2em;
  --radius: .5em;
  --polka: radial-gradient(circle, var(--color-1) var(--radius), transparent calc(var(--radius) + 1px));
  background: var(--polka) 0 0, var(--polka) var(--distance) var(--distance) var(--color-2);
  background-size: calc(var(--distance) * 2) calc(var(--distance) * 2);
}
```

▼ 円グラフ

```css
.pie {
  --p: 80%;
  width: 60px;
  aspect-ratio: 1;
  border-radius: 50%;
  background: conic-gradient(var(--color-1) var(--p), transparent 0%) var(--color-2);
}
```

■ やってはいけないこと（Don'ts）

- グラデーションでグラフを描くとき、テキストの代替なしで済ませない。スクリーンリーダー向けに意味のあるデータテーブルを必ず併設する。詳しくは[MWGアクセシビリティガイドの代替テキスト・メディアの指針](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/accessibility/accessibility.md#6-alternate-text-and-media)を参照してください

https://caniuse.com/css-gradients

> 原典: [Modern Web Guidance / CSS — 8. Visual effects](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#8-visual-effects)

## アニメーションとトランジション（Transitions & animations）

### 凝った演出には`clip-path`・`mask-image`・ビュートランジションを使う

■ やるべきこと（Dos）

- 図形を切り抜きながら見せる演出には`clip-path`と`mask-image`。`mask-image`は別の画像を使って要素の見える範囲を切り抜く機能で、滑らかなフェードアウトも作れる
- 複雑なレイアウト状態の切り替えには、ビュートランジション（View Transitions）

https://caniuse.com/css-clip-path

### アニメーションは`opacity`と`transform`を優先する

`opacity`と`transform`は、レイアウト計算とは別のスレッド（コンポジタ）で処理されます。レイアウトを再計算せずにアニメーションを動かせます。

■ やるべきこと（Dos）

- `opacity`と`transform`（`translate`などの個別プロパティを含む）を優先する

```css
.panel {
  transition: translate 0.2s; /* 別スレッド（コンポジタ）で処理される */
}
```

■ やってはいけないこと（Don'ts）

- `left`や`top`のようにレイアウトを伴うプロパティをアニメーションしない。描画コストが高くカクつきの原因になる

```css
.panel {
  transition: left 0.2s; /* レイアウトが走る */
}
```

### `display`の切り替えは`allow-discrete`と`@starting-style`でアニメーションする

`display`や`<dialog>`の開閉のような状態は、本来アニメーションできません。

■ やるべきこと（Dos）

- `transition-behavior: allow-discrete`と`@starting-style`でネイティブにアニメーションさせる。`@starting-style`は、要素が表示され始める瞬間の初期スタイルを指定する仕組み

```css
.popover-reveal {
  /* display の切り替えにアニメーションを許可する */
  transition: display 0.2s allow-discrete;
}
```

https://speakerdeck.com/tonkotsuboy_com/cssnozui-xin-torendo-ver-dot-2025

https://caniuse.com/mdn-css_at-rules_starting-style

### `content-visibility`には必ず`contain-intrinsic-size`を添える

`content-visibility: auto`は、画面外の要素の描画を後回しにして表示を速くする機能です。

■ やるべきこと（Dos）

- `content-visibility`には必ず`contain-intrinsic-size`を組み合わせる。描画をスキップした要素に仮の寸法を与えるため
- `contain-intrinsic-size`には`auto`キーワードと、中身から見積もった値を指定する。単位は`px`ではなく`rem`・`lh`・`cap`・`ch`を優先する。アイテムごとに大きさがばらつくなら平均値を使う

```css
.large-section {
  content-visibility: auto;
  contain-intrinsic-block-size: auto 800px;
}

.row {
  --row-gap: 0.4rem;
  --title-height: 1lh;
  --description-height: 0.85lh;

  display: grid;
  row-gap: var(--row-gap);
  content-visibility: auto;
  /* タイトル・行間・説明文の高さの合計を、描画スキップ時の寸法にする */
  contain-intrinsic-block-size: auto calc(
    var(--title-height) + var(--row-gap) + var(--description-height)
  );
}
```

■ やってはいけないこと（Don'ts）

- `content-visibility`を`contain-intrinsic-size`なしで単体で使わない。スクロールバーが飛び跳ね、レイアウトシフト（CLS）の原因になる

https://caniuse.com/css-content-visibility

### `contain`でコンポーネントの再描画を閉じ込める

■ やるべきこと（Dos）

- コンポーネント単位で描画更新を切り離すには`contain: layout style paint`を使う。`contain`は、要素の内部の変更が外側に波及しないとブラウザに伝える機能で、再描画の範囲を狭められる

https://caniuse.com/css-containment

### `prefers-reduced-motion`で激しい動きを抑える

■ やるべきこと（Dos）

- `prefers-reduced-motion`メディアクエリで激しいモーションをオフにする。減らし方は個別に対応するか、カスタムプロパティを使う

```css
@property --animation-reduced {
  syntax: "*";
  inherits: false;
  initial-value: none;
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation: var(--animation-reduced) !important;
  }
}
```

この方法なら、減らした版のアニメーションを元のアニメーションと同じ場所にまとめて書けます。

```css
progress:not([value]) {
  animation: slide 1s infinite linear;
  --animation-reduced: slide 20s infinite linear;
}
```

■ やってはいけないこと（Don'ts）

- 全要素に`animation-duration: 0.01ms`を一律で当てない。一部のアニメーションがかえって不快になることがある

```css
/* 一律に止めると、かえって不快になるアニメがある */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
  }
}
```

https://caniuse.com/prefers-reduced-motion

> 原典: [Modern Web Guidance / CSS — 9. Transitions & animations](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#9-transitions--animations)

### バネ・バウンドには`linear()`イージングを使う

`linear()`は、複数の停止点で複雑なカーブを近似するイージング関数です。`ease-in`や`cubic-bezier()`では作れないバネやバウンドを表現できます。停止点はツールで生成します。

■ やるべきこと（Dos）

- バネ・バウンドのような物理的な動きは`linear()`の多段ストップで表現する。停止点はカスタムプロパティに入れて使い回す
- `linear()`は時間を計算しないので`duration`を必ず指定する
- なめらかさとCSSサイズはトレードオフ。停止点を増やしすぎない

```css
.spring {
  --spring-easing: linear(0, 0.06 1%, 1.116 5.4%, 0.937 14.3% /* …停止点は省略… */, 1);
  /* 必須: linear() は duration を自動計算しない。必ず指定する */
  transition: scale 0.8s var(--spring-easing);
}
```

■ やってはいけないこと（Don'ts）

- バウンドのイージングを`opacity`に当てない。値が0未満や1超に行き過ぎてチラつく

https://caniuse.com/mdn-css_types_easing-function_linear-function

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/user-experience/physics-based-easing.md

### 単一の変形だけ動かすなら`translate`・`rotate`・`scale`を使う

`transform`は複数の変形をまとめて指定するため、1つだけ変えるにも全体を書き直す必要があります。個別プロパティ（`translate`・`rotate`・`scale`）なら、`:hover`などで1つの変形だけを上書きできます。適用順は記述順によらず`translate`→`rotate`→`scale`→`transform`の固定です。

■ やるべきこと（Dos）

- 状態変化で1つの変形だけ変えるなら個別プロパティを使う。重なり合うアニメーションを衝突させずに定義できる
- 状態変化（`:hover`など）で変形する要素には、基底に変形なしの初期値（`translate: 0`・`scale: 1`など）を置く。変形時にスタッキングコンテキストが急に変わるのを防ぐ

```css
.card {
  animation: float 3s infinite ease-in-out; /* translate を動かす */
  transition: scale 0.3s ease;
  scale: 1; /* 変形なしの初期値。突然のスタッキングコンテキスト変化を防ぐ */
}
.card:hover {
  scale: 1.05;
}
```

■ やってはいけないこと（Don'ts）

- 「先にscale、後でrotate」のように別の順序が必要なときは個別プロパティを使わない。順序は固定なので`transform`関数で書く

https://caniuse.com/mdn-css_properties_translate

https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/user-experience/individual-transform-properties.md

## 生成コンテンツ（Generated content）

### `content`で意味のあるテキストを表現しない

■ やってはいけないこと（Don'ts）

- ラベルや状態、操作の説明など、意味を持つテキストを`content`で出さない。これらはDOM側に置く（[WCAG F87](https://www.w3.org/WAI/WCAG22/Techniques/failures/F87)）

```css
/* 意味のあるテキストを CSS に置かない */
.badge::before {
  content: "新着";
}
```

`content`の代替テキスト引数は、装飾のつもりが意味を持ってしまった場合の被害を減らす手段です。意味のあるテキストをCSSに置く言い訳には使えません。

https://caniuse.com/css-gencontent

### `content`の代替テキスト引数を正しく使う

`content`のスラッシュ以降に書くテキストは、スクリーンリーダー向けの代替テキストになります。

■ やるべきこと（Dos）

- アイコン画像に読み上げ用の文言を添える

```css
.icon-save::before {
  content: url(cloud.svg) / "Save"; /* スクリーンリーダーは「Save」と読む */
}
```

- 純粋に装飾のテキストを読み上げさせたくないときは、代替テキストを空にする

```css
.decorative::before {
  content: "•" / ""; /* 装飾なので読み上げさせない */
}
```

- 代替テキスト引数は、その値が主たる値と異なり、かつDOMにまだ無いときだけ使う

■ やってはいけないこと（Don'ts）

- 画像に空の代替テキスト引数を付けない。画像はもともと装飾扱い

```css
.icon::before {
  content: url(cloud.svg) / ""; /* 画像はもともと装飾扱いなので不要 */
}
```

- 絵文字の説明は、公式の絵文字名と同じなら付けない（公式名と違う意図のときだけ付ける）

```css
.party::before {
  content: "🎉" / "celebration"; /* 公式名とほぼ同じなので不要 */
}
```

正しくは、公式名と違う意図を伝えるときだけ付けます。

```css
.party::before {
  content: "🎉" / "Yay!"; /* 公式名と違うので意味がある */
}
```

- DOMにある値と同じ代替テキストを付けない。二重に読み上げられる

```html
<button class="save">Save</button>
```

```css
button.save::before {
  content: url(cloud.svg) / "Save";
}
```

このとき、スクリーンリーダーは「Save save」と読み上げてしまいます。

https://caniuse.com/css-gencontent

> 原典: [Modern Web Guidance / CSS — 10. Generated content](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/css/css.md#10-generated-content)

## さいごに

ここまでCSSの2つのガイドからDos / Don'tsを網羅しました。量は多いですが、ひとつひとつは「古い手癖を新しい標準に置き換える」だけのものです。

筆者はCSSの新機能を追うのが好きです。それでも全部を覚えてはいられません。`:has()`や`@scope`、`light-dark()`、`text-wrap: balance`のように「知っていれば一行で済む」機能は、思い出せないと結局JavaScriptや回りくどいCSSに逃げてしまいます。このガイドはその抜けを埋めてくれます。

気になった項目から、自分のCSSを見直してみてください。AIエージェントに書かせるなら、冒頭で紹介したスキルをそのまま入れるのが手っ取り早いです。

https://github.com/GoogleChrome/modern-web-guidance

## 出典・ライセンス

本記事は **Modern Web Guidance**（Google Chrome、[Apache License 2.0](https://github.com/GoogleChrome/modern-web-guidance/blob/main/LICENSE)）を日本語へ翻訳・再構成し、筆者の見解や補足を加えたものです。コード例への日本語コメント追加など、原文に変更を加えています（逐語訳ではありません）。参照したガイドは各セクション末尾にリンクしています。「Google」「Chrome」は各社の商標で、出典明示のために使用しています。

https://github.com/GoogleChrome/modern-web-guidance