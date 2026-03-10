---
title: "Claude Codeを加速させる私の推しスキル・ツール・設定"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "ai", "raycast", "githubactions", "productivity"]
published: false
---

# はじめに

本記事は、Findy主催の「Claude Code 活用勉強会」（2025年3月）での登壇内容をまとめたものです。

AIエージェント時代の開発において、Claude Code（以下CC）を日常的に使い倒してきた中で発見した、**スキル・ツール・設定のTips**を紹介します。

---

# CCに早く正確に情報を伝えたい → CleanShot X

- **CleanShot X** でスクリーン上の任意の領域をスクショ → クリップボードにコピー → CCのチャット欄にそのまま貼り付け
- エラー画面、デザインカンプ、仕様書のスクショを即座にCCに渡せる
- **OCR機能**（光学文字認識）を使えば、コピー不可能なテキスト（PDF、画像内文字、他アプリのUI）も文字列として抽出してCCに貼り付け可能

> **OCRとは**: 画像に含まれる文字を認識してテキストデータに変換する技術。CleanShot XのOCR機能は、スクショを撮ると同時に文字列をクリップボードにコピーできる

https://cleanshot.com/

---

# GitHubのPRにスクショを貼り付けたい → upload-image-to-pr

- PRに画像がないとレビュアーが変更内容を把握しにくい
- しかし**GitHub APIには画像を直接アップロードするエンドポイントが存在しない**（2025年3月時点）
- 回避策として、PRコメント欄のファイルアップロード機能を経由してURLを発行する仕組みを自作

## upload-image-to-pr スキルの仕組み

1. **ブラウザ自動化**（Playwright MCP / Chrome DevTools MCP）でPRページを開く
2. コメント欄の `<input type="file">` に画像ファイルをアップロード
3. GitHubがアップロードした画像に `https://github.com/user-attachments/assets/...` という永続URLを発行
4. そのURLを取得し、**テキストエリアをクリアしてコメントは送信しない**（URLだけを使う）
5. `gh pr edit` CLIでPR説明文にURLを埋め込む

> **スキルとは**: `~/.claude/skills/` に置いたMarkdownファイルで定義する、CCのカスタム拡張機能。`/スキル名` というスラッシュコマンドで呼び出せる

```bash
# スキル呼び出し例
/upload-image-to-pr
```

---

# 現在の作業環境を迷わず把握したい → statusline

**ステータスライン**とは、CCの画面下部に任意の情報を表示できる機能。

## 設定方法

