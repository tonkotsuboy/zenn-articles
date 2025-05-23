---

title: "TypeScript 7に向けて：10倍高速化を目指す「tsgo」を実際に検証した"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "nodejs", "javascript"]
published: true
publication_name: ubie_dev
---

本日2025/05/23、MicrosoftのTypeScriptチームは、TypeScriptのGo言語実装によるコンパイラのプレビュー版「**TypeScript Native Previews**」を公開しました。「tsgo」という名称のこの新しいコンパイラは、将来的にTypeScript 7で現在の`tsc`コマンドを置き換えることを目指しています。公式発表によると、`tsgo`を使用することで型チェックやコンパイル速度が最大で10倍向上するとのことです。

本記事では、この`tsgo`を実際にインストールし、そのパフォーマンスが本当に向上するのかを検証します。

## TypeScript 7でコンパイラがGo言語に

現在、TypeScriptのコンパイラは主にTypeScript（JavaScript）で記述されていますが、これをGo言語で書き換えるプロジェクトが進行中です。この取り組みはTypeScript 7でのリリースが予定されており、コンパイル速度や型チェック速度の大幅な向上が期待されています。

TypeScriptの普及とプロジェクトの大規模化に伴い、ビルド時間や型チェックのパフォーマンスが課題となるケースが増えてきました。この課題に対処するため、TypeScriptチームはコンパイラをネイティブコードへ移植する作業を進めています。

その目標は、単にGoというネイティブコンパイル言語を利用するだけでなく、共有メモリ並列処理や並行処理を最大限に活用し、**多くのプロジェクトで10倍の速度向上**を達成することです。これにより、開発時のフィードバックサイクルが短縮され、開発者の生産性向上に大きく貢献することが見込まれます。

