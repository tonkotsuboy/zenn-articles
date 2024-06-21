---
title: "TypeScript 5.5で型述語を推論できて最高。配列のfilterも型安全に"
emoji: "🙂‍↕️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "typescript", "javascript" ]
published: true
publication_name: ubie_dev
---

2024/6/20
TypeScript 5.5が正式リリースしたので追記しました。

---

TypeScriptの次バージョン5.5で、開発者が長い間求めていた機能がついに実現されました。

従来のTypeScript 5.4以前では、ユーザー定義型ガードを使う際には型述語（用語は後ほど解説します）の記述が必要です。

▼ TypeScript 5.4

```ts
function isNumber(value: number | string): value is number {
  return typeof value === 'number';
}
```

2024年6月20日にリリースされたTypeScript 5.5では、関数の実体から型述語の型推論（infer type predicates）が可能になります。すなわち、次のようなコードが可能です。

▼ TypeScript 5.5

```ts
function isNumber(value: number | string) {
  return typeof value === 'number';
}
```

配列の`filter`メソッドで型を絞り込む際にも、型述語の型推論が可能となります。たとえば、次のようなコードのように`x is S`の記述をせずとも`filter`メソッドで型が正しく推論されます。

▼ TypeScript 5.5

```ts
const result = [12, null, 24, undefined, 48]
  .filter((value) => value != null);
//    resultはnumber[]に推論される
```

本記事では、 従来の型述語の危険性とTypeScript 5.5における型述語の型推論について具体的なコードを交えながら詳しく解説します。コードを動かせるプレイグラウンドのリンクも用意してあるので、ぜひ手を動かしてみて動作をご確認ください。

# これまでの型述語の危険性

従来、TypeScriptには、ユーザー定義型カード（User-defined type guard）を使って型を絞り込むことができました。たとえば、次のような関数を定義することで`value`が`number`であることをTypeScriptに伝えられます。 返り値の`value is number`の箇所は型述語（type predicate）と呼ばれます。

```ts
function isNumber(value: number | string): value is number {
  return typeof value === 'number';
}
```

`isNumber`関数を使うと次のように値の絞り込みができます。

```ts
function isNumber(value: number | string): value is number {
  return typeof value === 'number';
}

function main(value: number | string) {
  if (isNumber(value)) {
    // valueは数値型に絞り込まれる
    value.toFixed(2);
  }
}
```

しかし、型述語には危険性があります。`value is number`の箇所はユーザーが自身で定義しているものであり、関数本体の実装と一致しなくてもコンパイルエラーにならないのです。

次の例を見てみましょう。関数本体では`typeof value === 'string'`と`value`を文字列判定していて、型述語としては`value is number`となっているのですが、コンパイルエラーになりません。

```ts
function isNumber(value: number | string): value is number {
  // valueを文字列だと判定しているが、エラーにならない
  return typeof value === "string";
}
```

`isNumber`関数を使ったコードは型安全ではなく、ランタイムエラーを引き起こす可能性があります。

```ts
function isNumber(value: number | string): value is number {
  return typeof value === "string";
}

function main(value: number | string) {
  // 👎 ランタイムエラーになるまで気づけない
  if (isNumber(value)) {
    value.toFixed(2);
  }
}

main("豚骨きゅうり");
```

▼ 確認用Playground
https://www.typescriptlang.org/play?ts=5.4.2#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyJhZ76IA+iKU+MYA5gJS1lKuZCjoMCiAN4AoRIny4oIfEigBPAA644wRIKqIAvMcQAidpx6mA3NIC+06aEiwEiTKS4kKVWvRwSrBZcfFKyyLqEqBgBRPq4vLxhcnLxAHRQcABiMAAeuAAmhABMvLZyDg7SAPTViIC8G4ByO4iAlwyAzwyA-QyAJQyABwyAFQwtgD8MgNYMgFYMgNEMgH4MgOYMgOg2gKYMgIoMo4AiDNIeXqaAWjGAFVmAsgyAoQyAYgyAUQymZdJAA

![TS Playgroundのエラー](/images/ts-infer-type-predicates/run-result.png)
*TS PlaygroundでRunを実行したときにエラーが出ている様子*

# TypeScript 5.5から関数本体の実装から型を推論してくれるようになった