`~/.claude/settings.json` に以下を追加する：

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline.sh",
    "padding": 2
  }
}
```

`command` に指定したシェルスクリプトの標準出力がステータスラインに表示される。スクリプトにはCCの状態をJSON形式で渡してくれる（モデル名、コンテキスト使用量、カレントディレクトリなど）。

## 表示している情報

```
📂 ~/git/github.com/org/repo          ← カレントディレクトリ
🐙 repo-name │ 🌿 main +2 ~3          ← リポジトリ名 | ブランチ名 | diff
🧠 ████████░░░░░░░ 53% │ 💪 claude-sonnet-4-6  ← コンテキスト | モデル
💰 5h 23% (🔄 3pm) │ 7d 41% (🔄 3/15 3am)   ← 5時間/7日の使用量とリセット時刻
```

- **コンテキスト使用量**: 現在のセッションで使っているトークン量。90%以上で赤表示
- **5h/7dの使用量**: Claudeのサブスクリプション使用制限。キャッシュは360秒（APIコスト節約）
- Git情報はパフォーマンスのため5秒キャッシュ

---

# 複数の作業をCCに依頼したい → cmux

- **cmux** は複数のCCを並列で動かすためのmacOSネイティブターミナル

https://www.cmux.dev/

## 技術的特徴

- **libghosttyを内部ライブラリとして使用**している（Ghosttyのフォークではなく、レンダリングエンジンのみを拝借）
  - **libghosttyとは**: mitchellh氏が開発したターミナルエミュレータ「Ghostty」のレンダリングコンポーネント。WebブラウザがWebKitを使うように、cmuxはlibghosttyを使ってターミナル描画を行う
- **GPU加速レンダリング** → 高速・滑らか
- **Swift + AppKit** でネイティブ実装（Electronではない）→ 軽量
- ターミナルのキーバインドは `~/.config/ghostty/config` を読み込むため、Ghosttyユーザーはそのまま設定が流用できる

## エージェント開発向け機能

- サイドバーにタブ表示（Gitブランチ、作業ディレクトリ、ポート番号、通知テキスト）
- **ノーティフィケーションリング**: エージェントが入力待ちになったらタブが光って知らせてくれる
- インアプリブラウザ（ターミナルとブラウザを横並び表示）
- CLI・ソケットAPIによるスクリプタブルな制御

---

# 一つのリポジトリに対して並行作業したい → 複数クローン + Raycast

AIエージェント時代では、1つのリポジトリに対して複数のCCを並列稼働させたいケースが増える。

## なぜworktreeではなく複数クローンか

- `git worktree` はブランチ切り替えのオーバーヘッドがなく便利だが、設定ファイルやビルドキャッシュが共有されてしまうことがある
- 完全に独立した環境を確保したいので、**リポジトリを複数クローンして管理**するスタイルを選択

## ghq + peco で複数クローンを管理

**ghq** でクローン先をGitHub流の `org/リポジトリ名` 構造で統一管理：

```bash
# ghq: https://github.com/x-motemen/ghq
# peco: https://github.com/peco/peco
```

`g` コマンド1つでインタラクティブにリポジトリを移動：

```bash
function g() {
  local selected_dir=$(ghq list -p | peco --query "$LBUFFER")
  if [ -n "$selected_dir" ]; then
    cd "${selected_dir}"
  fi
}
```

## Raycastプラグインで自動採番クローン（自作）

ターミナルを開かずキーボードショートカットだけでクローンできるRaycastプラグインを自作。

1. RaycastのUI上でリポジトリURLとクローン数を指定
2. ghq管理ディレクトリ配下に `repo_1`, `repo_2`, `repo_3` ... と自動採番してクローン
3. `g` コマンドでpecoから選択して移動

> **Raycastとは**: macOSのランチャーアプリ。Alfred的なもの。Spotlight的なもの。拡張機能（プラグイン）をTypeScript/Reactで自作できる

- プラグインは発表後に公開予定

---

# アプリの起動を速くしたい → Raycastショートカット + スニペット

## Raycastでアプリを即起動

- 各アプリにグローバルショートカットを設定 → キー1発でフォーカス移動

## IDEをCLIから即起動

```bash
idea .   # IntelliJ IDEA でカレントディレクトリを開く
code .   # VS Code でカレントディレクトリを開く
cursor . # Cursor でカレントディレクトリを開く
```

## Raycastスニペットで長いコマンドを即入力

**スニペットとは**: 短い文字列を入力すると、設定した長い文字列に自動展開する機能。

例: `c;` と入力すると以下に展開

```bash
claude --dangerously-skip-permissions
```

> `--dangerously-skip-permissions` とは: CCの全ての権限確認をスキップして自動実行するフラグ。信頼できる環境でのみ使用する

---

# パッケージマネージャーを自動判定したい → ni

複数プロジェクトを扱っていると npm / yarn / pnpm / bun / deno が混在する。コマンドの違いを毎回調べるのはムダ。

**ni** はロックファイル（`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` 等）を検出して、自動的に適切なパッケージマネージャーのコマンドに変換してくれる。

```bash
# インストール (package.jsonに応じてnpm/yarn/pnpm/bunを自動選択)
ni

# パッケージを追加
ni vite
ni パッケージ名 -D  # 開発依存

# スクリプト実行
nr dev    # npm run dev / yarn dev / pnpm dev / bun dev
nr build  # npm run build / ...
```

https://github.com/antfu-community/ni

AIエージェントを使うようになり扱うリポジトリが増えたことで、このツールの恩恵が格段に大きくなった。

---

# AIエージェントにPRコメントを返信させる

CCにGitHubのPRレビューコメントを読ませ、返信・修正まで自動化できる。

詳細は以下の記事で解説済み：

https://findy-code.io/media/articles/aisaji-tonkotsuboy_com

---

# CCから画像を大量生成したい → nano-banana

**nano-banana**（nano-banana-2）は、Geminiの画像生成モデル（Imagen）をCLI経由で叩けるツール。これをCCのスキルとして登録すると、CCのチャット上から直接画像生成を指示できる。

- Geminiアプリを毎回立ち上げる手間が不要
- 画像を参照しながらの複雑な指示が可能
- 複数枚同時生成に対応

```bash
# スキル呼び出し
/nano-banana "ゆるマーカースタイルで、Reactのコンポーネント構造を示すインフォグラフィック"
```

スキルソース: https://github.com/kingbootoshi/nano-banana-2-skill

## 画像スタイル指定にはBanana X プロンプトパターン集

https://furoku.github.io/bananaX/projects/infographic-evaluation/?tab=history&id=biz_taste-yuru-marker

- お気に入りは **"ゆるマーカー"** スタイル

---

# 寝ながらスマホで作業を依頼したい → Remote Control

Claude Code Remote Controlを使うと、スマホのClaude.aiアプリからCCに作業指示を出せる。

- スマホからタスクを投げる → PCのCCが実行 → 結果をスマホで確認
- 寝ながら、移動中、風呂の中から開発作業を依頼できる

https://zenn.dev/ubie_dev/articles/claude-code-remote-control-intro

---

# もっとスキルを探したい → skillsmp.com + Grok検索

## スキルとは（補足）

`~/.claude/skills/` ディレクトリに置いたMarkdownファイルがスキルになる。フロントマターでツール権限を定義し、本文でCCへの指示を書く。

```markdown
---
name: my-skill
description: このスキルが呼び出される条件の説明
allowed-tools: Bash(git:*), Read, Write
---

