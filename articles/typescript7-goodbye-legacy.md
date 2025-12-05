---
title: "TypeScript 7で消えるtsconfigのレガシー設定。target: es5やbaseUrlにサヨウナラ"
emoji: "👴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:  ["typescript", "javascript", "tsgo"]
published: true
publication_name: ubie_dev
---

TypeScript 7では`target: es5`や`baseUrl`といった長年のレガシーな設定が削除され、`strict: true` が標準になるなどデフォルトの挙動が変更されます。本記事では、消えるレガシーな設定や挙動が変わる主要な設定について、設定の基本知識から代替案までを解説します。ご自身のtsconfig.jsonを更新する際の参考になれば幸いです。

# TypeScript 7とは

そもそもTypeScript 7ではコンパイラーが大きく変わります。現在のTypeScriptのコンパイラーはTypeScriptで記述されていますが、TypeScript 7ではGO言語によるネイティブコンパイラー「tsgo」となります。コンパイル速度が10倍向上するという公式発表があり、実際に私も検証したところ確かに10倍高速化されました。

TypeScript 7の基礎知識やインストール手順については次の記事を参照してください。

https://zenn.dev/ubie_dev/articles/typescript7-tsgo-whatsnew

# TypeScript 7で削除される設定👋

TypeScript 7で削除される設定や挙動が変わる主な設定について見てみましょう。

## 1\. `--strict` がデフォルトで有効に

従来は、TypeScriptの厳格なチェック（`noImplicitAny` や `strictNullChecks` など）を有効にするには明示的に `"strict": true` と書く必要がありました。**TypeScript 7からは`"strict": true` と書かなくても厳格なチェックが有効になります**。

逆に`"strict": false` を設定した場合は厳格なチェックが無効になります。現代において`"strict": false`を設定することはまずありえないと思うので、この挙動は嬉しいですね。

「もともと`tsc --init`したときは`"strict": true`のはずでは？」と思うかもしれませんが、TypeScript 7からはコンパイラの挙動そのものが変わり、`"strict": true` と書かなくても厳格なチェックが有効になる点がポイントです。

https://github.com/microsoft/TypeScript/issues/62333

## 2\. `--target` が最新のECMAScriptになる

`target`とはTypeScriptをコンパイルした後に「どのバージョンのJavaScriptを出力するか」を決める設定です。TypeScript 7からは最新の安定版ECMAScript（例: ES2025）がデフォルトになります。

tsconfig.jsonから`"target"`の行を消すと、TypeScript 7ではデフォルトで最新の安定版ECMAScript（例: ES2025）向けに必要最小限の変換を行います。モダンブラウザやNode.js最新版が対象なら環境向けの余計なダウンレベル変換が入らず、出力が軽くなり可読性も上がります

https://github.com/microsoft/TypeScript/issues/62198

## 3\. `--target: es5` の削除

さらば`es5`… IE11（大昔、私のようなおじさんが使っていた）のような古いブラウザ向けの出力サポートが削除されます。

従来は、最新の`async/await`などの構文をES5で動かすためにTypeScript内部で複雑な変換が行われていました。このサポートを終了することでTypeScript本体のコードがスリム化され、コンパイル速度の向上やメンテナンス性の改善が期待されています。

もしどうしてもレガシー環境向けにES5のコードが必要な場合、TypeScriptの出力をさらにBabelやSWCなどの外部ツールでトランスパイルする必要があります。

https://github.com/microsoft/TypeScript/issues/62196

## 4\. `--baseUrl` の削除

`baseUrl`とは`import` 文を書く際の「基準となるディレクトリ」を指定する設定でした。これを使うと、たとえば `src` フォルダーを基準にして `../../components/Button` ではなく `components/Button` のように絶対パス風に書くことができました。しかし、これは元々 AMD (RequireJS) 時代の遺産であり、現代の標準的なモジュール解決（Node.jsやブラウザのESモジュール）とは異なる挙動をしていました。

TypeScript 7からは、この`baseUrl`が削除されます。

「では`@/` などのエイリアスはどうなるの？」と心配になるかもしれませんが、`paths` オプション自体は残ります。`paths`とは、特定のインポートパスを実際のファイルパスにマッピング（紐付け）する設定です。「`@/` と書いたら `./src/` を参照する」といったルールを定義することで、深い階層のファイルも簡潔に記述できるようになります。

今後のエイリアスは、`baseUrl` に頼らずNode.js標準のSubpath imports（`#`から始まるエイリアス機能  https://nodejs.org/api/packages.html#subpath-imports ）機能を使うか、`paths` オプションによる明示的なパスマッピング（例：`"*": ["./src/*"]`）を使用しましょう。

https://github.com/microsoft/TypeScript/issues/62207

余談ですが、筆者はStorybookのモック機能でSubpath importsを使い始め、その便利さに感動してエイリアスをどんどんSubpath importsに置換しています。

