# Zenn Articles

[Zenn.dev](https://zenn.dev/tonkotsuboy_com)で公開している技術記事の管理リポジトリです。

## 概要

このリポジトリでは、主にJavaScript/TypeScript、CSS、Web技術、AIエージェントに関する記事を管理しています。

- 📝 公開記事: https://zenn.dev/tonkotsuboy_com
- 🌐 公式サイト: https://kano.codes/
- 🐦 X (Twitter): https://x.com/tonkotsuboy_com

## ディレクトリ構成

```
.
├── articles/       # 日本語の技術記事
└── images/         # 記事用画像(記事スラッグごとに整理)
```

## 開発環境のセットアップ

### 必要な環境

- Node.js (Voltaで自動管理)

### インストール

```bash
npm install
```

### 記事のプレビュー

```bash
npm run preview
```

ブラウザで http://localhost:8000 にアクセスすると、記事をプレビューできます。

### Lint

```bash
npm run lint        # 記事の文章校正チェック
npm run lint:fix    # 自動修正
```

## 記事の作成

### 新規記事の作成

```bash
npx zenn new:article
```

### 記事一覧の確認

```bash
npx zenn list:articles
```

## コントリビューション

誤字脱字の修正や改善提案など、PRを歓迎します!

- 記事の修正や改善提案
- サンプルコードの改善
- 新しい情報の追加

## Author

**Takeshi Kano**
- 🌐 https://kano.codes/
- 🐦 https://x.com/tonkotsuboy_com
- 📝 https://zenn.dev/tonkotsuboy_com
