---
title: "「subl: command not found」を解決しながらPATHの仕組みを理解する【Mac入門】"
emoji: "🔍"
type: "tech"
topics: ["mac", "shell", "zsh", "sublimetext", "初心者"]
published: true
---

## こんなエラーに遭遇しませんでしたか？

Sublime Text をインストールしてターミナルから開こうとしたら…

```zsh
$ subl .
zsh: command not found: subl
```

「インストールしたのになんで使えないの？」と感じた方、この記事はそんなあなたに向けて書きました。

この記事では **エラーを解決するだけでなく、なぜそうなるのかの仕組みも一緒に理解できる**ように書いています。仕組みがわかると、今後同じようなエラーに自分で対処できるようになります。

---

## この記事でやること

1. なぜ `command not found` になるのかを理解する
2. `subl` コマンドを使えるようにする（3つの手順）
3. 正しく設定できたか確認する

**所要時間：約10分**

---

## 前提条件

以下がすでに済んでいることを確認してください。

- [ ] MacBook（Apple Silicon / M1以降）を使っている
- [ ] Sublime Text がインストール済み（`/Applications/Sublime Text.app` が存在する）
- [ ] Homebrew がインストール済み（`brew --version` でバージョンが表示される）

---

## なぜ `command not found` になるのか

まずここを理解することが一番大事です。

ターミナルに `subl` と打つと、**zsh（シェル）** という翻訳係が「`subl` という命令はどこにある？」と探しに行きます。

:::message
**シェルとは**
人間が打ったコマンドをコンピュータに伝える翻訳係のことです。Mac のデフォルトは **zsh（ズィーシェル）** が使われています。
:::

このとき、シェルが探しに行く場所のリストを **PATH（パス）** と呼びます。

```
PATH（探す場所のリスト）
─────────────────────────
1番目 /opt/homebrew/bin
2番目 /usr/local/bin
3番目 /usr/bin
4番目 /bin
...
```

シェルはこのリストを**上から順番**に `subl` を探します。リストにない場所は**絶対に探しません。**

Sublime Text の実行ファイルはここにあります。

```
/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl
```

この場所は PATH のリストに含まれていないため、シェルが見つけられず `command not found` になります。これがエラーの原因です。

```
シェルが探す場所        subl の実際の場所
─────────────────       ─────────────────────────────────────────
/opt/homebrew/bin       /Applications/Sublime Text.app/   ← ここは探さない！
/usr/local/bin          └── Contents/
/usr/bin                    └── SharedSupport/
/bin                            └── bin/
...                                 └── subl  ← 見つからない → command not found
```

---

## 解決方法の全体像

この問題を解決するには、以下の3ステップが必要です。

```
① 自分専用のコマンド置き場（~/bin）を作る
      ↓
② subl の「ショートカット」を ~/bin の中に作る
      ↓
③ ~/bin を PATH（探す場所リスト）に追加する
```

順番に進めましょう。

---

## Step 1: コマンドを置くフォルダを作る

```zsh
mkdir ~/bin
```

`mkdir` は新しいフォルダを作るコマンドです（"make directory" の略）。

`~`（チルダ）は**ホームディレクトリ**（自分専用フォルダの最上位）を表す記号で、`/Users/あなたのユーザー名` と同じ意味です。

つまり `~/bin` は `/Users/kame/bin` という場所にフォルダを作ります。

:::message
**なぜ既存の `/usr/bin` などを使わないのか？**
PATH にはすでにいくつかの `bin` フォルダが存在しますが、これらは OS や Homebrew が管理する場所です。誤って大事なファイルを上書きすると OS が壊れる可能性があります。`~/bin` は**自分専用の安全な場所**なので、何かミスをしても影響範囲が自分だけに収まります。
:::

---

## Step 2: subl のショートカットを作る

Sublime Text の実行ファイルは深い場所にあります。毎回フルパスを入力するのは現実的ではないので、`~/bin` の中に**ショートカット**を作ります。

このショートカットのことを **シンボリックリンク** と言います。Windows の「ショートカット」・Mac の「エイリアス」と同じ概念です。

```zsh
ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" ~/bin/subl
```

`ln -s` がシンボリックリンクを作るコマンドです（"link symbolic" の略）。

