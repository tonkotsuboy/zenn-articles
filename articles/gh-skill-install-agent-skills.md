---
title: "gh skillが登場。GitHub公式のスキル管理ツールにnpx skillsから乗り換えた"
emoji: "🐙"
type: "tech"
topics: ["github", "claudecode", "ai", "agentskills"]
published: true
publication_name: ubie_dev
---

AIエージェント向けのスキル（Agent Skills）、みなさんはどう管理していますか？

2026/04/16、GitHub公式CLIの`gh`に、スキルをパッケージ管理する新しいサブコマンド`gh skill`が追加されました。GitHubのリポジトリに公開されているスキルを`gh`経由でインストール・アップデート・公開できます。私はこれまで`npx skills`でスキルをインストール・管理してきましたが、`gh skill`の方が安全面でよさそうなので乗り換えることにしました。

本記事では、メリットや実際の動作を紹介します。


https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/


## 3行まとめ

- `gh skill install スキル`でスキルをインストールできる
- `gh skill publish スキル`で自作スキルを仕様に沿って公開できる
- 改ざん検知・バージョン固定・由来情報の埋め込みなど、サプライチェーン対策が組み込み済み

## 環境準備

GitHub CLI v2.90.0以上が必要です。私の環境の実行例は次のとおりです。

```bash
$ gh --version
// gh version 2.90.0 (2026-04-16)
// https://github.com/cli/cli/releases/tag/v2.90.0
```

`$ gh skill --help`でサブコマンドの一覧を確認すると、次のコマンドが使えることがわかります。

- `search`: GitHub上に公開されているスキルを検索する
- `preview`: インストール前に`SKILL.md`の中身を確認する
- `install`: スキルをローカルにインストールする
- `update`: インストール済みスキルを最新版に更新する
- `publish`: 自分のスキルをagentskills.io仕様でバリデーションしつつ、GitHubリリースとして公開する

## 実際に使ってみる

