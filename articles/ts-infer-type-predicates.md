---
title: "TypeScript5.5で型述語を推論できて最高。配列のfilterも型安全に"
emoji: "🙂‍↕️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "javascript"]
published: true
publication_name: ubie_dev
---

結論を先に言うと、6月リリース予定のTypeScript 5.5で次のようなコードが型安全になります。

従来: TypeScript 5.4以前 

```ts
function isNumber(value: number | string): value is number {
  return typeof value === 'number';
}

const result = [12, null, 24, undefined, 48]
  .filter((value): value is number => value != null);
```

今後: TypeScript 5.5以降

```ts
function isNumber(value: number | string) {
  return typeof value === 'number';
}

const result = [12, null, 24, undefined, 48]
  .filter((value) => value != null);
```

TypeScript 5.5の実際のコードを交えながら、本記事で詳しく解説します。コードを動かせるプレイグラウンドのリンクも用意してあるので、ぜひ手を動かしてみて動作をご確認ください。

# これまでの型述語の危険性

従来、TypeScriptには、ユーザー定義型カード（User-defined type guard）を使って型を絞り込むことができました。たとえば、次のような関数を定義することで、`value`が`number`であることをTypeScriptに伝えることができます。 返り値の`value is number`の箇所は、型述語（type predicate）と呼ばれます。

```ts
function isNumber(value: number | string): value is number {
  return typeof value === 'number';
}
```

`isNumber`関数を使うと、次のように値の絞り込みができます。

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

しかし、型述語には危険性があります。`value is number`の箇所はユーザーが自身で定義しているものであり、関数本体の実装と一致しなくてもコンパイルエラーにならないのです。たとえば、次の例では関数本体では`typeof value === 'string'`と`value`を文字列判定していて、型述語としては`value is number`となっているのですが、コンパイルエラーになりません。

```ts
function isNumber(value: number | string): value is number {
  // valueをnullだと判定しているが、エラーにならない
  return typeof value === null;
}
```

型安全ではなく、`isNumber`関数を使ったコードは、ランタイムエラーを引き起こす可能性があります。


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

https://www.typescriptlang.org/play?ts=5.4.2#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyJhZ76IA+iKU+MYA5gJS1lKuZCjoMCiAN4AoRIny4oIfEigBPAA644wRIKqIAvMcQAidpx6mA3NIC+06aEiwEiTKS4kKVWvRwSrBZcfFKyyLqEqBgBRPq4vLxhcnLxAHRQcABiMAAeuAAmhABMvLZyDg7SAPTViIC8G4ByO4iAlwyAzwyA-QyAJQyABwyAFQwtgD8MgNYMgFYMgNEMgH4MgOYMgOg2gKYMgIoMo4AiDNIeXqaAWjGAFVmAsgyAoQyAYgyAUQymZdJAA



![TS Playgroundのエラー](/images/ts-infer-type-predicates/run-result.png)
*TS PlaygroundでRunを実行したときにエラーが出ている様子*

# TypeScript 5.5から、関数本体の実装から型を推論してくれるようになった

2024年6月リリース予定のTypeScript 5.5から、`x is S`の記述をすることなく、関数の本体から型述語が推論されるようになります。`x is S`の記述をしていない `isNumber`関数でも、正しくタイプガードが行われています。

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

https://www.typescriptlang.org/play?ts=5.5.0-dev.20240320#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyLgDWYcA7mAJSIDeAUIovrigh8SKAE8ADrjjBEZSrkQBeFYgDkYLHnxqA3NwC+iXsdCRYCRJlIwwJClVoMmrDjz4xZhVBhwF7CmxuJnwA9KFyDriA9gyADqaAJAqA0eqA1gyAer6AUQyAPfGAfgyAMQyA0QwhkQoAdFBwAGIwAB64ACaEAExs+nxhEQB6APwmBoZAA


意図しない型の絞り込みを行っていた場合、コンパイルエラーとして気づけます。

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

https://www.typescriptlang.org/play?ts=5.5.0-dev.20240320#code/GYVwdgxgLglg9mABDAzgORAWwEYFMBOAFAG4CGANiLgFyJhZ76IA+iKU+MYA5gJSIBvAFCJE+XFBD4kUAJ4AHXHGCIylXIgC82xACJ2nHroDcQgL5ChoSLASJMpLiQpVa9HARZsOXPoJGIAPSBiIC8G4CyO2pUgDIMBr6A1gyAer6AUQyAPfGAfgyAMQyAZgyAIgyA0QyAygyAFgyASQyAzQyAzwyAiwyAJQyA1wyAFQyAlwyAPwzFgOoM2YDoNoCmDICKDPkBMCqEqBgeRFG4vPzCoqLTAHRQcABiMAAeuAAmhABMvKaiFhZCDk66gFoxgBVZgLIMgKEMgGIMybpHQA


## 型述語の推論結果の確認

TS PlaygroundやVSCodeでユーザー定義型ガードの関数を確認すると、返り値として型述語が推論されていることがわかります。

![型述語の推論結果の確認](/images/ts-infer-type-predicates/infer-type.png)

