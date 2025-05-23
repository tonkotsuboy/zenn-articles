
---

title: "TypeScript 7に向けて：10倍高速化を目指す「tsgo」を実際に検証した"
emoji: "✂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "nodejs", "javascript"]
published: true
publication_name: ubie_dev
---

本日2025/05/23、MicrosoftのTypeScriptチームは、TypeScriptのGO実装のコンパイラーを「**TypeScript Native Previews**」として公開しました。「tsgo」という名前で、将来リリースされるTypeScript 5.7では、従来のTypeScriptのコンパイラー`tsc`と同様の働きをします。tsgoを使うと型チェックやコンパイル速度が10倍になります。

本記事では、早速tsgoをインストールし、型チェックやコンパイル速度の速度が本当に10倍になるのかを検証します。

## TypeScript 7でコンパイラがGO言語になる

現在、TypeScriptのコンパイラは◯◯で書かれていますが、それをGo言語で書き換えるプロジェクトが進行中です。TypeScript 7でリリース予定で、コンパイル速度や型チェック速度が10倍という劇的な速度で改善します。

TypeScriptの利用が拡大し、プロジェクトの規模が大きくなるにつれて、コンパイル時間や型チェックのパフォーマンスが課題となることがありました。この課題に対処するため、TypeScriptチームはコンパイラをネイティブコードに移植する取り組みを進めてきました。

その目的は、単にネイティブコンパイル言語であるGoを使用するだけでなく、共有メモリ並列処理や並行処理を最大限に活用し、**ほとんどのプロジェクトで10倍の速度向上**を達成することです。これにより、開発時のフィードバックがより迅速になり、開発者の生産性向上に貢献します。

「[A 10x Faster TypeScript](https://devblogs.microsoft.com/typescript/typescript-native-port/)」に詳しくまとまっていますが、つぎのとおり早くなるとのことです。

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

tsgoを試す環境の作り方です。まずは今回新しくリリースされた「@typescript/native-preview」をインストールします。

```ts
npm install -D @typescript/native-preview
```

このパッケージは `tsgo`という実行ファイルを提供します。

```ts
npx tsgo -v
// Version 7.0.0-dev.20250523.1
```

`tsgo`によるコンパイルを試すために、TypeScriptパッケージをインストールします。

```bash
npm install -D typescript
```

tsconfig.jsonを作ります。`tsgo`コマンドで作るため、initコマンドを実行してみます。しかし、エラーとなります。

▼ 動作しない

```bash
 npx tsgo --init
```

tsgoはまだプレビュー版であり、--initのようなプロジェクト初期化のための機能は提供されてないようです。したがって、`tsc`コマンドで作成します。

```bash
npx tsc --init
```

tsconfig.jsonが作成されます。今回は検証のため、次のような設定にしました。

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

# コンパイル速度の比較：`tsc` vs `tsgo`

実際に、ある程度の長さのTypeScriptコードを`tsc`と`tsgo`でコンパイルし、その速度を比較してみましょう。

```typescript

```

---

上記のコードをコンパイルする際の計測結果は次のとおりです。`--extendedDiagnostics`オプションとは、コンパイルの詳細な情報を出力するオプションです。

**`npx tsgo -p . --extendedDiagnostics` の結果:**

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

**`npx tsc -p . --extendedDiagnostics` の結果:**

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

```

## 結果の考察

上記の比較を見ると、`tsgo`の`Total time`が**0.025s**であるのに対し、`tsc`の`Program time`（コンパイル全体の処理時間に近い指標）が**0.10s**であることがわかります。

これは、この特定のコードベースにおいて、`tsgo`が**約4倍の速度向上**を達成していることを示しています。特に注目すべきは、型チェックにかかる時間（`Check time`）が`tsc`では明確な数値がないものの、`tsgo`では**0.003s**と非常に高速である点です。メモリ使用量も`tsgo`が**23681K**に対して`tsc`が**68658K**と、`tsgo`の方が効率的であることも示されています。

公式サイトの「10倍の速度向上」という目標は、より大規模で複雑なプロジェクトにおける平均的な結果であり、プロジェクトの構成によって結果は異なります。しかし、この例からも`tsgo`が大幅なパフォーマンス改善をもたらす可能性が示唆されます。

---

## 今後の展望

TypeScript Native Previewsはまだ開発の初期段階にあり、`--build`モードや宣言ファイル出力、自動インポートや「すべての参照を検索」といったエディター機能など、既存のTypeScriptが持つ一部の機能はまだ実装されていません。

しかし、毎晩の公開と継続的な開発により、これらの機能も順次追加されていく予定です。TypeScriptチームは、開発者からのフィードバックを積極的に求めており、この高速な新コンパイラがTypeScript開発体験をどのように変革していくか、今後の動向に期待が寄せられます。

あなたのプロジェクトでも、この新しい`tsgo`を試してみてはいかがでしょうか。
