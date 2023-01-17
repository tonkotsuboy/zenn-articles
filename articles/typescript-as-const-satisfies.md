---
title: "TypeScript 4.9のas const satisfiesが便利。型チェックとwidening防止を同時に行う"
emoji: "🐯"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "javascript"]
published: true
publication_name: moneyforward
---

TypeScript 4.9 から、`satisfies` operator が使えるようになりました。従来の`as const`と組み合わせ、型チェックと widening 防止を同時に行えます。筆者的には、"顧客が本当に必要だったもの"です。

本記事では `satisfies` とは何か？ `as const` とは何か？ 2つを組合わせるとどのようなメリットがあるのか？ について、実際のコードと共に解説します。

# 結論

TypeScriptで 定数を export する場合は、`as const satisfies` を設定しておくと便利です。

```ts
export const myName = "田中" as const satisfies string;

export const foodList = {
  ramen: "ラーメン",
  udon: "うどん",
  soba: "そば"
} as const satisfies {
  [key: string]: string
};
```

# `as const satisfies` の挙動

`as const satisfies` を使うことで、次の2つのメリットが得られます。

1. `satisfies` 演算子: 型がマッチするかどうかをチェックできる
2. `as const`: 値が widening （拡大）しない 

個別の機能の説明の前に、まずは `as const satisfies` のコードの挙動を確認してみましょう。

▼ `my-option.ts`

```ts
type MyOption = {
  foo: string;
  bar: number;
  baz: {
    qux: number;
  };
};

export const myOption = {
  foo: "foo",
  bar: 2,
  baz: {
    qux: 3,
  },
} as const satisfies MyOption;
```

`myOption` オブジェクトが `MyOption` 型にマッチしない場合、エラーが起こります。

```ts
export const myOption = {
  // ❌エラー
  foo: 120,
  bar: 2,
  baz: {
    // ❌エラー
    qux: "HELLO",
  },
} as const satisfies MyOption;
```

`myOption` オブジェクトの各プロパティは、「widening」しません（widening については後ほど解説します）。

とくに、 import 先で `myOption` オブジェクトを使いたい時、 widening しないことで型が安全になります。

![](/images/as-const-satisfies/as-const-satisfies.png)

# `satisfies` とは

`satisfies` とは、TypeScript 4.9 で導入された新しい演算子です。次のように定義したとき、`colorList` オブジェクトが、 `ColorList` 型にマッチするかどうかをチェックできます。

```ts
type ColorList = {
  [key in "red" | "blue" | "green"]: unknown;
};

const colorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  blue: "#0000ff"
} satisfies ColorList;
```

`ColorList` 型の `[key in "red" | "blue" | "green"]` とは、 オブジェクトのキーが `red` か `blue` か `green` のいずれかという意味です。オブジェクトの値が `unknown` なので、次のことを表現した型になります。

- オブジェクトのキーが  `red` か `blue` か `green` のいずれか
- オブジェクトの値は何でもいい

`colorList` オブジェクトが `ColorList` 型にマッチしない場合、エラーが起こります。

```ts
const colorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  // ❌エラー
  yellow: "#0000ff"
} satisfies ColorList;
```

▼ `yellow` キーが `ColorList` 型にマッチしない旨のエラー

![](/images/as-const-satisfies/satisfies-error.png)

## `satisfies` を使うと、型推論結果が保持される

`satisfies` の特徴は、**型チェックが行われつつも、型推論結果が保持されることです**。 「satisfies」（サティスファイズ）は「（条件などを）満たす、確信させる」といった意味があります。

次の `colorList` オブジェクトにおいて、`green` は `ColorList` の型の一部であることがチェックされ、かつ `number[]` 型と推論されます。配列なので、配列用メソッド `map()` が使えます。

▼ `green` は配列なので、配列用のメソッドが使える

```ts
const colorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  blue: "#0000ff"
} satisfies ColorList;

// 配列のメソッドを実行
colorList.green.map(value => value * 2);
```


## 型注釈では型推論結果が失われる

`satisfies ColorList` は、 `colorList` オブジェクトに型注釈 `: ColorList` をつけることと、何が違うのでしょうか？ 動作の違いを確認してみましょう。

```ts
const colorList: ColorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  blue: "#0000ff"
};
```

この場合も `colorList` オブジェクトが `ColorList` 型にマッチしない場合、エラーが起こります。

```ts
const colorList: ColorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  // ❌エラー
  yellow: "#0000ff"
};
```

**`satisfies` と異なるのは推論結果です**。型注釈を設定する場合、当然ですが型の推論結果は失われ、`colorList` オブジェクトの型情報は `ColorList` 型になります。

そのため `green` プロパティは `unknown` となり配列用の関数が使えません。開発者が `green` が配列であることを明らかにわかっていても、です。

```ts
const colorList: ColorList = {
  red: "#ff0000",
  green: [0, 255, 0],
  blue: "#0000ff"
};

// ❌エラー
// green は unknown なので、 map() は使えない
colorList.green.map(value => value * 2);
```