# スキルの指示内容
...
```

## スキルの探し方

- **skillsmp.com**: スキルのマーケットプレイス。カテゴリ別にスキルを検索・インストールできる
  https://skillsmp.com/
- **Grok（X/Twitterの検索AI）**: 世界中のユーザーがXに投稿した有用なスキルをリアルタイム検索できる

---

# スキルを改善したい → skill-creator

- **skill-creator** を使うと、既存スキルの改善・新規スキルの作成をCCに依頼できる
- 「このスキルのここが使いにくい」「こういう動作も追加したい」とCCに伝えると、SKILL.mdを修正してくれる

---

# CCの情報をキャッチアップしたい

## 公式ドキュメントのAsk AI機能

Claude Codeの公式ドキュメントには **Ask AI** 機能が搭載されている。

- ドキュメントページ右下の「Ask AI」ボタンから、ドキュメント全体を対象にした質問ができる
- 「このオプションの使い方は？」「◯◯の設定はどこに書く？」など、ドキュメントを検索しなくてもチャットで即座に回答が得られる
- CCの設定項目やAPIリファレンスを調べるときに特に便利

https://docs.anthropic.com/ja/docs/claude-code/overview

## Xの情報をメンタル損耗なしに追う → RSSパイプライン

X（Twitter）を開くとタイムラインに飲み込まれてメンタルが削られる。ブックマークは流れる。

そこで以下のパイプラインを自作：

```
RSSHub (Docker)
  ↓ XのアカウントをRSSフィードに変換
Gemini API
  ↓ 英語ツイートを日本語に翻訳
ハイライト生成スクリプト
  ↓ 毎朝8:00に1日分のツイートをまとめ
nano-banana CLI (Gemini Flash)
  ↓ サムネイル画像を自動生成
Obsidian
  ← 日本語ハイライト + 画像をMarkdownで保存
```

- **RSSHubとは**: OSSのRSS生成サーバー。X、YouTube、GitHub等あらゆるサービスのRSSフィードを生成できる。Docker一発で起動
- Xを開かずに必要な情報だけを受け取れる

---

# まとめ

| カテゴリ | ツール・スキル | 概要 |
|----------|--------------|------|
| 情報入力 | CleanShot X | スクショ・OCRでCCに高速入力 |
| PR管理 | upload-image-to-pr | GitHub PRへ画像添付を自動化 |
| 環境把握 | statusline | CC下部に作業状況を常時表示 |
| 並列作業 | cmux | GPU加速・ネイティブのマルチターミナル |
| リポジトリ管理 | ghq + peco + Raycastプラグイン | 複数クローンを快適に管理 |
| 起動速度 | Raycast + スニペット | ショートカット・テキスト展開 |
| パッケージ管理 | ni | ロックファイルでPMを自動判定 |
| 画像生成 | nano-banana | CCからGemini画像生成を直接呼び出し |
| リモート操作 | Remote Control | スマホからCCに作業依頼 |
| スキル探索 | skillsmp.com + Grok | 便利スキルを効率的に発見 |
| ドキュメント参照 | 公式Docs Ask AI | ドキュメントをチャットで即検索 |
| 情報収集 | RSSHub + Geminiパイプライン | Xをノイズなしで情報収集 |

CCを「使う」から「使いこなす」へ。小さな改善の積み重ねが、開発体験を大きく変えます。
