---
title: "TypeScript 5.8のerasableSyntaxOnlyフラグ。enumやnamespaceが消える日が来た"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "nodejs", "javascript"]
published: true
publication_name: ubie_dev

---

TypeScript 5.8で導入される`erasableSyntaxOnly`フラグを使うと、`enum`や`namespace`、クラスのパラメータプロパティ、`module`キーワードなどの構文をエラーとして検出できます。これらの構文はNode.jsでTypeScriptを実行する際に非互換な構文であり、本フラグの導入によりNode.jsとTypeScriptの互換性が高まります。

本記事では、`erasableSyntaxOnly`フラグの挙動と、なぜこのフラグが導入されたのかを解説します。

# `erasableSyntaxOnly`フラグの挙動

`erasableSyntaxOnly` とは、「削除可能な構文のみ」という意味です。削除可能な構文とは、Node.jsでTypeScriptを実行される際に削除される構文のことです。後ほど詳しく解説します。

`erasableSyntaxOnly`フラグは、tsconfig.jsonの`compilerOptions`に`"erasableSyntaxOnly": true`を設定することで有効になります。

```json
{
  "compilerOptions": {
    "erasableSyntaxOnly": true
  }
}
```

このフラグをONにした状態で、`enum`や`namespace`、クラスのパラメータプロパティなどの構文を使ってみます。

```ts
enum MyEnum {
  A,
  B,
  C,
}

namespace myNameSpace {
  export const foo = 1;
}

class MyClass {
  constructor(myField: string) {}
}
```

TypeScript Playgroundで動作を確認すると、各構文がエラーになっていることがわかります。

![erasableSyntaxOnlyによるエラー](/images/ts-erasable-syntax-only/ts-error.png)

次のリンクから動作を確認できます。

https://www.typescriptlang.org/play/?erasableSyntaxOnly=true&ts=5.8.0-dev.20250126#code/KYOwrgtgBAsgngUXNA3gKClAggGg1AIT0wGE8BfNNEAQwmAGcAHGgY2CgjgDk7gBlFuyjpMwAB5MA9gCcALlFZSQDBQDMpUqAF4oARgDcaSmghSAJmAA2HeDAvWOoqBOnzFy1VA1bdh41SsVjQMDLBwJMGhIvhKKnIyYKxysgAUTDIAlgBuNHIcXABimcBW5gBcUKpZIADmAJQxmJSUQA

※ TypeScript Playgroundにて、TS Configタブから`erasableSyntaxOnly`フラグをONにして動作確認

# なぜ`erasableSyntaxOnly`フラグが導入されたのか

## 前提: Node.jsでTypeScriptの型注釈を削除し、そのまま実行できるようになった

最近のNode.jsでは、TypeScriptのコードをそのまま実行できます。従来のようにts-nodeを使う必要はありません。Node.jsがTypeScriptの型注釈を削除し、JavaScriptのコードとして実行しているのです。

Node.js v22.7では `‎--experimental-strip-types` フラグを使う必要がありました。

最新のNode.js v23.6では、フラグすらなしにTypeScriptコードを実行できます。

たとえば次のようなTypeScriptのコードを記述し、`index.ts`というファイル名で保存します。

```ts
const myName: string = "とんこつ";
console.log(myName);
```

`index.ts`をNode.jsで実行するには、次のように`node`コマンドを使うだけです。

```bash
node index.ts
```

実行結果は次のとおりです。

![Node.jsでTypeScriptを実行する](/images/ts-erasable-syntax-only/node-for-ts.png)

## 型注釈の削除だけでは実行できないTypeScriptの構文をエラーにする

しかし、`enum`や`namespace`、クラスのパラメータプロパティなどの構文は、単純な型注釈の削除ではない変換処理が必要になり、「削除不可能な構文」とみなされます。

‎その場合Node.jsでは `--experimental-transform-types` フラグを使う必要があります。

`erasableSyntaxOnly` フラグが提案されたのは、こういったNode.jsの削除不可能な構文をエラーにし、Node.jsのTypeScript実行との互換性を高める目的がありました。

https://github.com/microsoft/TypeScript/issues/59601

`erasableSyntaxOnly` フラグでエラーとみなされる主な構文は次のとおりです。

- `enum`
- `namespace`
- クラスのパラメータプロパティ
- レガシーな`module`

レガシーな`module`とは、TypeScript独自の`module`キーワードを使ったコードのことです。以前のバージョンのTypeScriptで使用されていた構文です。

なお、DecoratorsもNode.jsでは削除不可能構文とみなされます[^1]が、TypeScriptの`erasableSyntaxOnly`フラグをONにしてもエラーになりません。

[^1]: https://nodejs.org/api/typescript.html#typescript-features

# `enum`や`namespace`などの構文がエラーになって困るか？

これは私個人の意見ですが、クラスのパラメータプロパティを除くと直近のコードで使う機会はほとんどありません。

`enum`を使っているコードは現場では極稀に見かけますが、`enum`のような挙動を実現したければ、次のようにオブジェクトリテラルを定義して、その型を使うようにしています。

```ts
const MyEnum = {
  A: 0,
  B: 1,
  C: 2,
} as const;

type MyEnum = typeof MyEnum;
```

`namespace`は、 モジュールと `import * as`を使うことで実現できます。

foo.ts

```ts
export const foo = "hello";
```

main.ts

```ts
// main.ts
import * as myNameSpace from "./foo";

console.log(myNameSpace.bar); // "hello"
```

クラスのパラメータプロパティは、次のように実現できます。

```ts
class MyClass {
  myField: string;

  constructor() {}
}
```

それぞれ、`erasableSyntaxOnly` フラグをONにしてもエラーになりません。

![TypeScriot 5.8でエラーが出ていない様子](/images/ts-erasable-syntax-only/ts-no-error.png)

https://www.typescriptlang.org/play/?ts=5.8.0-dev.20250127#code/KYDwDg9gTgLgBAYwgOwM7wGYQnAvHAIgAtgAbUiAgbgCgaFSBDVVOAWQE8BhJluAbxpw4YKAEsAboxjA4AWw4AxMWQAmALjjpxyAOa0hiFNoCuCGNAAUASgGHhMImNQA6BcrV5C7laVXVDAF8aYPpeVk4eZlQAJjthAGIfNU1tMT0DYSQ0GCgzCygbeOE4R2cXJKVfVS8CZL8A4WDgoA

※ TypeScript Playgroundにて、TS Configタブから`erasableSyntaxOnly`フラグをONにして動作確認


# `erasableSyntaxOnly`は歓迎すべき挙動

筆者的には`erasableSyntaxOnly`は嬉しい挙動です。とりわけ`enum`については、生成されるJavaScriptコードが好みでなかったり、オブジェクトで表現したほうが各値のループの表現がしやすかったりで、ESlintで禁止して使わないようにしていました。また、Node.jsで動作するTypeScriptとの互換性が高まったこともメリットです。TypeScript 5.8にアップデートしたら早速フラグをONにするつもりです。

唯一ちょっと悩んでいるのがクラスのパラメータプロパティで、筆者はNestJSのコードを書く際に使っています。NestJSでは、クラスのパラメータプロパティを使った依存性の注入が一般的なので、対応方法を検討中です。

TypeScript 5.8は2025年2月25日にリリース予定ですので、今の内から挙動を試しておきましょう。

https://github.com/microsoft/TypeScript/issues/61023

# 参考資料

https://github.com/microsoft/TypeScript/issues/59601

https://nodejs.org/api/typescript.html#typescript-features

https://www.totaltypescript.com/erasable-syntax-only

https://www.totaltypescript.com/typescript-is-coming-to-node-23