私が作ったXの投稿のバズり度を採点スキル「[x-impact-checker](https://github.com/tonkotsuboy/x-impact-checker)」を、`gh skill`で導入します。

### 1. `preview`で中身を先に確認

いきなりスキルをインストールする前に、`gh skill preview`で`SKILL.md`の中身を確認できます。怪しいスキルじゃないかの確認に便利です。

```bash
$ gh skill preview tonkotsuboy/x-impact-checker
```

確認ダイアログを進めると、スキルの中身が出力されます。この時点ではスキルはインストールされません。

![gh skill previewでSKILL.mdの中身を確認している画面](/images/gh-skill-install-agent-skills/gh-skill-preview-output.png)

`gh skill`がカバーしてくれるのは、スキルのインストール経路やバージョン情報まで。プロンプトインジェクション（AIへの命令文を不正に混ぜ込み、意図しない挙動を引き起こす攻撃）は、インストール前に中身を読まないと防げません。これは`gh skill`にかかわらず、スキルをインストールするときに行うべきことです。

> Skills are installed at your own discretion. They are not verified by GitHub and may contain prompt injections, hidden instructions, or malicious scripts. We strongly recommend inspecting the content of skills before installation,
> （意訳：スキルはGitHubによって検証されていません。プロンプトインジェクションや隠された命令、悪意のあるスクリプトが含まれている可能性があります。インストール前にスキルの中身を確認することを強く推奨します）
>
> https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/

実際には、中身を読んで判断するのも大変なので、`gh skill preview`で読んだ中身をチェックするスキルを作って運用するのがいいのではないかと考えています。

### 2. `install`でスキルをインストール

`gh skill install スキル`でインストールします。対話プロンプトでどのエージェントに入れるか、ユーザー全体かプロジェクト単位かを聞かれます。


```bash
$ gh skill install tonkotsuboy/x-impact-checker
```

![gh skill installでインストール先エージェントを選ぶ対話プロンプト](/images/gh-skill-install-agent-skills/gh-skill-install-select-agent.png)


![Claude Code向けに~/.claude/skills配下へインストールされた様子](/images/gh-skill-install-agent-skills/gh-skill-install-complete.png)


バージョン指定を省略したときは、最新のタグが優先的に選ばれます。

特定のバージョンに固定してスキルをインストールすることも可能です。次の例では、v2.0.5を指定してインストールしています。

```bash
$ gh skill install tonkotsuboy/x-impact-checker --pin v2.0.5
```


### 3. 配置先の確認

Claude Code向けにユーザー全体でインストールした場合、`~/.claude/skills/`直下に入ります。

```bash
$ ls ~/.claude/skills/x-impact-checker/
SKILL.md  references
```

`npx skills`の場合はスキルファイルは `~/.agents` に配置され、`~/.claude/skills/`にはシンボリックリンクが配置される挙動でしたが、`gh skill`の場合は指定したエージェント分スキルファイルがコピーされます。

なお、スキルの仕様を公開している[agentskills.io](https://agentskills.io/home)では、スキルをローカルのどこに置くかまでは規定していません。`npx skills`は自前ディレクトリ`~/.agents/skills/`を正とする設計、`gh skill`は各エージェントの純正ディレクトリ（Claude Codeなら`~/.claude/skills/`）に直接置く設計となっています。

### 4. `update`でスキルを更新

`gh skill update`で、インストール済みのスキルをまとめて最新版に更新できます。

```bash
$ gh skill update
```

`--dry-run`を付けると、実際の書き換えは行わず、更新があるかだけを確認できます。

## スキルが勝手に書き換わらない仕組み

スキルはAIへの命令書そのものです。入れたあとに中身がこっそり書き換わると、気づかないうちに意図しない指示でAIが動きます。配布元のリポジトリ乗っ取りや、タグを別コミットに付け替えるforce pushが典型的なケースです。

`gh skill`は、一度入れたスキルが知らないうちに書き換わらないように、4つの仕組みを用意しています。

- Immutable Releases: 公開後のリリース内容をリポジトリの管理者でも書き換えられなくする設定
- Tree SHA による検知: スキルフォルダ全体のハッシュ値（Tree SHA※）を`SKILL.md`に記録し、リモートの中身が変わると検出できる
- Version Pinning: `--pin`で特定のタグ・コミットに固定する。`update`でも自動更新されない
- Portable Provenance: 由来情報（provenance / プロベナンス）を`SKILL.md`自体に埋め込むので、ファイルを別環境にコピーしても由来情報がついて回る

※ Tree SHAは、Gitがディレクトリ配下のファイルツリー全体を一意に識別するハッシュ値です。ファイルの中身が1ビットでも変わると値も変わるので、改ざん検知に使えます。


### スキルのfrontmatterに書き込まれる由来情報（provenance）

`gh skill install`を実行したときには、`SKILL.md`のfrontmatter（冒頭の`---`で囲まれた部分）に由来情報（provenance / プロベナンス）が書き込まれます。

次の例は、`gh skill install`で入れた`x-impact-checker`の`SKILL.md`のfrontmatterです。

```bash
---
description: 'Analyze X (Twitter) posts for viral potential using the actual recommendation algorithm. ...'
metadata:
    github-path: skills/x-impact-checker
    github-ref: refs/tags/v2.0.6
    github-repo: https://github.com/tonkotsuboy/x-impact-checker
    github-tree-sha: f7a74cc1c1d6fc8650d00a7ad441d330e8f11e62
name: x-impact-checker
---
```

`metadata:`ブロックの4行が由来情報（provenance）の実体です。

- `github-path`はリポジトリ内でのスキルのパス
- `github-ref`はどのタグ・ブランチ由来か（`refs/tags/v2.0.6`ならv2.0.6タグ固定）
- `github-repo`はリポジトリのURL
- `github-tree-sha`はフォルダ全体のハッシュ値（改ざん検知用）

この情報が`SKILL.md`自体に埋まっているので、ファイルを別のマシンにコピーしても由来情報がついて回ります。`gh skill update`はここを読んで、リモートで中身が更新されているかを判定します。

## `gh skill publish`でスキルを公開できる

`gh skill publish パッケージ`は、スキルを公開するためのコマンドです。ローカルの`SKILL.md`群が[agentskills.io](https://agentskills.io/)仕様を満たしているかをバリデーションし、通ったら対話形式でGitHubリリースを作成します。

## agentskills.io仕様への準拠チェック

agentskills.ioは、`SKILL.md`が備えるべき命名規則やfrontmatterの項目を定めた共通仕様です。Claude Code、Cursor、Codex、GitHub Copilot、Gemini CLI、Antigravityなどのエージェントが、同じ仕様のスキルを読めるようになっています。

`gh skill publish --help`によると、次のようなチェック内容があることがわかります。`gh skill publish`をつかうことで、この仕様に沿ったスキルの公開が可能になります。

- スキル名がagentskills.ioの命名規則に従っているか
- スキル名とディレクトリ名が一致しているか
- `name` / `description`のfrontmatterがあるか
- `allowed-tools`が配列ではなく文字列か
- install metadata（`metadata.github-*`）が残っていないか

## リポジトリ側のセキュリティ設定もチェックする

`gh skill publish`は、スキル本体のバリデーションに加えて、GitHubリポジトリ側のセキュリティ設定が有効になっているかもチェックします。必須ではないものの、サプライチェーン※対策として推奨されているのが次の3つです。

- tag protection: タグを後から別コミットに付け替えられるのを防ぐ
- secret scanning: コミットに秘密情報（APIキー等）が紛れ込んでいないかのスキャン
- code scanning: コードの脆弱性スキャン

※ サプライチェーンとは、ライブラリやスキルといった外部の部品が、作られてから利用者の手元に届くまでの流れのこと。この流れのどこかに細工を仕込んで利用者を狙う攻撃を「サプライチェーン攻撃」と呼びます。

私が公開を試したときも、tag protectionが有効になっていないリポジトリで実行するとwarningが出ました。次のようにtag protectionを有効化することで解決しました。

![GitHubリポジトリ設定画面でtag protectionのRulesetを有効化する](/images/gh-skill-install-agent-skills/github-tag-protection-rule.png)


# さいごに


`gh skill`を使うと、スキルのインストール・更新・公開までを安全に行なえます。私は`npx skills`を気に入って使っていましたが、昨今のサプライチェーン攻撃などを見ていると、安全な管理方法がないかを考えていました。今回GitHubがリリースした`gh skill`はその一つの答えとなるもので、スキル管理の新しい選択肢の一つとなりうるでしょう。個人的にはスキルのインストール・更新のみならず、公開時のセキュリティチェックまでやってくれるのが助かりました。

男は黙って`gh skill`を使いましょう。

https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/

https://github.com/tonkotsuboy/x-impact-checker