## 5\. `--moduleResolution: node10` の削除

`moduleResolution`とは、TypeScriptがインポートされたファイルをどうやって探すかを決めるロジックです。`node10`（またはエイリアスである`node`）は、CommonJS時代の古いNode.jsの挙動を模倣するものでした。

TypeScript 7からは、`node10`（`node`）が削除されます。

大きな理由は、現代のライブラリ開発で標準となっているpackage.jsonの`exports`フィールドに対応していないためです。`exports`は「ライブラリの中で、外部に使わせて良いファイル」を厳密に定義する機能ですが、`node10` 設定はこの制限を無視して、ライブラリ内部のプライベートなファイルを勝手に`import`できてしまいます。これは、実行時エラーや、ライブラリのアップデートによる予期せぬ破損の原因となっていました。

今後は`bundler`（Vite、Next.js、webpack用）や`nodenext`（Node.js用）を使うようにしましょう。

https://github.com/microsoft/TypeScript/issues/62200

体感的に`moduleResolution: node`の設定は現場のプロジェクトで多く残っている印象です。

## 6\. `rootDir` の挙動変更

`rootDir`とは、「出力ディレクトリ（outDir）の中に、元のディレクトリ構造をどう反映させるか」を設定するものです。従来、`rootDir`を指定しない場合、TypeScriptがソースファイルの配置を見て「一番共通する親ディレクトリ」を推論していました。 TypeScript 7からは推論が廃止され、デフォルト値は常に「tsconfig.jsonのあるディレクトリ（`.`）」に固定されます。

変更の理由はコンパイル速度の向上です。これまでの「全ファイルを走査してルートを自動計算する処理」を廃止することで、TypeScriptの動作を高速化・単純化する狙いがあります。

### 何が起こるのか？

`src`フォルダーにソースを入れているプロジェクトで`rootDir`を明示していない場合、出力構造が変わってしまう可能性があります。

- 従来（自動計算）:
  - ソース: `src/index.ts`
  - 出力: `dist/index.js`（`src`がルートだと推論された）
- TS 7以降（デフォルト .）:
  - ソース: `src/index.ts`
  - 出力: `dist/src/index.js`（`src`ディレクトリがそのまま出力に含まれてしまう）

従来の挙動を維持したい場合はtsconfig.jsonで明示的に "rootDir": "./src" を指定してください。

https://github.com/microsoft/TypeScript/issues/62194

## 7. その他の変更点

紹介したものは使用頻度が高いものですが、それ以外の変更点は次で確認できます。

たとえば `"use strict"`のデフォルト化、`types`がデフォルトで`[]`になる、古いimport構文（`asserts { type: "json" }`）のサポート終了などがあります。

https://github.com/microsoft/TypeScript/issues?q=milestone%3A%22TypeScript%206.0.0%22%20label%3A%22Breaking%20Change%22

# 移行を助けるツール「ts5to6」

変更内容を手動で修正するのは大変ですよね。TypeScriptチームは、TypeScript 6への移行（および7への準備）を自動化する`ts5to6`というツールを開発しました。

https://www.npmjs.com/package/@andrewbranch/ts5to6

現在は主に `baseUrl`と `rootDir`の修正に対応しており、推論を使ってプロジェクト構成を解析し、自動で書き換えてくれます。

```bash
# baseUrlの設定を自動修正する
npx @andrewbranch/ts5to6 --fixBaseUrl tsconfig.json

# rootDirの設定を自動修正する
npx @andrewbranch/ts5to6 --fixRootDir tsconfig.json
```

たとえば、筆者のプロジェクトで`baseUrl`の設定を自動修正すると、次のようになりました。自動でやってくれるのはありがたいですね。

Before

```json
    "allowJs": false,
    "baseUrl": "./src",
    "paths": {
      "~/*": ["*"]
    },
```

After

```json
    "allowJs": false,
    "paths": {
      "~/*": ["./src/*"],
      "*": ["./src/*"]
    },
```

# さいごに

TypeScript 7は、ネイティブコンパイラーによるパフォーマンス向上だけでなく、長年の「今もう誰も使ってないだろう…」という設定とサヨナラするメジャーアップデートになります。まずは`ts5to6`を使いつつ、TypeScript 6に上げた段階で警告をすべて潰すことを目標にするとよいでしょう。

次のリリースはTypeScript 6.0で、TypeScript製コンパイラーの最後のバージョンとなります。その次のリリースはネイティブコンパイラー製のTypeScript 7.0がリリースされます。登場まであと少し。首を長くして待ちましょう。

参考記事

https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/

https://zenn.dev/ubie_dev/articles/typescript7-tsgo-whatsnew

この記事は Ubie Tech Advent Calendar 2025 の 4日目の記事です。

https://adventar.org/calendars/12070
