---
title: "CSS・TypeScriptの相性が抜群。vanilla-extractが最高のCSS開発体験をくれた"
emoji: "🍨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "css", "typescript", "react", "nextjs"]
published: true
publication_name: moneyforward
---

私はこれまで、 React ・ Next.js でのスタイリングには、CSS Modules + Sass を使っていましたが、最近は [vanilla-extract](https://vanilla-extract.style/) を使うようになりました。TypeScript との相性が良く、長い間求めていたCSS開発体験が実現できるためです。

vanilla-extract とは、CSS を TypeScript で型安全に書ける CSS in JS です。 State of CSS 2022 でも満足度が高く、先日は Next.js の appDir でも正式サポートされました。

![](/images/vanilla-extract/vanilla-extract_top.png)



本記事では、CSS Modules から vanilla-extract に移行した経緯と、そのメリットについて紹介します。

# CSS Modulesで限界を感じていた

CSS Modules を使っていた理由はいくつかありますが、主に次のようなものです。

- 従来の CSS の書き方と同じ
- スコープが使える

一方で、いくつかの限界を感じていました。

## 型のチェックができないので、存在しないセレクタを指定してもエラーにならない

CSS Modules をそのまま使うと、存在しない CSS セレクタを指定しても、型エラーになりません。

次のような CSS Modules のスタイル定義があったとします。

```scss
.foo {
  color: red;
}
```

このスタイルを使うコンポーネントで使ってみます。


```tsx
import type { FC } from "react";
import styles from "./MyComponent.module.scss";

export const MyComponent: FC = () => {
  const fooStyle = styles.foo; // string | undefined

  const barStyle = styles.bar; // string | undefined

  return (
    <div>
      <div className={fooStyle}></div>
      <div className={barStyle}></div>
    </div>
  );
};
```

`styles.foo` は存在するので、エラーにならないのは期待どおりです。
問題は `styles.bar` で、定義していないセレクタにも関わらず、型エラーになりません。`string | undefined` に推論されるためです[^1]。

[^1]: `strictNullChecks` が `true` の場合は `string | undefined` に、`false` の場合は `string` に推論されます


## `noPropertyAccessFromIndexSignature` を指定すると、ドットシンタックスでプロパティが書けない

CSS Modules におけるスタイルのオブジェクトは、次のように推論されます。

```ts
const styles: {
  reaonly [key: string]: string;
}
```


TypeScript 4.2 で導入された `noPropertyAccessFromIndexSignature` [^2]を `true` にすると、存在しない可能性があるプロパティへのドットシンタックスでのアクセスを禁止できます。 推論されたスタイルに対して、 `styles.foo` や `styles.bar` とドットシンタックスでアクセスしようとしても、`foo`や `bar` は存在しない可能性があるため、エラーとなるのです。

[^2]: https://www.typescriptlang.org/tsconfig#noPropertyAccessFromIndexSignature


これを防ぐためには、 `styles["foo"]` や `styles["bar"]` というように、ブラケットシンタックスでアクセスする必要があります。

`.foo` のセレクタの存在は明らかなのに、ブラケットシンタックスでアクセスを矯正されるのはもどかしいところです。

## typed-scss-modules を使えば型チェックが可能だが、型定義ファイルが煩わしい

私は、開発の現場でCSS modulesを使うときは、 [typed-scss-modules](https://www.npmjs.com/package/typed-scss-modules) を使っています。typed-scss-modules を使えば、`styles` 用の型定義ファイルが生成されます。

```ts
export type Styles = {
  foo: string;
};

export type ClassNames = keyof Styles;

declare const styles: Styles;

export default styles;
```

これにより、 型チェックが可能となり、前述の`foo`セレクタに対しても、`styles.foo`でのドットシンタックスアクセスが可能になります。存在しない`bar`に対しては、ドットシンタックスでエラーを検知できます。 `--watch` オプションを使えば、型定義ファイルも自動で生成できます。

概ね満足している動きではありますが、CSSファイルと型定義ファイルをそれぞれ別に管理しないといけないのが煩わしく感じていました。

# vanilla-extractで課題が解決した

私がCSS Moduleに感じていた課題を解決しつつ、大きな恩恵をもたらしてくれたのが、 [vanilla-extract](https://vanilla-extract.style/) でした。

vanilla-extract とは、CSS-in-JS の一つで、 CSS を TypeScript で型安全に書ける強力なライブラリです。一般的なCSS-in-JSとは異なり、実行時ではなくビルド時にCSSが生成されるので、ランタイムパフォーマンスの向上・出力されるCSSの最適化ができます。公式サイトでは、「Zero-runtime Stylesheets in TypeScript.」を謳っています。

vanilla-extract の作者 Mark Dalgleish 氏は、 CSS Modules の co-creator でもあります。 

https://twitter.com/markdalgleish

State of CSS 2022では、満足度が2位となっています。

![](/images/vanilla-extract/state-of-css.png)

https://2022.stateofcss.com/en-US/css-in-js/#css_in_js_experience_ranking



## 導入方法

vite, esbuild, webpack, rollupなど、各環境に応じて簡単に導入できます。

https://vanilla-extract.style/documentation/getting-started

## Next.js 13 appDirで正式サポート💐

先日、vanilla-extract は Next.js 13 の appDir でも動作するようになりました。

https://github.com/vercel/next.js/pull/48306

私は Next.13 appDir でポートフォリオサイトをリニューアルし、その際に vanilla-extract を導入しましたが、一瞬で導入できて本当にラクでした。

https://github.com/tonkotsuboy/kano-portfolio/pull/59/files

## vanilla-extract の基本的な使い方

まずは、vanilla-extract の基本的な使い方を見ていきましょう。

CSSは、`css.ts` というファイルに、 "TypeScriptで" 記述します。プロパティ名はキャメルケースで書きます。

```ts
import { style } from "@vanilla-extract/css";

export const foo = style({
  color: "red",
  backgroundColor: "blue",
});
```

TypeScript なので、 VSCode や JetBrains といった IDE でコード補完も動作します。他のライブラリを使うときと異なり、特別なプラグインを入れなくてもよいのが嬉しいですね。


このスタイルを使うコンポーネントで使ってみます。 TypeScript なので、スタイルは モジュールとしてインポートするだけです。

```tsx
import type { FC } from "react";
import { foo } from "./MyComponent.css";

export const MyComponent: FC = () => {
  return (
    <div>
      <div className={foo}></div>
    </div>
  );
};
```


## メリット①: 存在しないプロパティの型チェックができる

vanilla-extract では、スタイル定義をTypeScriptを使って記述します。これにより、存在しないプロパティ名を指定した場合や、不適切な値を指定した場合にコンパイルエラーを発生させられます。

```ts
export const foo = style({
  color: "red",
  hogehoge: "fugafuga", // エラー
});
```

## メリット②: スコープが持てる

CSS Modules や 他の CSS in JS  と同様に、スコープがあります。定義したスタイルは、他のコンポーネントに影響を与えません。

## メリット③: CSS 変数との相性が抜群

vanilla-extract はCSS 変数との相性が抜群で、私はこれがあるから vanilla-extract を選んだと言っても過言ではありません。

CSS 変数は、次のように定義します。

```ts
import { createGlobalTheme } from "@vanilla-extract/css";

export const vars = createGlobalTheme(":root", {
  color: {
    primary: "#3f3f9d",
    secondary: "#4a4e5a",
  },
  font: {
    size: {
      base: "1rem",
      l: "1.125rem",
    },
  },
});
```

定義した CSS 変数を使うときにも、型補完が効きます。

```ts
import { style } from "@vanilla-extract/css";
import { vars } from "./vars.css";

export const foo = style({
  fontSize: vars.font.size.s,
  color: vars.color.secondary,
});
```

![](/images/vanilla-extract/css-variables.png)

適用されたスタイルは、そのまま CSS 変数として出力されます。

![](/images/vanilla-extract/variables-result.png)
*CSS変数の出力を、Chrome DevToolで確認している様子*

## メリット④: ネストを深くできない

CSS Modules と Sass を使っているときの悩みの一つに、ネストを深く書けてしまうことがありました。

開発の現場で、地獄のように深いネストを見たことがある人は、きっと多いのではないでしょうか。 **私はあります**。どこにどのスタイルがあたっているのかがわかりづらく、消したらどんな影響があるかがわかりづらく、いいところは一つもありません。ネストは可能な限り減らすべきです。

```scss
.a {
  .b {
  }
  .c {
    .d {
      > .e {
      }
    }
    .e {
      .f {
      }
      .h, .i {
      }
      .g {
        .h {
        }
      }
    }
  }
}
```

vanilla-extract では、セレクタのネストを深くできません。エラーになります。

```ts
export const foo = style({
  color: "red",
  backgroundColor: "blue",
  
  ".bar": {} // エラー
});
```

一方で、`:hover` などの擬似クラスについては、ネストを使えます。便利です。

```ts
const bar = style({
  ':hover': {
    color: 'pink'
  },
  ':first-of-type': {
    color: 'blue'
  },
  '::before': {
    content: ''
  }
});
```

https://vanilla-extract.style/documentation/styling/#simple-pseudo-selectors


## その他の機能

vanilla-extractでは、 `@media` や `@container` も使えますし、型の恩恵も受けられます。たとえば、[コンテナクエリ](https://zenn.dev/moneyforward/articles/css-container-query)の記法は次のとおりです。

```ts
export const foo = style({
  fontSize: "16px",
  "@container": {
    "(800px < width)": {
      fontSize: "12px",
    },
  },
});
```

その他、必要な機能が揃っていて、今のところ不満はありません。

- `@media`
- `@container`
- `@layer`
- `@supports`
- `@font-face`

# まとめ

私は CSS のスタイル定義に、CSS Modules, emotion, tailwind 等を使っていきましたが、求めていた開発体験に一番近いのは vanilla-extract でした。 Next.js の appDir でも正式サポートしたことで、ますます使いやすくなりました。

私のポートフォリオサイトでは、 CSS Module から vanilla-extract へリプレイスしましたので、あわせて参考にしていただければと思います。

https://kano.codes/

https://github.com/tonkotsuboy/kano-portfolio
