---
title: "正式仕様リリース！ JavaScriptの最新仕様ES2022で追加された全ての新機能"
emoji: "🌼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: true
---
JavaScriptの仕様はECMAScriptで、ECMAScript 2015（ES2015）、ECMAScript 2016（ES2016）...というように毎年進化を続けています。

2022年6月までの最新仕様はES2021でしたが、本日6月22日にES2022が正式仕様として承認されました。

- [Ecma International approves new standards \- Ecma International](https://www.ecma-international.org/news/ecma-international-approves-new-standards-5/)

ブラウザ対応も完了しており、全モダンブラウザ（Google Chrome・Firefox・Safari・Microsoft Edge）でES2022の全機能が使えます。

本記事では、ES2022すべての新機能を詳しく紹介します。

この記事を読むと、インスタンスの配列の最終要素取得には`at()`を使うように

# クラスフィールド宣言ができるようになった

ES2022では、これまでできなかったクラスフィールド宣言ができるようになりました。

```js
class Human {
  age = 18;
  static category = "animal"
}
```

## これまでJavaScriptでは、クラスフィールド宣言ができなかった

ES2015でJavaScriptにはクラス構文が導入されました。しかし、他の言語のクラス構文と異なり、**JavaScriptではクラスフィールド宣言ができませんでした**。フィールドを使いたいときは、次のように `construtor()`の中で宣言するしかありませんでした。

▼ 従来のフィールド

```js
class Human {
  constructor() {
    this.age = 18;
  }
}

const human = new Human();
console.log(human.age); // 18
```

※ クラスメソッドはES2015から宣言可能です

## ES2022でクラスフィールド宣言

ES2022からは、他言語と同じようにクラスフィールド宣言ができるようになりました。より直感的に、クラスの取り扱いができるようになります。

```js
class Human {
  age = 18;
}

const human = new Human();
console.log(human.age); // 18
```

## スタティックなクラスフィールド宣言

ES2022では、静的な（スタティックな）クラスフィールド宣言もできるようになりました。次のように記述すると、インスタンス化なしでクラスのフィールドにアクセスできます。

```js
class Human {
  static category = "animal"
}

console.log(Human.category);  // animal
```

## 関連資料

- [Class field declarations for JavaScript - tc39](https://github.com/tc39/proposal-class-fields)

# プライベートなフィールドとメソッドが使えるようになった

プライベートなフィールドやメソッドとは、クラスの外からアクセスできず、クラスの内側からしかアクセスできないものです。

```js
class MyClass {
  #foo = "ラーメン";
  
  #bar() {
    console.log("うどん")
  }
}
```

## 従来のフィールドやメソッドはすべてパブリックだった

ES2022以前では、クラス構文で作ったフィールドやメソッドは、すべて「パブリック」でした。「パブリック」とは、クラスの内部、外部のどちらからでもアクセスできることを指します。

次の例では、コンストラクターに渡した`name`を`name`フィールドに保持し、`hello()`メソッドで出力するプログラムです。次の箇所で、`name`フィールドにアクセスできていていることがわかります。

- クラスの内部（`hello()`メソッド内部）
- クラスの外側（`foo.name`の箇所）

▼ パブリックなフィールドの挙動の確認

```js
class MyClass {
  name;
  
  constructor(name) {
    this.name = name
  }

  hello() {
    console.log(`こんにちは${this.name}さん！`)
  }
}

const foo = new MyClass("田中");
foo.hello(); // 「こんにちは田中さん！」と出力される
console.log(foo.name);  // 「田中」と出力される
// nameの書き換えが可能
foo.name = "鈴木";
foo.hello(); // 「こんにちは鈴木さん！」と出力される
```

一度「`田中`」の文字列を渡したら、クラスの外から書き換えたくない場合、上記のようなパブリックなフィールドでは不適切になります。

## `#`を使ってプライベートなフィールドを宣言する

`#`を使うと、「プライベートな」プロパティを宣言できます。「プライベート」とは、クラスの内部からしかアクセスできないことを指します。クラスの外部からアクセスしようとするとエラーになります。

▼ プライベートなフィールドの確認

```js
class MyClass {
  // プライベートなフィールド
  #name;
  
  constructor(name) {
    this.#name = name
  }

  hello() {
    console.log(`こんにちは${this.#name}さん！`)
  }
}

const foo = new MyClass("田中");
foo.hello(); // 「こんにちは田中さん！」と出力される
```

上記の処理のあとに、次のコードを記述するとエラーになります。プライベートなフィールドなので、クラスの外から読み取れないのです。
もちろん、読み取れないので、書き換えることも不可能です。

```js
console.log(foo.#name);  // エラー
```


## メソッドやスタティックとも組み合わせられる

プライベートなメソッド、プライベートでスタティックなフィールド・メソッドも作ることができます。前述のクラスフィールド宣言とあわせ、ES2022からは次のフィールド・メソッド宣言ができるようになりました。

- パブリックフィールド
- プライベートフィールド
- パブリック・スタティック・フィールド
- プライベート・スタティック・フィールド
- パブリック・メソッド
- プライベート・メソッド
- パブリック・スタティック・メソッド
- プライベート・スタティック・メソッド

すべてを試せるデモがこちらです。

```js
class MyClass {
  // public field
  foo = "public field: 寿司";
  // private field
  #bar = "private field: ラーメン";

  // public static field
  static qux = "public static field: うどん";
  // private static field
  static #corge = "private static field: 麻婆豆腐";

  // public method
  grault() {
    return console.log("public method: みかん");
  }

  // private method
  #garply() {
    return console.log("private method: ぶどう");
  }

  // public static method
  static waldo() {
    return console.log("public static method: みかん");
  }

  // private static method
  static #fred() {
    return console.log("private static method: ぶどう");
  }
}
```

デモは次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/KKQLzJG)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/KKQLzJG)

## 関連資料

- [Class field declarations for JavaScript - tc39](https://github.com/tc39/proposal-class-fields)

# `instanceof`よりも安全にインスタンスかどうかの確認ができる、プライベートフィールドの`in`演算子

プライベートフィールドに対して`in`演算子が使えるようになりました。`instanceof`よりも安全にインスタンスかどうかの確認ができるようになります。提案では「Ergonomic brand checks」と呼ばれています。

▼ 簡単な例

```js
class MyClass {
  #brand
  static isMyClass(object) {
    return #brand in object;
  }
}