TypeScript 5.5からは型述語（`x is S`）の記述をすることなく、関数の本体から型述語が推論されるようになります。型述語の記述をしていない `isNumber`関数でも正しくタイプガードが行われています。

▼ TypeScript 5.5

```ts
function isNumber(value: number | string) {
  return typeof value === 'number';
}

function main(value: number | string) {
  if (isNumber(value)) {
    // valueは数値型に絞り込まれる
    value.toFixed(2);
  }
}
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyLgDWYcA7mAJSIDeAUIovrigh8SKAE8ADrjjBEZSrkQBeFYgDkYLHnxqA3NwC+iXsdCRYCRJlIwwJClVoMmrDjz4xZhVBhwF7CmxuJnwA9KFyDriA9gyADqaAJAqA0eqA1gyAer6AUQyAPfGAfgyAMQyA0QwhkQoAdFBwAGIwAB64ACaEAExs+nxhEQB6APwmBoZAA

意図しない型の絞り込みを行っていた場合、コンパイルエラーとして気づけます。

▼ TypeScript 5.5

```ts
function isNumber(value: number | string) {
  return typeof value === "string";
}

function main(value: number | string) {
  // 👍valueがstringに絞り込まれていることをコンパイルエラーとして気づける
  if (isNumber(value)) {
    value.toFixed(2);
  }
}

main("豚骨きゅうり");
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyJhZ76IA+iKU+MYA5gJSIBvAFCJE+XFBD4kUAJ4AHXHGCIylXIgC82xACJ2nHroDcQgL5ChoSLASJMpLiQpVa9HARZsOXPoJGIAPSBiIC8G4CyO2pUgDIMBr6A1gyAer6AUQyAPfGAfgyAMQyAZgyAIgyA0QyAygyAFgyASQyAzQyAzwyAiwyAJQyA1wyAFQyAlwyAPwzFgOoM2YDoNoCmDICKDPkBMCqEqBgeRFG4vPzCoqLTAHRQcABiMAAeuAAmhABMvKaiFhZCDk66gFoxgBVZgLIMgKEMgGIMybpHQA

## 型述語の推論結果の確認

TS PlaygroundやVSCodeでユーザー定義型ガードの関数を確認すると、返り値として型述語が推論されていることがわかります。

![型述語の推論結果の確認](/images/ts-infer-type-predicates/infer-type.png)

TypeScript 5.4以前では、型述語を記述しない場合の返り値は`boolean`型と推論されていました。

# 配列の`filter`で型を絞り込むのがより型安全になる

本記法が便利なのは配列の `filter` メソッドで型を絞り込むときです。

`filter` メソッドで`null`や`undefined`を取り除く処理というのは頻出します。 たとえば、数値と `null` と `undefined` が混在する配列から`null` と `undefined` を取り除いた配列を作るとします。次のようなコードが考えられるでしょう。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value != null);
```

開発者は、`result`の型は`number[]`に推論されることを期待するでしょう。

## 従来の課題

TypeScript 5.4以前では、`result` は `number[]`ではなく `(number | null | undefined)[]`にしか推論されませんでした。`filter`関数で明らかに `null` と `undefined` を除外しているにも関わらず、です。

▼ TypeScript 5.4

```ts
const result = [12, null, 24, undefined, 48]
                  .filter((value) => value != null);
// resultは (number | null | undefined)[]
```

▼ 確認用Playground

https://www.typescriptlang.org/play/?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugHQt0COAAoRANwCGqZAgCUMDAD4YUmQhgBCLKVSo5AbgBQAehMwLMAHoB+I0A

`filter`関数で絞り込まれる型を明示的に表現するため、型述語を使って次のように記述する方法が取られています。筆者もよく書くコードです。

▼ TypeScript 5.4

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value != null);
// resultは number[]
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugHQt0COAAoRANwCGqZAgCUALhhSZCGMwglkAWwBGwmBgB8y6bJgBCLKVSo5AbgBQAemcx3MAHoB+IA


しかし、型述語はあくまでユーザー定義のものであり、誤った判定をしたとしてもコンパイルエラーになりません。

たとえば次の判定では`value`が`null`のときに`true`を返してしまっていますが、`value is number` により `result` は `number[]` に推論されてしまいます。`number`用のメソッド `toFixed()`を使い`result[0].toFixed(2)` と記述してしまうと、ランタイムエラーになるまで気づけません。

▼ TypeScript 5.4

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value): value is number => value === undefined);

// 👎コンパイルエラーにならず、ランタイムエラーになるまで気づけない
result[0].toFixed(2);
```

