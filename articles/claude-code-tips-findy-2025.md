---
title: "Claude Codeを加速させる私の推しスキル・ツール・設定"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "ai", "raycast", "githubactions", "productivity"]
published: false

---

# Claude Codeを素早く起動したい

Raycastショートカット + スニペットを使う

- ターミナルの起動
  - Raycastを使い、キー1発でアプリを起動できるようにする
- Claude Codeの起動
  - Raycastのスニペットを使い、短い入力（例`c;`）でClaude Codeを起動できるように
- 最初のタスクの依頼
  - `claude 依頼したいタスク` と入力すると、Claude Code起動後にタスクが実行される


https://www.raycast.com/

https://www.raycast.com/core-features/snippets


## 補足

- Raycastとは: ランチャーアプリ
- **スニペットとは**: 短い文字列を入力すると、設定した長い文字列に自動展開する機能。
`--dangerously-skip-permissions` とは: Claude Codeのすべての権限確認をスキップして自動実行するフラグ


# 複数の作業をClaude Codeに依頼したい

cmuxを使う


- **cmux** は複数のClaude Codeを並列で動かすためのmacOSネイティブターミナル
- サイドバーにタグを表示でき、Claude Codeのタスクが完了したら通知される
- 複数の作業をClaude Codeに依頼したいときに便利

https://www.cmux.dev/


# Claude Codeに画像や画像内文字を伝えたい → CleanShot X

[https://cleanshot.com/](https://cleanshot.com/)

- スクショ
  - **CleanShot X** でスクリーン上の任意の領域をスクショしてClaude Codeに貼り付け
- コピペできないテキスト
  - PDF、画像内文字、他アプリのUIなどを貼り付けたい
  - OCR機能を使ってテキストをコピーできる

---

# GitHubのPRにスクショを貼り付けたい


自作スキル「upload-image-to-pr」を使う


- PRのdescriptionに画像を貼り付けられる自作スキル
- **GitHub APIには画像を直接アップロードするエンドポイントが存在しない**
- 仕組み
  - Chrome DevTools MCPかPlaywrightを使いページを開く
  - コメント欄に一度画像をアップロード
  - 発行されたURLをPRに貼り付け


https://github.com/tonkotsuboy/github-upload-image-to-pr

## 補足

1. **ブラウザ自動化**（Playwright MCP / Chrome DevTools MCP）でPRページを開く
2. ブラウザにログインしておく
3. コメント欄の `<input type="file">` に画像ファイルをアップロード
4. GitHubがアップロードした画像に `https://github.com/user-attachments/assets/...` という永続URLを発行
5. そのURLを取得し、**テキストエリアをクリアしてコメントは送信しない**（URLだけを使う）
6. `gh pr edit` CLIでPR説明文にURLを埋め込む

> **スキルとは**: `~/.claude/skills/` に置いたMarkdownファイルで定義する、Claude Codeのカスタム拡張機能。`/スキル名` というスラッシュコマンドで呼び出せる

```bash
# スキル呼び出し例
/upload-image-to-pr
```

---

# 現在の作業環境を迷わず把握したい

ステータスラインを使う

- **ステータスライン**とは、Claude Codeの画面下部に任意の情報を表示できる機能
- 次の内容を表示している

```
📂 ~/git/github.com/org/repo          ← カレントディレクトリ
🐙 repo-name │ 🌿 main +2 ~3          ← リポジトリ名 | ブランチ名 | diff
🧠 ████████░░░░░░░ 53% │ 💪 claude-sonnet-4-6  ← コンテキスト | モデル
💰 5h 23% (🔄 3pm) │ 7d 41% (🔄 3/15 3am)   ← 5時間/7日の使用量とリセット時刻
```

- `/statusline 表示したい内容` を指定すればOK 
## 補足

-  `~/.claude/settings.json` に設定値が追加される

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline.sh",
    "padding": 2
  }
}
```

- 設定の実体は `statusline.sh`
- 使用量表示のためにAPIを叩いているが、利用規約的にはグレーゾーンらしい

# 一つのリポジトリをたくさんクローンして並行作業したい

Raycastの自作ツール「Multi-Folder Git Clone」を使う

- AIエージェント時代では、1つのリポジトリに対して複数のClaude Codeを並列稼働させたい
- worktreeはNOT FOR MEだった
  - worktreeを使うのなら`npx vibe-kanban` がよさそう
- 同じリポジトリを複数クローンにしつつ、自動採番するツールを作った
- Claude Codeのベスト・プラクティスでも推奨されている（fact check）

https://github.com/tonkotsuboy/multi-folder-git-clone



## 「Multi-Folder Git Clone」の仕組み

1. RaycastのUI上でリポジトリURLとクローン数を指定
2. ghq管理ディレクトリ配下に `repo_1`, `repo_2`, `repo_3` ... と自動採番してクローン
3. 後続のghq + pecoでリポジトリ移動

# リポジトリ間を素早く移動したい

## なぜworktreeではなく複数クローンか

- `git worktree` はブランチ切り替えのオーバーヘッドがなく便利だが、設定ファイルやビルドキャッシュが共有されてしまうことがある
- 完全に独立した環境を確保したいので、**リポジトリを複数クローンして管理**するスタイルを選択
- `g` コマンド1つでインタラクティブにリポジトリを移動できる


ghq + pecoを使う

- ghq
  - クローン先をGitHub流の `org/リポジトリ名` 構造で統一管理する
- peco
  - ◯◯をするツール

https://github.com/x-motemen/ghq
https://github.com/peco/peco


## pecoの設定


```bash
function g() {
  local selected_dir=$(ghq list -p | peco --query "$LBUFFER")
  if [ -n "$selected_dir" ]; then
    cd "${selected_dir}"
  fi
}
```


# npm, pnpm, yarn等のパッケージマネージャーの違いに悩みたくない

niを使う


- 複数プロジェクトを扱っていると npm / yarn / pnpm / bun / deno が混在する
  - コマンドの違いを毎回調べるのはムダ。
- niを使えば、一つのコマンドで環境に応じたパッケージマネージャーを実行してくれる

例


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


## **ni**のパッケージマネージャー抽出の仕組み


**ni** はロックファイル（`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` 等）を検出して、自動的に適切なパッケージマネージャーのコマンドに変換してくれる。


# Claude Codeから画像を大量生成したい


nano-banana-2-skillを使う

- nano-bananaは高品質だが、画像を作るために、Geminiアプリから起動するのは面倒
- nano-banana-2-skillを使えば、Claude Code上から画像をつくれる
- 試行錯誤して画像ができる修正
- 複数枚同時生成に対応できる
- 画像スタイル指定は、プロンプト集を使う
  - 日本語インフォグラフィックにはbananaXが便利 

呼び出し例

```bash
TODO
```


https://github.com/kingbootoshi/nano-banana-2-skill

https://furoku.github.io/bananaX




# もっとスキルを探したい


スキルマーケットプレイス（skillsmp.com）や、最近はGrok検索も便利


- **skillsmp.com**: スキルのマーケットプレイス。カテゴリ別にスキルを検索・インストールできる
[https://skillsmp.com/](https://skillsmp.com/)
- **Grok（X/Twitterの検索AI）**: 世界中のユーザーがXに投稿した有用なスキルをリアルタイム検索できる
- Slack AI: 社内の人が活用してる生のスキルを検索


## スキルとは（補足）

- `~/.claude/skills/` ディレクトリに置いたMarkdownファイルがスキルになる
- 自然言語で書ける
- 自動で呼び出してくれる

```markdown
---
name: my-skill
description: このスキルが呼び出される条件の説明
allowed-tools: Bash(git:*), Read, Write
---