console.log(MyClass.isMyClass(new MyClass())); // true
console.log(MyClass.isMyClass(new Date())); // false
```

## 前提知識① `insntanceof`演算子は正確にインスタンスのチェックができない

あるインスタンスが、あるクラスのインスタンスかどうかを調べるかどうかを調べるために、`instanceof`メソッドが使われているケースをよく見かけます。

```js
class MyClass {
}

const myInstance = new MyClass();
console.log(myInstance instanceof MyClass); // true
```

しかし、`instanceof`には注意すべき挙動があります。次のコードを見てください。

```js
class MyClass {
}

const myInstance = new MyClass();

const foo = {
  name: "名探偵コナン"
};

// ポイント
Object.setPrototypeOf(foo, myInstance);
console.log(foo instanceof MyClass); // trueになってしまう！
```

`foo`は、`MyClass`のインスタンスではありません。しかし、 オブジェクトのプロトタイプを書き換える`Object.setPrototypeOf`を用いると、`foo instanceof MyClass`が`true`になってしまうのです。

`instanceof`はプロトタイプベースであり、正確に「インスタンスかどうか」をチェックできるものではないのです。


## 前提知識② プライベートフィールドでインスタンスのチェック

上記の問題を解決するために、プライベートフィールドを使います。プライベートフィールドには、存在しないプライベートフィールドにアクセスしようとすると例外をスローをする仕組みがあります。

```js
class MyClass {
  #myBrand
  
  static check(object) {
    object.#myBrand;
  }
}

const myInstance = new MyClass();
MyClass.check(myInstance); // OK

const foo = {
  name: "名探偵コナン"
};
MyClass.check(foo); //例外をスロー
```

この仕組みを使うと、`instanceof`よりも正確にインスタンスかどうかの確認ができます。

`MyClass`にスタティックなメソッド`isMyClass`を定義して、引数のオブジェクトが`MyClass`のインスタンスかどうかをチェックできるようにしてみましょう。`try`・`catch`を使い、エラーが起こらない（=存在するプライベートフィールドへのアクセス）であれば`true`を返すようにします。

```js
class MyClass {
  #myBrand

  static isMyClass(object) {
    try {
      object.#myBrand;
      return true;
    } catch {
      return false
    }
  }
}

const myInstance = new MyClass();
console.log(MyClass.isMyClass(myInstance)); // true

const foo = {
  name: "名探偵コナン"
};

