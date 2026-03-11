---
title: "Claude Codeを加速させる私の推しスキル・ツール・設定（Findyイベント登壇資料）"
emoji: "🌹"
type: "tech"
topics: ["claudecode", "ai", "raycast", "githubactions", "productivity"]
published: true
publication_name: ubie_dev

---

本記事は、Findyイベント「鹿野さんに聞く！Claude Codeをさらに加速させる私の推しツール」の登壇資料です。

https://findy.connpass.com/event/384179/


Claude Codeの開発を加速させるための推しスキル・ツール・設定を紹介しています。


# Claude Codeを素早く起動したい

Raycastショートカット + スニペットを使う


## ターミナルの起動

- Raycastとは、様々な機能を呼び出せるランチャーアプリ
  - Alfredやspotlightみたいなもの
- 最近、Windows Betaが出た
- Raycastを使い、hotkeyキー1発でアプリを起動できるようにする
  -  Windowsでも動作する
- 私は `⌥⌘T` で起動させている
  - ターミナルのT

![ターミナルの起動](/images/claude-code-tips-findy-2026/raycast-hotkey.png)

https://www.raycast.com/


## Claude Codeの起動

- スニペットを使う
- 私はRaycastのスニペット機能を使っている
  - （おそらく）macOS限定
- 短い入力（例`c;`）でClaude Codeを起動できるように


![Claude Codeの起動](/images/claude-code-tips-findy-2026/raycast-snippet.png)



## 最初のタスクの依頼

- `claude 依頼したいタスク` と入力すると、Claude Code起動後にタスクが実行される
- Claude Code起動待ちの短い時間を撲滅

1. タスクを依頼する

![タスクを依頼する](/images/claude-code-tips-findy-2026/raycast-task-request.png)

2. Claude Codeが起動し、タスクが始まる

![Claude Codeが起動し、タスクが始まる](/images/claude-code-tips-findy-2026/raycast-task-started.png)


https://www.raycast.com/

https://www.raycast.com/core-features/snippets


# Claude Codeに画像や画像内文字を伝えたい

CleanShot Xを使う

- スクショ
  - CleanShot X でスクリーン上の任意の領域をスクショしてClaude Codeに貼り付け
- コピペできないテキスト
  - OCR機能を使ってテキストをコピーできる
  - PDF、画像内文字、他アプリのUIなどの中にある文字を貼り付けたい
  - 画像を貼り付けて読み取らせるよりもトークン使用量が少ない
- CleanShot XはmacOS専用。Windowsの情報求む

https://cleanshot.com/



# GitHubのPRにスクショを貼り付けたい

自作スキル「upload-image-to-pr」を使う

- PRのdescriptionに画像を貼り付けられる自作スキル
- GitHub APIには画像を直接アップロードするエンドポイントが存在しない
- 仕組み
  - Chrome DevTools MCPかPlaywrightを使いページを開く
  - コメント欄に一度画像をアップロード
  - 発行されたURLをPRに貼り付け


実行例

![実行例1](/images/claude-code-tips-findy-2026/upload-image-to-pr-1.png)

![実行例2](/images/claude-code-tips-findy-2026/upload-image-to-pr-2.png)

![実行例3](/images/claude-code-tips-findy-2026/upload-image-to-pr-3.png)


https://github.com/tonkotsuboy/github-upload-image-to-pr

## 詳細補足

1. ブラウザ自動化（Playwright MCP / Chrome DevTools MCP）でPRページを開く
2. ブラウザにログインしておく
3. コメント欄の `<input type="file">` に画像ファイルをアップロード
4. GitHubがアップロードした画像に `https://github.com/user-attachments/assets/...` という永続URLを発行
5. そのURLを取得し、テキストエリアをクリアしてコメントは送信しない（URLだけを使う）
6. `gh pr edit` CLIでPR説明文にURLを埋め込む


# 現在の作業環境を迷わず把握したい

ステータスラインを使う

- ステータスラインとは、Claude Codeの画面下部に任意の情報を表示できる機能
- `/statusline 表示したい内容` を指定すれば、Claude Codeが設定してくれる
- 私の表示は次のとおり


![ステータスライン](/images/claude-code-tips-findy-2026/statusline.png)