コマンドの構造はシンプルです。

```
ln -s  "本体のありか"  "ショートカットを置く場所"
```

実行後のイメージ：

```
~/bin/subl（ショートカット）
    │
    └──→ /Applications/Sublime Text.app/.../subl（本体）
```

本体は動かさず、呼び出しやすい `~/bin` からアクセスできるようになりました。

---

## Step 3: PATH に ~/bin を追加する

ショートカットを作っただけでは、まだシェルは `~/bin` を探しに行きません。PATH（探す場所リスト）に `~/bin` を追加する必要があります。

```zsh
echo 'export PATH=$PATH:$HOME/bin' >> ~/.zshrc
```

少し複雑に見えますが、分解するとシンプルです。

**`~/.zshrc` とは**
ターミナルを起動するたびに**自動で読み込まれる設定ファイル**です。ここに書いた内容は毎回自動で適用されます。

**`echo '...' >>` とは**
`echo` は文字列を出力するコマンドで、`>>` と組み合わせることでファイルの末尾に追記できます。

:::message alert
`>>` と `>` を間違えないでください。
- `>>` → ファイルの末尾に**追記**（安全）
- `>` → ファイルを**上書き**（既存の内容が全部消える）
:::

**`export PATH=$PATH:$HOME/bin` とは**

```
今の PATH の内容    +    ~/bin を追加
     ↓                       ↓
export PATH = $PATH    :    $HOME/bin
                        ↑
                   コロンが区切り文字
```

この1行を `.zshrc` に追記することで、**ターミナルを起動するたびに `~/bin` が PATH に追加される**ようになります。

---

## Step 4: 設定を今すぐ反映する

`.zshrc` を編集しただけでは、**今開いているターミナルにはまだ反映されていません。** 以下のコマンドで即時反映させます。

```zsh
source ~/.zshrc
```

`source` は設定ファイルを今のターミナルに即時反映させるコマンドです。これを実行しないと新しいターミナルを開くまで `subl` が使えないままになります。

---

## 動作確認

2つのコマンドで確認します。

### 確認1: subl コマンドが動くか

```zsh
subl .
```

Sublime Text が開けば**成功**です🎉

`.` は「今いるフォルダ」という意味で、`subl .` で現在のフォルダを Sublime Text で開きます。

### 確認2: シェルが subl をどこで見つけているか

```zsh
which subl
```

以下のように表示されれば正しく設定できています。

```
/Users/kame/bin/subl  ✅
```

`which` は「このコマンドはどこにある？」をシェルに問い合わせるコマンドです。PATH を上から探して最初に見つかった場所を返します。

---

## うまくいかない場合のチェックリスト

`command not found` が続く場合は以下を順番に確認してください。

**① `source ~/.zshrc` を実行したか？**
これを忘れているケースが最も多いです。必ず実行してください。

**② シンボリックリンクが正しく作れているか？**

```zsh
ls -la ~/bin/subl
```

以下のように `->` で本体を指していれば正常です。

```
lrwxr-xr-x  ~/bin/subl -> /Applications/Sublime Text.app/.../subl ✅
```

`No such file or directory` と出た場合は Step 2 のコマンドを再実行してください。

**③ PATH に `~/bin` が入っているか？**

```zsh
echo $PATH | tr ':' '\n' | grep 'kame/bin'
```

`/Users/kame/bin` が表示されれば正常です。表示されない場合は Step 3〜4 を再実行してください。

---

## まとめ

この記事でやったことを整理します。

```
問題
└── subl の実行ファイルが PATH の外にある → command not found

解決策
├── Step 1: mkdir ~/bin
│          自分専用のコマンド置き場を作る
│
├── Step 2: ln -s ".../subl" ~/bin/subl
│          実行ファイルへのショートカットを置く
│
├── Step 3: echo 'export PATH=...' >> ~/.zshrc
│          ~/bin を PATH（探す場所リスト）に追加
│
└── Step 4: source ~/.zshrc
           設定を今すぐ反映
```

`brew install` が便利な理由は、この4ステップを**全部自動でやってくれるから**です。仕組みを理解しておくと、`brew` が使えない場面でも自分で対処できるようになります。