// ポイント
Object.setPrototypeOf(foo, myInstance);
console.log(MyClass.isMyClass(foo)); // false
```

正しくインスタンスのチェックができるようになりました。が、`try`・`catch`の処理は煩雑ですね。そこで登場したのが、ES2022のプライベートフィールドのための`in`です。

## `in`でシンプルにインスタンスかどうかの確認

`in`を使えば、わざわざ `try`・`catch`を使わず、シンプルにインスタンスかどうかのチェックができます。

```js
class MyClass {
  #myBrand

  static isMyClass(object) {
    return #myBrand in object;
  }
}

const myInstance = new MyClass();
console.log(MyClass.isMyClass(myInstance)); // true

const foo = {
  name: "名探偵コナン"
};
console.log(MyClass.isMyClass(foo)); // false
```

`Object.setPrototypeOf(foo, myInstance);`と`foo`のプロトタイプを書きかえたとしても、正しく動作します。

```js
Object.setPrototypeOf(foo, myInstance);
console.log(MyClass.isMyClass(foo)); // falseのまま
```

デモは次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/RwQmbXg)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/RwQmbXg)


### コラム: tc39の「ブランドチェック」

あるコードで作られたデータが、特定のデータ型かどうかをチェックすることを、tc39では「ブランドチェック（brand check）」と呼んでいます。`Array.isArray(データ)`で引数のデータが配列かどうかをチェックすることは、ブランドチェックの好例です。**よく間違えられていますが、`instanceof`はブランドチェックではありません**。

今回のようにあるオブジェクトが`MyClass`のデータかどうかをチェックすることは、プライベートフィールドと`in`演算子を使ってブランドチェックをしていると言えます。

## 関連資料

- [Ergonomic brand checks for Private Fields - tc39](https://github.com/tc39/proposal-private-fields-in-in)
- [Brand Check](https://github.com/tc39/how-we-work/blob/main/terminology.md#brand-check)

# `async`なしでも`await`が使えるようになる、トップレベルでの`await` 

トップレベルでの`await`とは、`async`を使わずとも`await`が使えるようになる構文のことです。

▼ 簡単な例

```js
await new Promise((resolve) => {
  setTimeout(() => {
    alert("1秒経ちました");
    resolve();
  }, 1000);
});
````

## 従来は`async`関数の中でしか`await`を使えなかった

`await` を使うと、非同期処理を簡潔に記述できます。しかし、これまで `await` は、 `async` 関数の中でしか使えず、トップレベルで使うことはできませんでした。

▼ ES2021以前の `await`

```js
const main = async () => {
  await new Promise((resolve) => {
    setTimeout(() => {
      alert("ヨーソロー！");
      resolve();
    }, 1000);
  });
}

main();
```

## ES2022からは `async` 関数外でも`await`を使える

ES2022からは、`async` 関数の外でも `await`を使えます。

▼ ES2022以降の `await`

```js
await new Promise((resolve) => {
  setTimeout(() => {
    alert("ヨーソロー！");
    resolve();
  }, 1000);
});
```

## トップレベルでの `await` はモジュールでのみ使用可能

トップレベルでの `await`は、モジュールでのみ使用可能です。
`script`タグでJavaScriptを読み込む際に `type="module"`とすることで、JavaScriptがモジュールで実行されます。

▼ HTMLでJavaScriptの読み込み

```html
<script type="module" src="index.js"></script>
```

## ユースケース：i18n用にファイルを読み込む

トップレベル`await`の活用例として、i18n対応（国際化対応）の方法を見てみましょう。次のように、タイトルとボタンを表示するHTMLがあるとします。タイトルとボタンのラベルは、ブラウザUIが英語設定であれば英語を、日本語設定であれば日本語を表示したいです。

```html
<h1></h1>
<button></button>
```

日本語用・英語用のテキストは、次の2つのJavaScriptファイルにまとまっており、ユーザーの環境に応じてどちらか1つのファイルを読み込みたいです。

▼ 日本語用の`lang-ja.js`

```js
export const translations = {
  title: "私のウェブサイト",
  button: "ボタン"
};
```

▼ 英語用の`lang-en.js`

```js
export const translations = {
  title: "My Website",
  button: "Button"
};
```

トップレベル`await`を使えば次のようにできます。ブラウザーUIの言語設定（`navigator.language`）に英語（`en`）が含まれていれば `lang-en.js`を、そうでなければ `lang-ja.js`を読み込みます。読み込んだ結果を、さらに `export`します。

▼ lang.js

```js
export const { translations } =
  navigator.language.includes("en") ?
    await import("./i18n/lang-en.js"):
    await import("./i18n/lang-ja.js");
```