```
📂 ~/git/github.com/org/repo          ← カレントディレクトリ
🐙 repo-name │ 🌿 main +2 ~3          ← リポジトリ名 | ブランチ名 | diff
🧠 ████████░░░░░░░ 53% │ 💪 claude-sonnet-4-6  ← コンテキスト | モデル
```



## 補足

- `~/.claude/settings.json` に設定値が追加される
- 設定の実体は `statusline.sh`


```json
{
  "statusLine": {
    "type": "command",
    "command": "bash ~/.claude/statusline.sh",
    "padding": 2
  }
}
```


# 一つのリポジトリをたくさんクローンして並行作業したい

Raycastの自作ツール「Multi-Folder Git Clone」を使う

- AIエージェント時代では、1つのリポジトリに対して複数のClaude Codeを並列稼働させたい
- worktreeはNOT FOR MEだった
  - エディターで追いづらい、見づらい
  - worktreeを使うのなら`npx vibe-kanban` がよさそう
- 同じリポジトリを複数フォルダにクローンしつつ、自動採番するツールを作った


![Multi-Folder Git Clone 1](/images/claude-code-tips-findy-2026/multi-clone-1.png)


![Multi-Folder Git Clone 2](/images/claude-code-tips-findy-2026/multi-clone-2.png)

![Multi-Folder Git Clone 3](/images/claude-code-tips-findy-2026/multi-clone-3.png)



https://github.com/tonkotsuboy/multi-folder-git-clone


# リポジトリ間を素早く移動したい

ghq + pecoを使う

- ghq
  - `ghq get` でクローンすると `~/git/github.com/org/repo` のような統一構造で保存される（保存先は `ghq.root` の設定値に依存。デフォルトは `~/ghq`）
  - 「どこにクローンするか」を毎回考えなくていい
  - `ghq list` で管理下のリポジトリ一覧をパス形式で出力できる
- peco
  - ghqが管理するリポジトリ一覧をインタラクティブに絞り込み検索できるフィルタツール
  - 選んだディレクトリに即移動
- 私は前述の自作ツールでクローンしており、ghqでクローンはしていない。
  - ただし `ghq list` は設定したルートディレクトリを自動スキャンしてくれるため、クローン方法を問わず既存のリポジトリを認識してくれる。これをもとにpecoで検索してる

次の設定を、`~/.zshrc` にしている

```bash
function g() {
  local selected_dir=$(ghq list -p | peco --query "$LBUFFER")
  if [ -n "$selected_dir" ]; then
    cd "${selected_dir}"
  fi
}
```

https://github.com/x-motemen/ghq

https://github.com/peco/peco

https://zenn.dev/obregonia1/articles/e82868e8f66793


# Claude Codeから画像を大量生成したい

nano-banana-2-skillを使う

- nano-bananaは高品質だが、画像を作るために、Geminiアプリから起動するのは面倒
- nano-banana-2-skillを使えば、Claude Code上から画像をつくれる
- 試行錯誤しながら画像を修正できる
- 複数枚同時生成に対応できる
- 画像スタイル指定は、プロンプト集を使う
  - 日本語インフォグラフィックにはbananaXが便利

呼び出し例


![nano-banana呼び出し例1](/images/claude-code-tips-findy-2026/nano-banana.png)

![nano-banana呼び出し例2](/images/claude-code-tips-findy-2026/nano-banana-3.png)


![nano-banana呼び出し例3](/images/claude-code-tips-findy-2026/nano-banana-1.png)

![nano-banana呼び出し例4](/images/claude-code-tips-findy-2026/nano-banana-2.png)


https://github.com/kingbootoshi/nano-banana-2-skill

https://furoku.github.io/bananaX/projects/infographic-evaluation/




# 手戻りを減らしたい

Claude Codeのfeature-devプラグインを使う

- 複雑な仕様の場合、いきなり開発せず、まず壁打ちしてから始めたい
- `/feature-dev 作りたい機能` と入力すると、コードベースの現状を把握し、必要に応じて深堀り質問してくれる
- 要件が固まってからアーキテクチャ設計→実装→レビューまで行う
- 各フェーズで確認を挟むので「勝手に突き進んで手戻り」が起きにくい
- 複雑な機能開発に向いている。単純なバグ修正には不要