TypeScript 5.4以前では、型述語を記述しない場合は 返り値は`boolean`型と推論されていました。


# 配列の filter で型を絞り込むのがより型安全になる

本記法が便利なのは、配列の `filter` メソッドで型を絞り込むときです。

`filter` メソッドで`null`や`undefined`を取り除く処理というのは頻出します。 たとえば、数値と `null` と `undefined` が混在する配列から、`null` と `undefined` を取り除いた配列を作りたいとします。次のようなコードが考えられるでしょう。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value != null);
```

## 従来の課題

TypeScript 5.4以前では、上記コードで `result` は `number[]`ではなく、 `(number | null | undefined)[]`にしか推論されませんでした。`filter`関数で明らかに `null` と `undefined` を除外しているにも関わらず、です。

https://www.typescriptlang.org/play?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEAShgYAfDCUqEMAIRZSqVGoDcfIA

`filter`関数で絞り込まれる型を明示的に表現するため、型述語を使って次のように記述する方法がよく取られています。筆者もよく書くコードです。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value): value is number => value != null);
```

https://www.typescriptlang.org/play?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEASgBcMJSoQxmEEsgC2AI2kwMAPm3LVMAIRZSqVGoDcQA


しかし、型述語はあくまでユーザー定義のものであり、誤った判定をしたとしてもコンパイルエラーになりません。たとえば次の判定では、`value`が`null`のときに`true`を返してしまっていますが、`value is number` により `result` は `number[]` に推論されてしまいます。よって、`number`用のメソッド `toFixed()`を使い、`result[0].toFixed(2)` と記述してしまうと、ランタイムエラーになるまで気づけません。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value): value is number => value === undefined);

// 👎コンパイルエラーにならず、ランタイムエラーになるまで気づけない
result[0].toFixed(2);
```

https://www.typescriptlang.org/play?ts=5.4.2#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEASgBcMJSoQxmEEsgC2AI2kwMAPm3LVljFloMWbOmoDcfPgHofMQF4NwDkdwGaGQGeGQEWGQBKGQGuGQAqGQEuGQB+GQGsGQCsGQEiGQC0GQEAGBLDAfoYowAOGRNS0wGiGQD8GQHMGQHQbQFMGQEUGNMARBj5EFHRsAAYeMSgQADFmAA92GXxPIA

## TypeScript 5.5からの改善

TypeScript 5.5から、関数本体の実装から型述語を推論してくれるようになったので、`x is S` の記法が不要になります。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value != null);
```

https://www.typescriptlang.org/play?ts=5.5.0-dev.20240320#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEAShgYAfDCUqEMAIRZSqVGoDcfPohTorAegcwAegH4gA



誤って、`value`が`null`のときに`true`を返すようなコードを書いた場合、`result` は `(number | null)[]` に推論されます。よって、`number`用のメソッド `toFixed()`を使い、`result[0].toFixed(2)` と記述したとき、ランタイムエラーであなくコンパイルエラーとして気づけます。

```ts
const result = [12, null, 24, undefined, 48]
                .filter((value) => value === undefined);

// コンパイルエラーになる👍
result[0].toFixed(2);
```

https://www.typescriptlang.org/play?ts=5.5.0-dev.20240320#code/MYewdgzgLgBATgUwgVwDawLwwNoEYBMANDGGqsfgCzHJgAmCAZgJZgJ3GUAcAugFAxBQ4SJgA6FugRwAFDIBuAQ1TIEAShgYAfDCUqEmjFloMWbOmoDcfPgHpbMQM0MgZ4ZAiwyAShkDXDIAqGQJcMgH4ZAawZAKwZAaIZAXg3AWR2+RBR0bAAGHjEoEAAxZgAPdhl8KyA

# `instanceof`も使える

`instanceof`を使ったユーザー定義型ガードも、関数本体の実装から型述語が推論されます。

```ts
class Foo {}
class Bar {}

const result = [new Foo(), new Bar()].filter(x => x instanceof Foo);
//    ^?
//    Foo[] に推論される
```

# プロパティによる絞り込みは現状だとできない

TypeScript 5.5-dev.20240320の段階では、プロパティによる絞り込みを行うユーザー定義型ガードの結果は推論してくれないようです。具体的には、次のようなコードで `isA`関数の返り値が `x is A` と推論されることはないようです。

```ts
type A = { type: "A"; a: number };
type B = { type: "B"; b: number };

function isA(x: A | B) {
  return x.type === "A";
}

let foo: any;

// doesn't work
if (isA(foo)) {
  foo;
// ^?
}
```

# `filter`の型の絞り込みが型安全になって最高

配列の`filter`メソッドと型述語を使う度に、そのコードの危険性に震えていました。TypeScript 5.5での型述語の推論（infer type predicates）のおかげでその危険性がなくなるので一安心です。

https://twitter.com/tonkotsuboy_com/status/1769994147291889669

TypeScript 5.5は、現状だと2024年6月18日のリリース予定です。楽しみに待ちましょう。

https://github.com/microsoft/TypeScript/issues/57475


# 参考記事


https://github.com/microsoft/TypeScript/pull/57465

https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates

https://www.totaltypescript.com/type-predicate-inference