# スキルの指示内容
...
```


# Claude Codeの情報をキャッチアップしたい

公式ドキュメントのAskAI機能とX RSS化

## 公式ドキュメントのAsk AI機能

- Claude Codeの公式ドキュメントには **Ask AI** 機能が搭載されている
- ドキュメントページ右下の「Ask AI」ボタンから、ドキュメント全体を対象にした質問ができる
- 野良の記事を頑張って探すより、一次ソースを使って調べるほうがわかりやすい
- 私はこのAI機能で、`[Pasted text #1 +19 lines]`みたいに展開されてしまった長いテキストの編集方法を知った
  - control G

https://docs.anthropic.com/ja/docs/claude-code/overview


## XのRSS化

- X（Twitter）を開くメンタルが削られる
  - 椅子から転げ落ちた人だらけ
  - 誰かが誰かを攻撃してる 
  - 子どもと猫の写真だけ流れてほしい…
- ブックマークやリスト機能は、どんどん更新されて見逃しちゃう
- そこで以下のパイプラインを自作：

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


（画像）


# あるコンテキストを持ったまま、複数の会話をしたい

`/fork`や`/rewind`、`ccresume`を使う

- `/fork`: 会話を分岐させる
- `/rewind`: 会話を巻き戻せる
- `ccresume`: 過去の会話を見やすくできるツール


# 寝ながらスマホで作業を依頼したい → Remote Control

Claude Code Remote Controlを使うと、スマホのClaude.aiアプリからClaude Codeに作業指示を出せる。

- スマホからタスクを投げる → PCのClaude Codeが実行 → 結果をスマホで確認
- 寝ながら、移動中、風呂の中から開発作業を依頼できる

https://zenn.dev/ubie_dev/articles/claude-code-remote-control-intro

# スキルを改善したい → skill-creator

- **skill-creator** を使うと、既存スキルの改善・新規スキルの作成をClaude Codeに依頼できる
- 「このスキルのここが使いにくい」「こういう動作も追加したい」とClaude Codeに伝えると、SKILL.mdを修正してくれる