:::details 補足
`type ColorList = { red: string,  green: [number, number, number],  blue: string };` と表現すれば `green` で正しく `map` が使えます。しかし、`purple` や `pink` など、新しくキーを追加するたびにそれに対応する値の型を追加する必要があります。 「オブジェクトの値はなんでもいいので、キーの種類だけを制限したい」という当初の目的からは外れてしまいますし、大して詳細な情報を持たない型注釈と、詳細な情報を持つ値を二重で管理するのも手間です。
:::

「型のチェックをしたいが、推論結果は保持したい」というケースには、型注釈よりも `satisfies` が有効です。

# `as const` とは

`satisfies` と一緒に使うと強力なのが `as const` です。

`as const` とは、次の効果があります。

- 文字列・数値・真偽値などのリテラル型を widening しない
- オブジェクト内のすべてのプロパティが readonly になる
- 配列リテラルの推論結果がタプル型になる
- テンプレート文字列リテラル型の推論結果が widening しない

*※ 参考: [プロを目指す人のためのTypeScript入門 安全なコードの書き方から高度な型の使い方まで｜技術評論社](https://gihyo.jp/book/2022/978-4-297-12747-3)*

## widening とは

次の `myString` は、リテラル型の「`"鈴木"`型」に推論されます。

```ts
const myString = "鈴木";
// "鈴木"型
```

`let` で変数を宣言すると、`myString2` の型は、リテラル型の `"鈴木"`型ではなく、より広い `string` になります。なぜなら、 `myString2` は別の文字列を割り当て可能なため、`"鈴木"`しか入れられない`"鈴木"`型は合わないためです。

```ts
let myString2 = "鈴木";
// string型
```

このように、より型が大きくなる挙動のことを widening と呼びます。

![](/images/as-const-satisfies/string-widening.png)


オブジェクトの場合、各プロパティが widening し、`{ age: number,  name: string }` と推論されます。 `{ age: 18,  name: "鈴木" }` とは推論されません。オブジェクトの各プロパティが widening するのは、各プロパティが書き換え可能なことが原因です。

```ts
const myObject = {
  age: 18,
  name: "鈴木",
};

// 推論結果
// {
//  age: number,
//  name: string
//}
```

もちろん、配列も widening します。次の例では、 `myArray` は `string[]` 型になります。`[string, string, string]` や `["a", "b", "c"]` とは推論されません。

```ts
const myArray = ["a", "b", "c"];
// 推論結果はstring[]
```

## import・export での widening 

1つのモジュール内での話あれば気をつければ済む話でしょう。問題は、import・export した際です。

他の開発者が `myObject` を import してロジックを書いたとします。

```ts
export const myObject = {
  age: 18,
  name: "鈴木",
};
```

```ts
import { myObject } from "./my-object";

// "鈴木"型ではなく、string型
const myName = myObject.name;
```

`myObject.name`は `"鈴木"` ではなく、 `string` 型になってしまいましたが、使用者側は気づきづらいでしょう。

## widening を `as const` で防ぐ

widening は、`as const` で防げます。 

オブジェクトにおける widening の抑制例は次のとおりです。

▼ export 側

```ts
export const myObject = {
  age: 18,
  name: "鈴木",
} as const;
```

▼ import 側

```ts
import { myObject } from "./my-object";

// "鈴木"型
const myName = myObject.name;
```

文字列や数値といったプリミティブ型も、 `as const` で widening を防げます。


▼ export 側

```ts
// as const あり
export const foo = "田中";
// as const なし
export const bar = "吉田" as const;
```

▼ import 側

```ts
import { foo, bar } from "./sub";

// string型
const foo2 = foo;
// "吉田"型
const bar2 = bar;
```

基本的に readonly な値を export する場合は、`as const` をつけておくほうが安全です。筆者のプロジェクトでは、すべてそのようにしています。

# `satisfies` と `as const` を組み合わせる

本記事のキモです。

`satisfies` と `as const` を組み合わせると、2つのメリットを組み合わせられます。

1. `satisfies 型`: 型がマッチするかどうかをチェックできる
2. `as const`: 値が widening しない

次の `MyOption` を使って考えてみましょう。

```ts
type MyOption = {
  foo: string;
  bar: number;
  baz: {
    qux: number;
  };
};
```

`MyOption` 型を満たす `myOption` オブジェクトを作って、 export したいとします。



```ts
export const myOption = {
  foo: "foo",
  bar: 2,
  baz: {
    qux: 3,
  },
};
```

## `satisfies` なし、 `as const` なしの場合

`myOption` は何一つ型安全ではありません。大規模なプロジェクトになればなるほど、死者が増えるでしょう。

```ts
export const myOption = {
  foo: "foo",
  bar: 2,
  baz: {
    qux: 3,
  },
};
```

## `satisfies` あり、 `as const` なしの場合

型のチェックはできます。

```ts
export const myOption = {
  foo: "foo",
  // 型エラーになる
  bar: "HELLO",
  baz: {
    qux: 3,
  },
} satisfies MyOption;
```

しかし、 `myOption` オブジェクトは widening してしまいます。

▼ `myOption` オブジェクトの推論結果

```ts
{
  foo: string,
  bar: number,
  baz: {
    qux: number,
  },
}
```

他の開発者が `myOption` オブジェクトを使う時、型が widening していることに気づきづらいです。

## `satisfies` なし、 `as const` ありの場合

widening は防げますが、型チェックができなくなります。

```ts
export const myOption = {
  foo: "foo",
  // 型エラーにならない
  bar: "HELLO",
  baz: {
    qux: 3,
  },
} as const
```

▼ `myOption` の推論結果

```ts
{
  foo: "foo",
  bar: "HELLO",
  baz: {
    qux: 3,
  },
}
```

## `satisfies` あり、 `as const` ありの場合

型チェックが可能であり、かつ widening も防げます💐

```ts
export const myOption = {
  foo: "foo",
  // 型エラーになる
  bar: "HELLO",
  baz: {
    qux: 3,
  },
} as const
```

▼ `myOption` を正しく修正する

```ts
export const myOption = {
  foo: "foo",
  bar: 2,
  baz: {
    qux: 3,
  },
} as const satisfies MyOption;
```

▼ `myOption` の推論結果

```ts
{
  foo: "foo",
  bar: 2,
  baz: {
    qux: 3,
  },
}
```

▼ IDE での推論結果の確認

![](/images/as-const-satisfies/as-const-satisfies.png)


各コードのデモは次の URL で確認できます。
https://tsplay.dev/mbK3BW


# 配列と `as const satisfies` 

配列を使う際も、`as const satisfies` は便利です。

たとえば、次の `myArray` を `string[]` で型注釈したとき、`typeof myArray[number]` の型（任意のインデックスの要素の型）は `string` にしかなりません。

```ts
const myArray: readonly string[] = ["りんご", "みかん", "ぶどう"];
type MyElement = typeof myArray[number];
// string型 
```

`as const satisfies` を使うと、`typeof myArray[number]` の型（任意のインデックスの要素の型）は `"りんご" | "みかん" | "ぶどう"` になります。もちろん、`myArray` に文字列以外の要素を追加するとエラーになります。

```ts
const myArray = ["りんご", "みかん", "ぶどう"] as const satisfies readonly string[];
type MyElement = typeof myArray[number];
// "りんご" | "みかん" | "ぶどう" 型
```

# プリミティブ型と `as const satisfies`

文字列・数値といったプリミティブ型の場合にも、`as const satisfies` は便利です。

readonly の定数を作りたいが、型は `string` や `number` に制限しておきたいケースに使えます。前述のように export したプリミティブ型は widening しますので、 `as const satisfies` で型を制限しておくと安全です。

```ts
// myString は string に制限されつつつ、 'バナナ'型 に推論される
export const myString = 'バナナ' as const satisfies string;

// myNumber は number に制限されつつつ、 18型 に推論される
export const myVersion = 18 as const satisfies number;

// isDevelopMode は boolean に制限されつつつ、 true型 に推論される
export const isDevelopMode = true as const satisfies boolean;
```

# `as const satisfies` の活用例

`as const satisfies` をどのような場面で活用できるか、いくつか例を挙げます。


▼ アプリケーションのバージョンの定数

```ts
export const appVersion = "1.0.2" as const satisfies `${number}.${number}.${number}`;
```

▼ 元号のリスト

```ts
export const eraNames = ["昭和", "平成", "令和"] as const satisfies readonly string[];
```

▼ URLのリスト

```ts
export const urlList = {
  apple: "https://www.apple.com/jp/",
  google: "https://www.google.com/",
  yahoo: "https://www.yahoo.co.jp/",
} as const satisfies {
  // 値は https:// で始まるURLに限定する
  [key: string]: `https://${string}`
};
```

▼ ステータスのリスト

```ts
export const statusList = [
  {
    status: "processing",
    title: "作業中"
  },
  {
    status: "cancel",
    title: "キャンセル"
  },
  {
    status: "completed",
    title: "完了"
  },
] as const satisfies readonly {
  status: string,
  title: string
}[]
```

各コードは次の URL で試せます。

https://tsplay.dev/mqeyQm

# `as const satisfies` は人類が本当に求めていたもの

筆者は `as const` なオブジェクトを export することがよくありますが、推論結果を保ったまま型で縛りたいとずっと思っていました。TypeScript 4.9 で使えるようになった `satisfies` の登場により、`as const satisfies` の指定ができるようになったのは嬉しい限り。定数を早速置き換えています。

プロジェクトで `as const` なオブジェクトを export する場合は、積極的に使いましょう。

## 参考文献

- [Announcing TypeScript 4\.9 \- TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-4-9/)
- [Typescript’s new ‘satisfies’ operator \| by Cefn Hoile \| Medium](https://medium.com/@cefn/typescript-satisfies-6ba52e74cb2f)
- [プロを目指す人のためのTypeScript入門 安全なコードの書き方から高度な型の使い方まで｜技術評論社](https://gihyo.jp/book/2022/978-4-297-12747-3)
- [O'Reilly Japan \- プログラミングTypeScript](https://www.oreilly.co.jp/books/9784873119045/)