▼ 確認用Playground
https://www.typescriptlang.org/play?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEASgBcMJSoQxmEEsgC2AI2kwMAPm3LVljFloMWbOmoDcfPgHofMQF4NwDkdwGaGQGeGQEWGQBKGQGuGQAqGQEuGQB+GQGsGQCsGQEiGQC0GQEAGBLDAfoYowAOGRNS0wGiGQD8GQHMGQHQbQFMGQEUGNMARBj5EFHRsAAYeMSgQADFmAA92GXxPIA



## TypeScript 5.5からの改善

TypeScript 5.5から、関数本体の実装から型述語を推論してくれるようになったので`x is S` の記法が不要になります。次のコードで`result` は `number[]`となります。

▼ TypeScript 5.5

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value != null);
// resultはnumber[]
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugHQt0COAAoRANwCGqZAgCUMDAD4YUmQhgBCLKVSo5AbgBQAehMwLMAHoB+IA

誤って`value`が`null`のときに`true`を返すようなコードを書いた場合、`result` は `(number | null)[]` に推論されます。`number`用のメソッド `toFixed()`を使い、`result[0].toFixed(2)` と記述したとき、ランタイムエラーではなくコンパイルエラーとして気づけます。

▼ TypeScript 5.5

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value === undefined);

// コンパイルエラーになる👍
result[0].toFixed(2);
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEAShgYAfDCUqEmjFloMWbOmoDcfPgHpbMQM0MgZ4ZAiwyAShkDXDIAqGQJcMgH4ZAawZAKwZAaIZAXg3AWR2+RBR0bAAGHjEoEAAxZgAPdhl8KyA

# `instanceof`も使える

`instanceof`を使ったユーザー定義型ガードも、関数本体の実装から型述語が推論されます。

▼ TypeScript 5.5

```ts
class Foo {}

class Bar {}

const result = [new Foo(), new Bar()].filter(x => x instanceof Foo);
//    ^?
//    Foo[] に推論される
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/MYGwhgzhAEBiD29oG8C+AodpIwEJgCcUMt4A7CAF2gIFMIBXEagXmgG0zaB3ORACgCUAGmhde+AkIC6AOgBmAS2a0pAD2gsAfNA2KKlMGWC148vvEEBudAHpb0R9AB6AfjsOnCeO2nRA1gyAFcaAa1GAqgyAMQyA0QxAA



#  タグ付きユニオンによる絞り込み（プロパティによる絞り込み）も可能 

タグ付きユニオンを使ったユーザー定義型ガードの結果も推論できます。具体的には、次のようなコードで `isA`関数の返り値が `x is A` と推論されます。これをずっとやりたかった・・・！

▼ TypeScript 5.5

```ts
type A = { type: "A"; a: number };
type B = { type: "B"; b: number };

function isA(x: A | B) {
  return x.type === "A";
}

function check(foo: A | B) {
  if (isA(foo)) {
    // OK！
    console.log(foo.a);
  }
}
```

▼ 確認用Playground
https://www.typescriptlang.org/play/?#code/C4TwDgpgBAglC8UDeVSQFxQEQywbigENMA7AVwFsAjCAJygF88AoNaAIQWVXAky3b4oVUpRr0mzZgDMyJAMbAAlgHsSUJQGcYACgAemOAB8o7AJTJmUKLQjAytdXoB0bBPEQ58zBlNkLlNSh5AAsIeQBrHWkVFUMoE3NLayVpKB0tXRiVMwskK2soAHoiqAB5AGlAQH+C63k1TRUAGwhnJpUAc2jY50IzFmtfXyA

# `filter`の型の絞り込みが型安全になって最高

配列の`filter`メソッドと型述語を使う度に、そのコードの危険性に震えていました。TypeScript 5.5での型述語の推論のおかげでその危険性がなくなるので一安心です。

https://twitter.com/tonkotsuboy_com/status/1769994147291889669

TypeScript 5.5は2024年6月20日にリリースされました。

https://devblogs.microsoft.com/typescript/announcing-typescript-5-5/

# 参考記事

https://github.com/microsoft/TypeScript/pull/57465

https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates

https://www.totaltypescript.com/type-predicate-inference