呼び出し例

![feature-devプラグインの呼び出し例](/images/claude-code-tips-findy-2026/feature-dev.png)



7フェーズの流れ

```
1. Discovery        … 要件の明確化・制約の洗い出し
2. Codebase探索     … 既存コードのパターンや類似機能を並列で調査
3. 質問             … 曖昧な点をすべて質問し、回答を待ってから次へ
4. アーキテクチャ設計 … 複数の設計案を提示し、トレードオフを比較
5. 実装             … 承認後にコーディング開始
6. コードレビュー    … 品質・バグ・規約の3観点で並列レビュー
7. サマリー         … 変更内容と次のステップをまとめ
```

https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev


# あるコンテキストを持ったまま、複数の会話をしたい

`/btw`、`/fork`、`/rewind`、`ccresume`を使う

- `/btw`: メインタスクを中断せずにサイドで素早く質問
- `/fork`: 会話を分岐させる
- `/rewind`: 会話を巻き戻せる


`/btw` の呼び出し例

![/btwコマンドの呼び出し例](/images/claude-code-tips-findy-2026/btw-command.png)


# 寝ながらスマホで作業を依頼したい → Remote Control

Claude Code Remote Controlを使うと、スマホのClaude.aiアプリからClaude Codeに作業指示を出せる。

- スマホからタスクを投げる → PCのClaude Codeが実行 → 結果をスマホで確認
- 寝ながら、移動中、風呂の中から開発作業を依頼できる

https://zenn.dev/ubie_dev/articles/claude-code-remote-control-intro


# 複数の作業をClaude Codeに依頼したい

cmuxを使う


- cmux は複数のClaude Codeを並列で動かすためのmacOSネイティブターミナル
- サイドバーにタグを表示でき、Claude Codeのタスクが完了したら通知される
- 複数の作業をClaude Codeに依頼したいときに便利

https://www.cmux.dev/



# npm, pnpm, yarn等のパッケージマネージャーの違いに悩みたくない

niを使う

- 複数プロジェクトを扱っていると npm / yarn / pnpm / bun / deno が混在する
  - コマンドの違いを毎回調べるのはムダ
- niを使えば、一つのコマンドで環境に応じたパッケージマネージャーを実行してくれる
- niはロックファイル（`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` 等）を検出して、自動的に適切なパッケージマネージャーのコマンドに変換してくれる


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

https://github.com/antfu-collective/ni



# もっとスキルを探したい

スキルマーケットプレイス（skillsmp.com）や、最近はGrok検索も便利

- skillsmp.com: スキルのマーケットプレイス。カテゴリ別にスキルを検索・インストールできる
- Grok（X/Twitterの検索AI）: 世界中のユーザーがXに投稿した有用なスキルをリアルタイム検索できる
- Slack AI: 社内の人が活用してる生のスキルを検索


https://skillsmp.com/

https://grok.com/


# Claude Codeを理解したい

公式ドキュメントのAskAI機能が便利

- Claude Codeの公式ドキュメントには Ask AI 機能が搭載されている
- ドキュメントページ右下の「Ask AI」ボタンから、ドキュメント全体を対象にした質問ができる
- 野良の記事を頑張って探すより、一次ソースを使って調べるほうがわかりやすい


![公式ドキュメントのAskAI機能](/images/claude-code-tips-findy-2026/docs-ask-ai.png)


https://docs.anthropic.com/ja/docs/claude-code/overview


# Claude Codeの最新情報を追いたい

XをRSS化し、obsidianに貯める

- Xでは開発者がTipsや最新情報を発信している
- Xのタイムラインを眺めると無駄情報が多く、メンタルが削られる
- ブックマークやリスト機能は、どんどん更新されて見逃しちゃう
- 以下のパイプラインを自作した

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


# 最後に

ここで紹介したものはほんの一部。
ぜひ皆さんの推しツール・スキル・設定を教えてください！知りたいです😍

また、Ubieでは、エンジニアはもちろんデザイナー、PO、管理職、人事にいたるまで、全員がClaude Codeを導入し、あらゆる業務を行っています。ぜひ一緒にUbieでAIネイティブな生活をしましょう。

https://recruit.ubie.life/
