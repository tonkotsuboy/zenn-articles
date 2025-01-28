---
title: "TypeScript 5.8でenumやnamespaceを撲滅できるようになった"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "javascript"]
published: true
publication_name: ubie_dev
---

TypeScript 5.8で導入される`erasableSyntaxOnly`フラグ、`enum`や`namespace`、クラスのパラメータプロパティ、`module`キーワードなどの構文をエラーできるようになります。これらの構文はNode.jsでTypeScriptを実行する際に非互換な構文であり、本フラグの導入によりNode.jsとTypeScriptの互換性が高まります。

本記事では、`erasableSyntaxOnly`フラグの挙動と、なぜこのフラグが導入されたのかを解説します。

# `erasableSyntaxOnly`フラグの挙動

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
  constructor(private myField: string) {}
}
```

各構文がエラーになっていることがわかります。エラーになっているのは文です。

![erasableSyntaxOnlyによるエラー](/images/ts-erasable-syntax-only/ts-error.png)

次のリンクから動作を確認できます。

https://www.typescriptlang.org/play/?erasableSyntaxOnly=true&ts=5.8.0-dev.20250126#code/KYOwrgtgBAsgngUXNA3gKClAggGg1AIT0wGE8BfNNEAQwmAGcAHGgY2CgjgDk7gBlFuyjpMwAB5MA9gCcALlFZSQDBQDMpUqAF4oARgDcaSmghSAJmAA2HeDAvWOoqBOnzFy1VA1bdh41SsVjQMDLBwJMGhIvhKKnIyYKxysgAUTDIAlgBuNHIcXABimcBW5gBcUKpZIADmAJQxmJSUQA

# なぜ`erasableSyntaxOnly`フラグが導入されたのか

## 前提: Node.jsでTypeScriptの型注釈を削除し、そのまま実行できるようになった

Node.js 22.7で導入された `experimental-transform-types` オプションを使うと、TypeScriptのコードをnode.jsで実行する際に、TypeScriptの型注釈などの構文を削除し、JavaScriptのコードとして実行できるようになります。

たとえば次のようなTypeScriptのコードを記述し、`index.ts`というファイル名で保存します。

```ts
const myName: string = "とんこつ";
console.log(myName);
```

Node.js 22.7で導入された `experimental-transform-types` オプションを使うと、型注釈（`const myName: string`の`: string`部分）が削除され、JavaScriptのコードとして実行できるようになります。

```bash
node --experimental-transform-types index.ts
```

実行結果は次のとおりです。

![Node.jsでTypeScriptを実行する](/images/ts-erasable-syntax-only/node-for-ts.png)

mizchiさんの記事がわかりやすいです。

https://zenn.dev/mizchi/articles/experimental-node-typescript

## 型注釈の削除だけでは実行できないTypeScriptの構文をエラーにする

しかし、`enum`や`namespace`、クラスのパラメータプロパティなどの構文は、単純な型注釈の削除ではない変換処理が必要になり、「削除不可能な構文」とみなされます。

`erasableSyntaxOnly` フラグが提案されたのは、こういったNode.jsの削除不可能な構文をエラーにし、Node.jsのTypeScript実行との互換性を高める目的がありました。

https://github.com/microsoft/TypeScript/issues/59601

`erasableSyntaxOnly` フラグでエラーとみなされる主な構文は次のとおりです。

- `enum`
- `namespace`
- クラスのパラメータプロパティ
- レガシーな`module`

レガシーな`module`とは、TypeScript独自の`module`キーワードを使ったコードのことです。昔使われていました。

なお、DecoratorsもNode.jsでは削除不可能構文とみなされますが、TypeScriptの`erasableSyntaxOnly`フラグをONにしてもエラーになりません。

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
  private myField: string;

  constructor() {}
}
```

JavaScriptのプライベートフィールドを使うのも手でしょう。

```ts
class MyClass2 {
  #myField: string;

  constructor() {}
}
```

それぞれ、`erasableSyntaxOnly` フラグをONにしてもエラーになりません。

![TypeScriot 5.8でエラーが出ていない様子](/images/ts-erasable-syntax-only/ts-no-error.png)

※ TypeScript Playgroundにて、TS Configタブから`erasableSyntaxOnly`フラグをONにして動作確認

唯一ちょっと悩んでいるのがクラスのパラメータプロパティで、筆者はNestJSのコードを書く際に使っています。

# 参考資料

https://www.totaltypescript.com/erasable-syntax-only

https://nodejs.org/api/typescript.html#typescript-features