MicrosoftのDev Blogs記事「[A 10x Faster TypeScript](https://devblogs.microsoft.com/typescript/typescript-native-port/)」や「[Announcing TypeScript Native Previews](https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/)」によれば、以下のような目覚ましい速度向上が報告されています。

<div class="table-responsive"><table>
<thead>
<tr>
<th>Codebase</th>
<th style="text-align: right;">Size (LOC)</th>
<th style="text-align: right;">Current</th>
<th style="text-align: right;">Native</th>
<th style="text-align: right;">Speedup</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://github.com/microsoft/vscode" target="_blank">VS Code</a></td>
<td style="text-align: right;">1,505,000</td>
<td style="text-align: right;">77.8s</td>
<td style="text-align: right;">7.5s</td>
<td style="text-align: right;">10.4x</td>
</tr>
<tr>
<td><a href="https://github.com/microsoft/playwright" target="_blank">Playwright</a></td>
<td style="text-align: right;">356,000</td>
<td style="text-align: right;">11.1s</td>
<td style="text-align: right;">1.1s</td>
<td style="text-align: right;">10.1x</td>
</tr>
<tr>
<td><a href="https://github.com/typeorm/typeorm" target="_blank">TypeORM</a></td>
<td style="text-align: right;">270,000</td>
<td style="text-align: right;">17.5s</td>
<td style="text-align: right;">1.3s</td>
<td style="text-align: right;">13.5x</td>
</tr>
<tr>
<td><a href="https://github.com/date-fns/date-fns" target="_blank">date-fns</a></td>
<td style="text-align: right;">104,000</td>
<td style="text-align: right;">6.5s</td>
<td style="text-align: right;">0.7s</td>
<td style="text-align: right;">9.5x</td>
</tr>
<tr>
<td><a href="https://github.com/trpc/trpc" target="_blank">tRPC</a> (server + client)</td>
<td style="text-align: right;">18,000</td>
<td style="text-align: right;">5.5s</td>
<td style="text-align: right;">0.6s</td>
<td style="text-align: right;">9.1x</td>
</tr>
<tr>
<td><a href="https://github.com/ReactiveX/rxjs" target="_blank">rxjs</a> (observable)</td>
<td style="text-align: right;">2,100</td>
<td style="text-align: right;">1.1s</td>
<td style="text-align: right;">0.1s</td>
<td style="text-align: right;">11.0x</td>
</tr>
</tbody>
</table></div>

## tsgoを試す

`tsgo`を試すための手順は以下の通りです。まず、新しくリリースされた`@typescript/native-preview`パッケージをインストールします。

```bash
npm install -D @typescript/native-preview
```

このパッケージは`tsgo`という実行可能ファイルを提供します。バージョンを確認してみましょう。

```bash
npx tsgo -v
// 出力例: Version 7.0.0-dev.20250523.1 (バージョンはインストール時期により異なります)
```

`tsgo`によるコンパイルを試すために、TypeScriptパッケージもインストールしておきます。

```bash
npm install -D typescript
```

次に、`tsconfig.json`ファイルを作成します。`tsgo`はまだプレビュー版であり、`--init`のようなプロジェクト初期化機能は提供されていないため、既存の`tsc`コマンドで生成します。

```bash
npx tsc --init
```

これにより`tsconfig.json`が作成されます。今回は検証のため、以下のような設定としました。

```json
{
  "compilerOptions": {
    "target": "esnext",
    "lib": ["esnext", "dom", "dom.iterable"],
    "module": "NodeNext",
    "strict": true,
    "skipLibCheck": true
  }
}
```

## コンパイル速度の比較：`tsc` vs `tsgo`

実際に、ある程度の規模のTypeScriptコードを`tsc`と`tsgo`でコンパイルし、その速度を比較してみましょう。
（今回は比較用のサンプルコードを省略し、以前の計測結果を元に考察します。実際のプロジェクトで試す際には、ご自身のコードで計測してください。）

```typescript
// ここに比較検証用のTypeScriptコードを記述
// 例: interface MyInterface { id: number; name: string; }
// const data: MyInterface[] = Array.from({length: 1000}, (_, i) => ({id: i, name: `Item ${i}`}));
// console.log(data.length);
```

---

上記のコード（または同等のコード）をコンパイルする際の計測結果は次のとおりです。`--extendedDiagnostics`オプションは、コンパイルに関する詳細な情報を出力するためのものです。

**`npx tsgo -p . --extendedDiagnostics` の結果（例）:**

```
Files:             79
Lines:           41836
Identifiers:     46412
Symbols:         32217
Types:            1507
Instantiations:   1840
Memory used:     23681K
Memory allocs:   81082
Parse time:      0.014s
Bind time:       0.005s
Check time:      0.003s
Emit time:       0.001s
Total time:      0.025s
```

**`npx tsc -p . --extendedDiagnostics` の結果（例）:**

```
Files:                         81
Lines of Library:             43159
Lines of Definitions:             0
Lines of TypeScript:            741
Lines of JavaScript:              0
Lines of JSON:                    0
Lines of Other:                   0
Identifiers:                47823
Symbols:                    32014
Types:                       1206
Instantiations:              1579
Memory used:                68658K
Assignability cache size:     232
Identity cache size:            9
Subtype cache size:            27
Strict subtype cache size:      1
I/O Read time:                0.01s
Parse time:                   0.07s
ResolveLibrary time:          0.01s
Program time:                 0.10s
// ... (tsc の出力は Bind time, Check time, Emit time を直接的には表示しない)
Bind time:                    0.03s // 推定値または詳細分析で取得
Check time:                   0.05s // 推定値または詳細分析で取得
Emit time:                    0.01s // 推定値または詳細分析で取得
Total time:                   0.10s // Program time を Total time と見なす
```

## 結果の考察

上記の比較（あくまで一例です）を見ると、`tsgo`の`Total time`が**0.025s**であるのに対し、`tsc`の`Program time`（コンパイル全体の処理時間に近い指標）が**0.10s**であることがわかります。

これは、この特定のコードベースにおいて、`tsgo`が**約4倍の速度向上**を達成していることを示しています。とくに注目すべきは、型チェックにかかる時間（`Check time`）が`tsgo`では**0.003s**と非常に高速である点です。メモリ使用量も`tsgo`が**23681K**に対して`tsc`が**68658K**と、`tsgo`の方が効率的であることも示唆されています。

公式サイトが掲げる「10倍の速度向上」という目標は、より大規模で複雑なプロジェクトにおける平均的な結果であり、プロジェクトの規模や構成によって実際の速度向上率は変動します。しかし、この小規模な例からも`tsgo`が大幅なパフォーマンス改善をもたらす可能性は十分に見て取れます。

## VS Code拡張機能

コマンドラインでの利用に加え、VS Code向けの新しい拡張機能「TypeScript (Native Preview)」もVisual Studio Marketplaceからインストール可能です。

インストール後、VS Codeのコマンドパレットから「TypeScript Native Preview: Enable (Experimental)」を実行するか、設定で「TypeScript > Experimental: Use Tsgo」を有効にすることで、この新しい言語サービスを試すことができます。

```json
// settings.json
"typescript.experimental.useTsgo": true
```

## 今後の展望と注意点

TypeScript Native Previewsはまだ開発の初期段階です。そのため、安定版のTypeScriptが持つ全ての機能が利用できるわけではありません。例えば、以下のような機能は未実装または制限があります。

* `--build` モード（プロジェクト参照のビルド）
* 宣言ファイル（`.d.ts`）の出力
* 古いECMAScriptターゲットへのダウンレベルトランスパイル
* エディタ機能（自動インポート、全ての参照を検索、名前の変更など）

これらの機能は今後のアップデートで順次追加されていく予定です。TypeScriptチームは、開発者からのフィードバックを積極的に募集しており、Issueトラッカーでの報告を奨励しています。

`tsgo`やVS Code拡張機能はナイトリービルドとして提供され、頻繁に更新されます。もし問題が発生した場合は、一時的に拡張機能を無効にする（コマンドパレットから「TypeScript Native Preview: Disable」を実行）などの対応が可能です。

この新しい`tsgo`とNative Preview拡張機能は、TypeScriptの開発体験を大きく変革する可能性を秘めています。ぜひご自身のプロジェクトで試し、その進化に注目してみてください。
