# 【Mac】Zennに記事を投稿するまでの完全手順
所要時間は**約30〜60分**です。
## 【情報】
* [一次情報：Zenn CLI](https://zenn.dev/zenn/articles/zenn-cli-guide)
* [その他](https://zenn.dev/sosa/articles/how-zenn-github)
***

## 必要なもの

- GitHubアカウント（なければ [github.com](https://github.com) で作成）
- macOS + ターミナル
- VS Code（任意だが推奨）

***

## STEP 1｜Node.jsを入れる

```bash
# nvmをインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# シェルを再読み込み
exec zsh

# Node.jsのLTS版をインストール
nvm install --lts

# バージョン確認（数字が出ればOK）
node -v
npm -v
```



***

## STEP 2｜GitHubリポジトリを作る

**ブラウザ操作：**

1. [github.com](https://github.com) → **「New repository」**
2. Repository name：`zenn-contents`
3. Public / Private どちらでもOK
4. **「Add a README file」はチェックしない**（空のまま）
5. **「Create repository」** をクリック

**ターミナル操作：**

```bash
mkdir zenn-contents
cd zenn-contents
git init
git remote add origin https://github.com/あなたのユーザー名/zenn-contents.git
```



***

## STEP 3｜ZennとGitHubを連携する

1. [zenn.dev](https://zenn.dev) にアクセスし、GitHubアカウントでログイン
2. 右上のアイコン → **「GitHubからデプロイ」**
3. **「リポジトリを連携する」→「連携へ進む」**
4. GitHubの認証画面で **「Only select repositories」** を選択
5. STEP 2で作った `zenn-contents` を指定
6. **「Install & Authorize」** → Zennに戻れば連携完了 ✅
***

## STEP 4｜Zenn CLIをセットアップ

⚠️ **`zenn-contents/` ディレクトリの中で実行すること**

```bash
# zenn-contents/ 内にいるか確認
pwd  # → /Users/ユーザー名/zenn-contents と表示されればOK

npm init --yes
npm install zenn-cli
npx zenn init
```

実行後のフォルダ構成： [zenn](https://zenn.dev/long910/articles/2025-05-25-zenn-github-integration)

```
zenn-contents/
├── articles/    ← 記事のMarkdownを置く場所
├── books/       ← 本を書くときに使う
└── .gitignore
```

初回コミット：

```bash
git add .
git commit -m "initial commit"
git push -u origin main
```

***

## STEP 5｜記事を書いて公開する

### 記事ファイルを作成

```bash
# 場所の確認
pwd  # → /Users/ユーザー名/zenn-contents と表示されればOK

# 記事ファイル作成
npx zenn new:article
```

### frontmatterを編集

`articles/` 以下に生成されたMarkdownファイルを開いて、冒頭を編集します：

```markdown
---
title: "記事のタイトル"
emoji: "🚀"
type: "tech"           # tech: 技術記事 / idea: アイデア
topics: ["go", "aws"]  # タグは最大5つ
published: false       # まずfalseで執筆する
---

## はじめに
ここから本文を書く！
```

### ローカルでプレビュー確認

```bash
npx zenn preview
# → ブラウザで http://localhost:8000 を開く
```

Zennと同じ見た目でリアルタイム確認できます 。 [zenn](https://zenn.dev/dfuji/articles/zenn-cli-point)

### 公開する

`published: false` → **`published: true`** に変更してからpush：

```bash
git add .
git commit -m "add: 記事タイトル"
git push origin main
```

**pushするだけでZennに自動公開されます** 。 [zenn](https://zenn.dev/sosa/articles/how-zenn-github)

***

## ⚠️ つまずきポイント3つ

- `npx zenn init` は必ず `zenn-contents/` の中で実行する [zenn](https://zenn.dev/dfuji/articles/zenn-cli-point)
- pushするブランチは **main**（masterだと反映されない） [zenn](https://zenn.dev/sosa/articles/how-zenn-github)
- `published: false` のままpushしても下書き保存のみで公開されない [zenn](https://zenn.dev/long910/articles/2025-05-25-zenn-github-integration)
