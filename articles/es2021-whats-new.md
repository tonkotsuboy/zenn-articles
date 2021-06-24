---
title: "正式仕様リリース！ JavaScriptの最新仕様ES2021で追加された新機能まとめ"
emoji: "💐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: true
---

JavaScriptの仕様はECMAScriptで、ECMAScript 2015（ES2015）、ECMAScript 2016（ES2016）...というように毎年進化を続けています。

2021年6月までの最新仕様はES2020でしたが、先日6月22日にES2021が正式仕様として承認されました。

- [Ecma International approves new standards \- Ecma International](https://www.ecma-international.org/news/ecma-international-approves-new-standards-4/)

ブラウザ対応も完了しており、全モダンブラウザ（Google Chrome・Firefox・Safari・Microsoft Edge）でES2021の全機能が使えます。

本記事では、ES2021すべての新機能をまとめて紹介します。

# 大きな数値を`_`区切りで書ける

| 構文         |
|:-----------|
| `数値_数値_数値` |

▼ 簡単な例

```js
100_000_000;  // 1億（100,000,000）
````

## 説明

`10000000`という数値を見たとき、それが「10,000,000」なのか「1,000,000」なのか、ぱっと見て区別をつけるのは難しいです。数字を`_`（アンダースコア）で区切り、数値を読みやすくできるようになります。

```js
100_000_000;  // 1億（100,000,000）
12_234; // 1万2234（12,234）
`${1_000_000}ドルの夜景`;  // 100万ドルの夜景
````

`_`で区切られていても、通常の数値として扱えるので計算など各種数値の操作が可能です。

```js
console.log(222_222 * 2);
// 結果: 444444
```

## 16進数の区切りでも入れられる

区切りは3桁ごとに入れる必要はなく、任意の箇所で入れられます。ただ、10進数の場合は3桁区切りにしておいたほうが他の開発者の混乱を招かないでしょう。

```js
// OKだが読みづらい
1_23_4;
```

区切りは数値であれば何でも使えます。16進数で色表現をする際の表現としても便利です。

```js
0xff_00_00; // 0xff0000
0xa0_b0_c0; // 0xa0b0c0
```

## 関連資料

- [tc39/proposal\-numeric\-separator](https://github.com/tc39/proposal-numeric-separator)
- [数値の区切り文字 - MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#:~:text=%E6%95%B0%E5%80%A4%E3%81%AE%E5%8C%BA%E5%88%87%E3%82%8A%E6%96%87%E5%AD%97)

# マッチした文字列をすべて置換できる`文字列.replaceAll()`

| 構文              | 戻り値       |
|:----------------|:----------|
| `文字列.replaceAll(文字列または正規表現, 文字列)` | `String` |

▼ 簡単な例

```js
"👺👺👺😈😈😈".replaceAll("😈", "🔥");
// 結果: "👺👺👺🔥🔥🔥"
```

## 説明

`replaceAll()`とは、引数にマッチした文字列をすべて置換できるメソッドです。

従来、文字列の置換用のメソッドとして、`文字列.replace()`メソッドがありました。`replace()`メソッドは、正規表現または文字列を引数に取り、マッチした文字列を置換するメソッドです。引数が文字列の場合、「**最初にマッチした文字列だけを**」置換します。すべての文字列を置換するためには、引数に正規表現を使う必要がありました。

`replaceAll()`は、`replace()`と異なり文字列の引数でもすべての文字列を置換できます。

## `文字列.replaceAll()`の挙動確認

「猫田猫男は猫好きだ」という文字列の、「猫」を「犬」に変換する例で考えてみましょう。文字列`"猫"`を引数にして`replace()`メソッドを使った場合、最初の「`"猫"`」しか置換されません。

```js
"猫田猫男は猫好きだ".replace("猫", "犬");
// 結果: "犬田猫男は猫好きだ"
```

すべての「猫」を「犬」に変換するためには、引数を正規表現にして`g`フラグを指定し、「`/猫/g`」とする必要があります。

```js
"猫田猫男は猫好きだ".replace(/猫/g, "犬");
// 結果: "犬田犬男は犬好きだ"
```

ES2021の`replaceAll()`を使えば、正規表現を使わずとも引数の文字列すべてを置換できます。よりシンプルに文字列の一括置換ができるようになったと言えるでしょう。

```js
"猫田猫男は猫好きだ".replaceAll("猫", "犬");
// 結果: 「"犬田犬男は犬好きだ"」
```

▼ 実行結果

![](https://storage.googleapis.com/zenn-user-upload/60296dd5b5e80e3071caaed9.png)

## 関連資料

- [tc39/proposal\-string\-replaceall](https://github.com/tc39/proposal-string-replaceall)
- [String\.prototype\.replaceAll\(\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll)

# 複数のPromiseのうち、最初のresolveを待つ`Promise.any`

| 構文              | 戻り値       |
|:----------------|:----------|
| `Promise.any(Promiseの配列)` | `Promise` |

▼ 簡単な例

```js
Promise.any([
  promise1, promise2, promise3
]).then(first => {
  // 3つのpromiseのうち、最初に解決したpromiseが出力される
  console.log(first)
});
```

## 説明

`Promise`では非同期処理を扱えますが、制作の現場では複数の非同期処理を同時に扱いたいことが多いでしょう。`Promise.any`とは、複数のPromiseのうち、どれか1つでもresolveしたらその時点で解決されるPromiseです。最初のresolveがあるまでは、rejectは無視されます。

## `Promise.any()`の挙動確認コード

次のコードで`Promise.any()`の挙動を確認してみましょう。3つのPromiseは、それぞれ次の挙動をします。

- `promise1`: 1秒後に「reject」されます
- `promise2`: 2秒後にresolveします
- `promise2`: 3秒後にresolveします

`promise1`は一番完了が早いですが、rejectされるため無視されます。`promise2`がresolveされるのが一番早いため、`Promise.any`では`promise2`をもってresolveします。

```js
// 1秒後に「reject」されるPromise
const promise1 = new Promise((resolve, reject) =>
  setTimeout(reject, 1000, "promise1")
);

// 2秒後にresolveするPromise
const promise2 = new Promise((resolve) =>
  setTimeout(resolve, 2000, "promise2")
);

// 3秒後にresolveするPromise
const promise3 = new Promise((resolve) =>
  setTimeout(resolve, 3000, "promise3")
);

Promise.any([promise1, promise2, promise3]).then((resolve) => {
  console.log(resolve);
});
```

▼ 実行結果

![](https://storage.googleapis.com/zenn-user-upload/55cd17d21a0efef0a4ab81d4.png)

resolveされるPromiseが1つもない場合は、`AggregateError`でrejectされます。

## `Promise.race()`との違い

`Promise.any()`に似た処理として、`Promise.race()`があります。

■ `Promise.any()`
引数のPromiseが最初に「**resolveした時点で**」終了する

■ `Promise.race()`
引数のPromiseが最初に「**終了（resolveまたはreject）した時点で**」終了する

▼ `Promise.race()`の実行結果

![](https://storage.googleapis.com/zenn-user-upload/f18a1e1318f2e9fe6f7fa919.png)

## 他のPromiseの複数処理 

ES2015で導入された`Promise.all()`、ES2020で導入された`Promise.allSettled()`など、`Promise`の複数処理はいくつかあります。この機会にそれぞれとの違いを確認しておくとよいでしょうw。

https://twitter.com/tonkotsuboy_com/status/1252519470523772929?conversation=none

## 関連資料

- [tc39/proposal\-promise\-any](https://github.com/tc39/proposal-promise-any)
- [Promise\.any\(\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)
[Promise\.race\(\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

# `??`や`||`や`&&`と`=`を組み合わせて使える演算子

| 構文        |
|:----------|
| `a ??= b` |
| `a ||= b` |
| `a &&= b` |

▼ 簡単な例

```js
let a = null;
a ??= "🐈";
console.log(a); // 結果： "🐈"

let b = "🐷";
b ??= "🐈";
console.log(b); // 結果： "🐷"
```

## 説明

`??=`演算子、`||=`演算子、`&&=`演算子とは、`??`・`||`・`&&`と`=`を組み合わせられる演算子です。

## `??=`演算子

`??=`演算子とは、`a`が`null`か`undefined`のときに、`a`に`b`を代入するための演算子です。

ES2020では、`??`演算子が実装されました。`a ?? b`という形で用い、`a`が`null`または`undefined`のときのみ`b`を返せます。`||`演算子は`a`がfalsyなもの（`0`や`""`や`false`）のときも`b`を返すのに対して、厳密に`null`や`undefined`を判定できるので、開発の現場では重宝する処理です。

`??=`演算子は、`??`に`=`を組み合わせたもの。次の挙動と同じ意味です。

```js
// aがnullかundefinedのときに、aにbを代入する
a ?? (a = b);
```

`const human = { name: "田中" }`というオブジェクトのプロパティを通して、`??=`の挙動を確認してみましょう。

`human.age ??= 18`で`age`プロパティに`18`の代入を試みています。`age`プロパティは`undefined`なので、`18`が代入されます。

`human.name ??= "鈴木"`では、`name`プロパティが`undefined`ではないので、`"鈴木"`は代入されません。

```js
const human = { name: "田中" };
human.age ??= 18;
// hunan.ageはundefinedなので18が代入される
human.name ??= "鈴木";
// hunan.ageはnullではないので何も代入されない
console.log(human);
// 結果: {name: "田中", age: 18}
```

▼ 実行結果

![](https://storage.googleapis.com/zenn-user-upload/8173c94a52f1f42572e5e309.png)

## `||=`演算子

`a`がfalsyなものの場合に、`a`に`b`を代入します。

```js
let a = 0;
a ||= "🐈";
console.log(a); // 結果： "🐈"

let b = "🐷";
b ||= "🐈";
console.log(b); // 結果： "🐷"
```

`||=`は、たとえばHTML要素の`innerHTML`のデフォルト値を設定するようなケースで使えます。次のコードでは、`.foo`要素の中身が空の場合に、`<p>値なし</p>`の要素を代入できます。

```js
document.querySelector(".foo").innerHTML ||= "<p>値なし</p>";
```

## `&&=`演算子

`a`がtruthyなものの場合に、`a`に`b`を代入します。

```js
let a = null;
a &&= "🐈";
console.log(a); // 結果： null

let b = "🐷";
b &&= "🐈";
console.log(b); // 結果： "🐈"
```

## 関連資料

- [tc39/proposal\-logical\-assignment](https://github.com/tc39/proposal-logical-assignment)
- [Null 合体代入 \(??=\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Logical_nullish_assignment)
- [論理和代入 \(\|\|=\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment)
- [論理積代入 \(&&=\) \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Logical_AND_assignment)

# 弱参照を作れるWeakRef

| 構文              | 戻り値       |
|:----------------|:----------|
| `new WeakRef(対象)` | `WeakRef` |

▼ 簡単な例

```js
const myObject = { name: "田中" };
// myObjectへの弱参照が作られる
const ref = new WeakRef(myObject);
```

## 説明

`WeakRef`は、JavaScriptにおいて弱参照を作れるオブジェクトです。参照はされているものの、参照先はガベージコレクションの対象となります。また、オブジェクトがガベージコレクトされた際、`FinalizationRegistry`オブジェクトによりコールバックを実行できるようになりました。ただし、ガベージコレクション周りの処理は複雑であり、可能であれば使用を避けることがよいとされています（「[tc39 / proposal\-weakrefs](https://github.com/tc39/proposal-weakrefs#:~:text=this%20proposal%20contains%20two%20advanced%20features%2C%20weakref%20and%20finalizationregistry.%20their%20correct%20use%20takes%20careful%20thought%2C%20and%20they%20are%20best%20avoided%20if%20possible.)」参照）。

WeakRefの詳しい内容は、[@uhyo](https://twitter.com/uhyo_)さんの解説記事がわかりやすいので、そちらを参照するとよいでしょう。

- [WeakRef: JavaScriptに弱参照がやってくる（ついでにFinalizationも） \- Qiita](https://qiita.com/uhyo/items/5dc97667ba90ce3941cd)

※ `FinalizationGroup`は`FinalizationRegistry`に読み替えてください

## 関連資料

- [tc39 / proposal\-weakrefs](https://github.com/tc39/proposal-weakrefs)
- [WeakRef \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/WeakRef)
- [FinalizationRegistry \- MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry)

# 対応環境

本記事で紹介したES2021の全機能は、各環境で使用可能です。

- Google Chrome
- Firefox
- Safari
- Microsoft Edge
- TypeScript
- Node.js

「[ECMAScript compatibility table](https://kangax.github.io/compat-table/es2016plus/)」で、細かい対応バージョンを確認できます。
※ 執筆時点では、TypeScriptの最新対応状況が反映されていないようです

# ES2021を使って便利に開発しよう

本記事では正式仕様としてリリースされたES2021の新機能を紹介しました。どれも開発をラクにしてくれるものばかりで、筆者も積極的に開発の現場で使っています。ECMAScriptは次のES2022に向けて仕様策定がすでに始まっています。top levelでの`await`やclass fieldなど、また便利な機能が入ってきそうです。新しい機能をキャッチアップし、楽しく開発していきましょう。

ES2021のLanguage Specificationは、こちらから確認できます。
- [ECMAScript® 2021 Language Specification](https://262.ecma-international.org/12.0/)

TwitterでもJavaScriptやCSSの最新情報を発信しています！
https://twitter.com/tonkotsuboy_com
