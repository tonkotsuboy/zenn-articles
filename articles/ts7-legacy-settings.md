---
title: "TypeScript 7.0で削除される「レガシー設定」たち"
emoji: "👴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:  ["typescript", "nodejs", "javascript"]
published: true
publication_name: ubie_dev
---


現在開発中のTypeScript 7では、target: es5、baseUrl等の長年のレガシーな設定が削除されたり、strict: trueが標準になるなどのデフォルトの挙動が変更されます。



# TypeScript 7とは

現在のTypeScriptのコンパイラはTypeScriptで記述されていますが、TypeScript 7ではGO言語によるネイティブコンパイラーとなります。コンパイル速度が10倍向上するという公式発表があり、実際に私も検証したところ確かに10倍高速化されました。

https://zenn.dev/ubie_dev/articles/typescript7-tsgo-whatsnew

https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/

TypeScript 7.0といえば、Go言語による移植版（tsgo）による爆速化が大きな話題です。実際にどれくらい速いのか、どうやって試すのかについては以前の記事で紹介しました。

https://zenn.dev/ubie_dev/articles/typescript7-tsgo-whatsnew

# TypeScript 7.0でサヨナラになる設定たち 👋

TypeScript 7.0では、バージョン6.0で非推奨（Deprecated）となる予定の挙動やフラグが、実際に削除されます。これまで `.eslintrc` や `tsconfig.json` に何気なく書いていた設定がエラーになる可能性があるため、今のうちに予習しておきましょう。

主な変更点は以下の通りです。

## 1\. `--strict` がデフォルトで有効に

従来は、TypeScriptの厳格なチェック（`noImplicitAny` や `strictNullChecks` など）を有効にするには、明示的に `"strict": true` と書く必要がありました。TypeScript 7.0からは、これがデフォルトでONになります。現代において、`"strict": true`を設定してないことはまずありえないと思うので、この挙動は嬉しいですね。

### TypeScript 5.9から `"strict": true`のはずでは？

TypeScript 5.9からは、`tsc --init` で生成された `tsconfig.json` では、最初から `"strict": true` が記述されていました。

しかし、TypeScript 6.0からはコンパイラの挙動そのものが変わります。

- これまで
  - tsconfig.json に `strict` の記述がない場合、コンパイラは OFF (false) として扱う
- これから
  - tsconfig.json に`strict`の記述がない場合、コンパイラは**ON (true)**として扱う

https://github.com/microsoft/TypeScript/issues/62333

### 2\. `--target` が最新のECMAScriptに

`target` は、TypeScriptをコンパイルした後に「どのバージョンのJavaScriptを出力するか」を決める設定です。TypeScript 7.0からは最新の安定版ECMAScriptがデフォルトになります。

モダンブラウザやNode.jsの最新版を使っている場合、余計なダウンパイル（`async/await` を `generator` に変換するなど）が発生しなくなり、生成コードがスッキリします。

https://github.com/microsoft/TypeScript/issues/62198

### 3\. `target: es5` の削除

これが一番影響が大きいかもしれません。**IE11時代の遺物である `es5` への出力サポートが削除されます。**

- **これまで:** IE11など古いブラウザのために `target: "es5"` を指定していた。
- **これから:** サポートされる最も古いターゲットは `es2015` (ES6) になる。

もしどうしてもES5のコードが必要な場合（レガシー環境など）、TypeScriptの出力結果をさらにBabelなどで変換する必要があります。

### 4\. `--baseUrl` の削除

`baseUrl` は、`import` 文を書く際の「基準となるディレクトリ」を指定する設定でした。これを使うと `../../components/Button` ではなく `components/Button` のように絶対パス風に書くことができました。

しかし現在では、Node.js標準の機能や `package.json` の `exports` / `imports`、あるいはバンドラーの機能を使うのが主流です。TS 7.0ではこの古い解決方法は削除されます。

:::message
今後は `paths` オプションの設定方法なども見直す必要があるかもしれません。
:::

### 5\. `--moduleResolution: node10` の削除

`moduleResolution` は、TypeScriptが `import` されたファイルをどうやって探すか（解決するか）を決めるロジックです。
`node10`（または単に `node`）は、CommonJS時代の古いNode.jsの挙動を模倣するものでした。

- **これから:** `bundler`（ViteやWebpack用）や `nodenext`（最新のNode.js用）を使うのが正解になります。

### 6\. `rootDir` の挙動変更

`rootDir`（ソースファイルのルートディレクトリ）のデフォルト値が「カレントディレクトリ」になります。
`outDir`（出力先）を使っている場合、トップレベルのソースファイルが `tsconfig.json` と同じ階層にあるか、明示的に `rootDir` を指定する必要があります。

-----

## 移行を助けるツール「ts5to6」

「えっ、`tsconfig.json` 全部書き直すの…？」と思った方、安心してください。
TypeScriptチームは、設定ファイルの更新を自動化する実験的なツール **`ts5to6`** を開発しています。

[https://www.npmjs.com/package/@andrewbranch/ts5to6](https://www.google.com/search?q=https://www.npmjs.com/package/%40andrewbranch/ts5to6)

現在は主に `baseUrl` と `rootDir` の修正に対応しており、ヒューリスティック（推論）を使ってプロジェクト構成を解析し、自動で書き換えてくれます。

### 使い方

```bash
# baseUrlの設定を自動修正する
npx @andrewbranch/ts5to6 --fixBaseUrl tsconfig.json

# rootDirの設定を自動修正する
npx @andrewbranch/ts5to6 --fixRootDir tsconfig.json
```

このツールはまだ実験段階ですが、TS 7.0への移行時には強力な味方になりそうです。

## さいごに

TypeScript 7.0は、パフォーマンス向上だけでなく、長年の「歴史的経緯」を断ち切るメジャーアップデートになりそうです。
特に `target: es5` の削除や `moduleResolution` の変更は、古いプロジェクトを抱えている現場では早めの対策が必要になるでしょう。

今のうちに `strict: true` や `moduleResolution: bundler` に慣れておき、スムーズにTS 7.0時代を迎えましょう！