メインの処理`index.js`では、ファイルの読み分けを行っている`lang.js`を静的な`import`（`import()`ではない）で読み込めます。**`lang.js`でのファイル読み込みが完了するまで`index.js`内の処理は遅延され、`translations` には必ずデータが入った状態でタイトルやテキストの書き換えが行われます**。

▼ index.js

```js
import { translations } from "./lang.js";

document.querySelector("h1").textContent = translations.title;
document.querySelector("button").textContent = translations.button;
```

実行結果は次のとおりです。Google Chromeでは、言語設定（[chrome://settings/languages](chrome://settings/languages)）から言語設定を変更できます。日本語設定の場合、日本語のタイトル・ボタンラベルが表示されます。

![](/images/es2022-whats-new/await-ja.png)

英語設定に変更すると、英語のタイトル・ボタンラベルが表示されます。

![](/images/es2022-whats-new/await-en.png)

もしトップレベル`await`がなかったら、同様の処理をしたければ複雑なコードを記述する必要があります（[参考](https://github.com/tc39/proposal-top-level-await#avoiding-the-race-through-significant-additional-dynamism)）。トップレベル`await`のおかげでスッキリとしたコードが書けるのです。

デモは次のとおりです。

- [新しいタブでデモを開く](https://tonkotsuboy.github.io/kanobox/20220621_toplevel_await/index.html)

### コラム: tc39のサンプルコードは注意

念のためにお伝えしておくと、tc39のREADMEに記述されている次のコードは、実用にはオススメできません。

```js
const strings = await import(`/i18n/${navigator.language}`);
```

理由は2つ。

- `navigator.language`の数だけファイルを準備する必要があります。英語だけでも、`en`や`en-US`のファイルを作る必要があります
- `strings`の型が`any`型にしか推論されなくなります。`navigator.language`を直接使っているので、ファイル名が一意に定まりません。前述の`lang.js`の記法では、ファイル名は `lang-en.js`か`lang-ja.js`のどちらかに定まります。よって、`translations`（tc39でいう`strings`変数）は`title`と`button`がそれぞれ`string`のオブジェクトと推論されます。

あくまで説明用の部分的なコードであると注意しましょう。

## 関連資料

- [Top-level await - tc39](https://github.com/tc39/proposal-top-level-await)
- [top-level awaitがどのようにES Modulesに影響するのか完全に理解する](https://qiita.com/uhyo/items/0e2e9eaa30ec2ff05260)

# JavaScriptで「staticイニシャライザー」ができるように

staticイニシャライザーとは、クラス定義時に初期化処理を一度だけ実行できるブロックのことです。Javaなどの世界ではすでにある仕組みです。

## 簡単な例

```js
class MyClass {
  static x;
  
  static {
    this.x = "こんにちは"
  }
}

console.log(MyClass.x); // こんにちは
```

## staticイニシャライザーがない場合の問題点

ひょんなことから、`enum`（列挙型）を作る必要が出てきたとしましょう。列挙型とは、似た値を集めたものです。たとえば、果物用の列挙型を作成し、`FruitsEnum.apple`や`FruitsEnum.grape`のような形で扱えるようにしてみましょう。

次のような `FruitsEnum` の実装が考えられます。

```js
class FruitsEnum {
  static apple = Symbol("りんご");
  static orange = Symbol("みかん");
  static grape = Symbol("ぶどう");
}
```

`FruitsEnum`から、すべてのキーを取得できるスタティックなフィールド`allFruits`をクラス内に実装したいとします。

 staticイニシャライザーが使えない時代は、次のようにクラス外に処理を記述する必要がありました。

```js
class FruitsEnum {
  static apple = Symbol("りんご");
  static orange = Symbol("みかん");
  static grape = Symbol("ぶどう");

  static allFruits;
}

FruitsEnum.allFruits = Object
  .keys(FruitsEnum)  // 各キーを取得する
  .filter(key => key !== "allFruits");  // allFruitsを除外する

console.log(FruitsEnum.allFruits);
// [apple, orange, grape]
```
## staticイニシャライザーを使えば、クラス内に処理を記述できる

staticイニシャライザーを使えば、次のようにクラス内に処理を記述できます。クラスの外部に処理を記述しなくてよくなるので、処理のまとまりがわかりやすくなります。

```js
class FruitsEnum {
  static apple = Symbol("りんご");
  static orange = Symbol("みかん");
  static grape = Symbol("ぶどう");

  static {
    this.allFruits = Object.keys(this);
  }
}

console.log(FruitsEnum.allFruits);
// [apple, orange, grape]
```

デモは次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/LYQoeqM)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/LYQoeqM)


## 関連資料

- [class static initialization blocks - tc39](https://github.com/tc39/proposal-class-static-block)
- [ES2022 feature: class static initialization blocks](https://2ality.com/2021/09/class-static-block.html)


# 正規表現で、マッチ部分の開始・終了インデックスを取得できる`d`フラグ

`d`フラグとは、正規表現でマッチ部分の開始・終了インデックスを取得できるものです。

## 構文

```js
const result = /正規表現/d.exec(文字列);
console.log(result.indices);
```

■ 意味<br>
文字列から、正規表現にマッチする部分を取得してください。
取得したら、`indices` プロパティにマッチ部分の開始・終了インデックスを格納してください。

## 前提知識① 正規表現でマッチ部分の情報を取得する

`/正規表現/.exec(文字列)`や、`文字列.match(正規表現)`を実行すると、正規表現にマッチした文字列の情報を取得できます。たとえば、次のコードは`/私の姓は(.*)、名前は(.*)です/`という正規表現にマッチする文字列の情報を取得しています。

```js
// 正規表現
const regrex = /私の姓は(.*)、名前は(.*)です/;

const result = '私の姓は山田、名前は太郎です'.match(regrex);
console.table(result);
```

実行結果は次の通り。「`山田`」と「`太郎`」という文字列が取得できているのがわかります。

![](/images/es2022-whats-new/dflag-1.png)

## 前提知識② フラグとマッチ部分に名前をつける

ES2018では、Named Capture Groupsという仕様が入りました。正規表現で`?<名前>`とすることで、マッチした部分に名前をつけることができ、マッチ部分の情報を探しやすくなります。なお、この場合は`u`フラグが必要です


```js
const regrex = /私の姓は(?<family>.*)、名前は(?<name>.*)です/u;

const result = '私の姓は山田、名前は太郎です'.match(regrex);
console.table(result);

console.log(result.groups.family);  // 山田
console.log(result.groups.name);  // 太郎
```

実行結果は次のとおりです。

![](/images/es2022-whats-new/dflag-2.png)

## `d` フラグでマッチ部分の開始・終了インデックスを取得する

本題の`d`フラグです。`d`フラグを使うと、マッチ部分の開始・終了インデックスを取得できます。前述のNamed Capture Groupsとあわせて使うことで、マッチ部分に名前をつけつつ、簡単に開始・終了インデックスを取得できるようになります。


```js
const regrex = /私の姓は(?<family>.*)、名前は(?<name>.*)です/du;

const result = '私の姓は山田、名前は太郎です'.match(regrex);
console.log(result);

// ☆ indicesプロパティでマッチ部分の開始・終了インデックスを取得する
const indicesGroups = result.indices.groups;
console.log(indicesGroups.family);  // [4, 6]
console.log(indicesGroups.name);  // [10, 12]
```

![](/images/es2022-whats-new/dflag-3.png)

「☆ indicesプロパティでマッチ部分の開始・終了インデックスを取得する」のところでは、それぞれ`[4, 6]`と`[10, 12]`という配列が取得されています。次のことを示します。

- 「山田」という文字列の開始インデックスは`4`、終了インデックスは`6`
- 「太郎」という文字列の開始インデックスは`10`、終了インデックスは`12`

デモは次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/dydEbBO)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/dydEbBO)

## 関連資料

- [RegExp Match Indices for ECMAScript - tc39](https://github.com/tc39/proposal-regexp-match-indices)
- [RegExp Named Capture Groups - tc39](https://github.com/tc39/proposal-regexp-named-groups)


# 配列の「最後の要素」が簡単に取得できるようになる`at()`

配列の`at()`メソッドを使うと、インデックスを指定して要素を取得できます。

## 構文

`配列`.at(`インデックス`)

■ 意味
`配列`の`インデックス`位置の要素を取得してください

※ `文字列.at()`でもOK

## 従来は、配列の最後の要素を取得するのは煩雑だった

「配列の最後の要素を取得したい」というケースはよくあります。配列は`[]`でインデックスを指定して要素を取得できますが、`-1`と指定したからといって最後の要素は取得できません（できる言語もあります）。

よって、配列の最後の要素を取得するには、次のような煩雑な処理が行われていました。`myArray.length`が配列の要素数を返すことを利用し、`myArray.length - 1`で配列の最後の要素のインデックスを指定していたのです。

```js
const myArray = ["りんご", "バナナ", "ぶどう"];
console.log(myArray[myArray.length - 1]); // ぶどう
```

## `at(-1)`で最後の要素を取得できる

配列の`at()`メソッドを使うと、位置を指定して要素を取得できます。**本メソッドの嬉しいところは、配列の最後の要素の取得が簡単になることです**。前述の処理よりも直感的ですね。

```js
const myArray = ["りんご", "バナナ", "ぶどう"];
console.log(myArray.at(-1)); // ぶどう
```

デモはつぎのとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/vYdwWKZ)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/vYdwWKZ)

## 関連資料

- [.at() method on all the built-in indexables](https://github.com/tc39/proposal-relative-indexing-method)

# オブジェクトが指定のプロパティを持っているかを簡単にチェックできる `Object.hasOwn()`

`Object.hasOwn()`を使うと、オブジェクトが指定のプロパティを持っているかを簡単かつ安全にチェックできます。

## 構文

```js
Object.hasOwn(オブジェクト, プロパティ);
```

■ 意味<br>
オブジェクトが指定のプロパティを持っているかどうか？を真偽値で返す

■ 戻り値<br>
真偽値

## 従来は`hasOwnProperty()`で面倒な記述が必要だった

`hasOwn()`メソッドを使うと、オブジェクトが指定のオブジェクトを持っているかどうかを簡単に調べられます。

従来、同様の判定をするメソッドとして `hasOwnProperty()` メソッドがありました。しかし、安易に`hasOwnProperty()`を使うのは危険です。対象のオブジェクトが同名の`hasOwnProperty`という名前のメソッドを持っていた場合、上書きされてしまうためです。

▼ 従来の`hasOwnProperty()`の危険性

```js
const myObject = {
  name: "鈴木",
  hasOwnProperty: () => {
    // hasOwnPropertyが上書きされる
    return false;
  },
}

console.log(myObject.hasOwnProperty("name"));
// nameはあるのに常にfalse
```

この挙動を回避するためには、次のようにプロトタイプの`hasOwnProperty`プロパティを使う必要があります。

```js
Object.prototype.hasOwnProperty.call(myObject, 'name'); // true
```

ちなみに、ESLintを使っていれば、`myObject.hasOwnProperty("name")`のような処理はデフォルトで警告されます。

▼ ESLintでのエラー（[確認コード](https://eslint.org/demo#eyJ0ZXh0IjoiY29uc3QgbXlPYmplY3QgPSB7XG4gIG5hbWU6IFwi6Yi05pyoXCIsXG59XG5cbm15T2JqZWN0Lmhhc093blByb3BlcnR5KFwibmFtZVwiKTsiLCJvcHRpb25zIjp7InBhcnNlck9wdGlvbnMiOnsiZWNtYVZlcnNpb24iOiJsYXRlc3QiLCJzb3VyY2VUeXBlIjoic2NyaXB0IiwiZWNtYUZlYXR1cmVzIjp7fX0sInJ1bGVzIjp7ImNvbnN0cnVjdG9yLXN1cGVyIjoyLCJmb3ItZGlyZWN0aW9uIjoyLCJnZXR0ZXItcmV0dXJuIjoyLCJuby1hc3luYy1wcm9taXNlLWV4ZWN1dG9yIjoyLCJuby1jYXNlLWRlY2xhcmF0aW9ucyI6Miwibm8tY2xhc3MtYXNzaWduIjoyLCJuby1jb21wYXJlLW5lZy16ZXJvIjoyLCJuby1jb25kLWFzc2lnbiI6Miwibm8tY29uc3QtYXNzaWduIjoyLCJuby1jb25zdGFudC1jb25kaXRpb24iOjIsIm5vLWNvbnRyb2wtcmVnZXgiOjIsIm5vLWRlYnVnZ2VyIjoyLCJuby1kZWxldGUtdmFyIjoyLCJuby1kdXBlLWFyZ3MiOjIsIm5vLWR1cGUtY2xhc3MtbWVtYmVycyI6Miwibm8tZHVwZS1lbHNlLWlmIjoyLCJuby1kdXBlLWtleXMiOjIsIm5vLWR1cGxpY2F0ZS1jYXNlIjoyLCJuby1lbXB0eSI6Miwibm8tZW1wdHktY2hhcmFjdGVyLWNsYXNzIjoyLCJuby1lbXB0eS1wYXR0ZXJuIjoyLCJuby1leC1hc3NpZ24iOjIsIm5vLWV4dHJhLWJvb2xlYW4tY2FzdCI6Miwibm8tZXh0cmEtc2VtaSI6Miwibm8tZmFsbHRocm91Z2giOjIsIm5vLWZ1bmMtYXNzaWduIjoyLCJuby1nbG9iYWwtYXNzaWduIjoyLCJuby1pbXBvcnQtYXNzaWduIjoyLCJuby1pbm5lci1kZWNsYXJhdGlvbnMiOjIsIm5vLWludmFsaWQtcmVnZXhwIjoyLCJuby1pcnJlZ3VsYXItd2hpdGVzcGFjZSI6Miwibm8tbG9zcy1vZi1wcmVjaXNpb24iOjIsIm5vLW1pc2xlYWRpbmctY2hhcmFjdGVyLWNsYXNzIjoyLCJuby1taXhlZC1zcGFjZXMtYW5kLXRhYnMiOjIsIm5vLW5ldy1zeW1ib2wiOjIsIm5vLW5vbm9jdGFsLWRlY2ltYWwtZXNjYXBlIjoyLCJuby1vYmotY2FsbHMiOjIsIm5vLW9jdGFsIjoyLCJuby1wcm90b3R5cGUtYnVpbHRpbnMiOjIsIm5vLXJlZGVjbGFyZSI6Miwibm8tcmVnZXgtc3BhY2VzIjoyLCJuby1zZWxmLWFzc2lnbiI6Miwibm8tc2V0dGVyLXJldHVybiI6Miwibm8tc2hhZG93LXJlc3RyaWN0ZWQtbmFtZXMiOjIsIm5vLXNwYXJzZS1hcnJheXMiOjIsIm5vLXRoaXMtYmVmb3JlLXN1cGVyIjoyLCJuby11bmRlZiI6Miwibm8tdW5leHBlY3RlZC1tdWx0aWxpbmUiOjIsIm5vLXVucmVhY2hhYmxlIjoyLCJuby11bnNhZmUtZmluYWxseSI6Miwibm8tdW5zYWZlLW5lZ2F0aW9uIjoyLCJuby11bnNhZmUtb3B0aW9uYWwtY2hhaW5pbmciOjIsIm5vLXVudXNlZC1sYWJlbHMiOjIsIm5vLXVudXNlZC12YXJzIjoyLCJuby11c2VsZXNzLWJhY2tyZWZlcmVuY2UiOjIsIm5vLXVzZWxlc3MtY2F0Y2giOjIsIm5vLXVzZWxlc3MtZXNjYXBlIjoyLCJuby13aXRoIjoyLCJyZXF1aXJlLXlpZWxkIjoyLCJ1c2UtaXNuYW4iOjIsInZhbGlkLXR5cGVvZiI6Mn0sImVudiI6e319fQ==)）

![](/images/es2022-whats-new/hasown-error.png)


https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty#using_hasownproperty_as_a_property_name

## ES2022からは`hasOwn()`でシンプルに記述できる

ES2022で実装された`hasOwn()`メソッドを使えば、前述のようなプロトタイプのプロパティを使うことなく、シンプルにプロパティの確認ができます。

```js
const myObject = {
  name: "鈴木",
}

console.log(Object.hasOwn(myObject, "name"));
// true
```

`hasOwnProperty`を上書きしたとしても、正しくチェックできます。


```js
const myObject = {
  name: "鈴木",
  hasOwnProperty: () => {
    return false;
  },
}

console.log(Object.hasOwn(myObject, "name"));
// true
```


@[codepen](https://codepen.io/tonkotsuboy/pen/oNErWJN)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/oNErWJN)

## 関連資料

- [Accessible Object.prototype.hasOwnProperty() - tc39](https://github.com/tc39/proposal-accessible-object-hasownproperty)
- [JavaScript は hasOwnProperty というプロパティ名を保護していません - MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty#using_hasownproperty_as_a_property_name)


# 複数のエラーをチェインし、原因を追跡しやすくできる`Error.cause`

## 構文

```js
try {
  // なにかしらのエラーが発生する
} catch(error) {
  throw new Error("エラーの内容", { cause: error })
}
```

■ 意味<br>
エラーを投げるとき、エラーの`cause`プロパティに`error`を格納してください


## 複数のエラーを投げるときの問題点

「API1の通信に成功したら、API2と通信したい。それぞれどこで失敗したかに応じて、処理を振り分けたい」というように、複数のエラーを取り扱いたいケースは多くあります。今回は、50%の確率でエラーになる関数を使って確認してみましょう。

次の恐ろしく危険な2つの関数があります。それぞれ、50%の確率でオブジェクトが定義されていないというエラー（ReferenceError）が投げられます。

▼ 恐ろしく危険な2つの関数

```js
const function1 = () => {
  if (Math.random() > 0.5) {
    // fooが定義されていないというReferenceError
    foo.bar
  }
}

const function2 = () => {
  if (Math.random() > 0.5) {
    // bazが定義されていないというReferenceError
    baz.qux
  }
}

function1()
function2()
```

それぞれの関数内で、`try`・`catch`を使って例外をキャッチしましょう。このとき、`catch`の中身をどうするのかにはいくつかのアプローチがあります。


```js
// 50%の確率でエラー
const function1 = () => {
  try {
    if (Math.random() > 0.5) {
      foo.bar
    }
  } catch(error) {
    // どうする？
  }
}

// 50%の確率でエラー
const function2 = () => {
  try {
    if (Math.random() > 0.5) {
      baz.qux
    }
  } catch(error) {
    // どうする？
  }
}

// function1とfunction2を実行する
try {
  function1()
  function2()
  console.log("成功です！")
} catch (error) {
  console.log(error);
  console.log(error.cause);
}
```

方法1: `throw new Error("文字列")`

```js
.catch(error) {
  throw new Error("fooプロパティなんてないよ😡")
}
```

この場合、`ReferenceError`の情報は消えてしまいます。

![](/images/es2022-whats-new/error-bad.png)

方法2: `Error`を拡張したカスタムクラスを作成する

```js
class CustomError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause
  }
}

// 中略
.catch(error) {
  throw new CustomError("fooプロパティなんてないよ😡", error)
}
```

この場合は、`CustomError`を作るのが煩雑です。

## `Error.cause` を使ってエラーをチェインさせる

`cause`プロパティを使うと、元のエラーの情報を残しつつ、エラーを投げることができます。

```js
// 50%の確率でエラー
const function1 = () => {
  try {
    if (Math.random() > 0.5) {
      foo.bar;
    }
  } catch (error) {
    throw new Error("fooが存在しないですよ😡", {
      cause: error
    });
  }
};

// 50%の確率でエラー
const function2 = () => {
  try {
    if (Math.random() > 0.5) {
      baz.qux;
    }
  } catch (error) {
    throw new Error("bazが存在しないですよ🤯", {
      cause: error
    });
  }
};

// function1とfunction2を実行する
try {
  function1();
  function2();
  console.log("成功です！");
} catch (error) {
  console.log(error);
  console.log(error.cause);
}
```

`function1`と`function2`でエラーが発生すると、それぞれ次のような出力になります。いずれも、エラーメッセージと、`error.cause`から元エラー（ReferenceError）の情報が取得できているのがわかります。

▼ `function1`でエラー

![](/images/es2022-whats-new/error-cause.png)

▼ `function2`でエラー

![](/images/es2022-whats-new/error-cause-2.png)

デモは次のとおりです。

@[codepen](https://codepen.io/tonkotsuboy/pen/RwQmjGZ)

- [新しいタブでデモを開く](https://codepen.io/tonkotsuboy/pen/RwQmjGZ)


## 関連資料

- [Error Cause](https://github.com/tc39/proposal-error-cause)


# 対応環境

本記事で紹介したES2022の全機能は、各環境で使用可能です。

- Google Chrome
- Firefox
- Safari
- Microsoft Edge
- TypeScript
- Node.js

「[ECMAScript compatibility table](https://kangax.github.io/compat-table/es2016plus/)」の「2022 features」で、細かい対応バージョンを確認できます。

SafariのES2022対応が遅れていましたが、今年リリースされた15.4で全て対応しました。

# ES2022を使って便利に開発しよう

本記事では正式仕様としてリリースされたES2022の新機能を紹介しました。どれも開発をラクにしてくれるものばかりで、筆者も積極的に開発の現場で使っています。

ECMAScriptは次のES2023に向けて仕様策定がすでに始まっています。`findLast()`や`findLastIndex()`など、また便利な機能が入ってきそうです。新しい機能をキャッチアップし、楽しく開発していきましょう。

ES2022のLanguage Specificationは、こちらから確認できます。
- [ECMAScript® 2022 Language Specification](https://262.ecma-international.org/13.0/)

ES2021の全新機能はこちらです。

https://zenn.dev/tonkotsuboy_com/articles/es2021-whats-new



TwitterでもJavaScriptやCSSの最新情報を発信しています！
https://twitter.com/tonkotsuboy_com
