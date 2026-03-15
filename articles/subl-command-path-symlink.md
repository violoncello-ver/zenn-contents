---
title: "Sublime Text |「subl: command not found」を解決しながらPATHの仕組みを理解する【Mac】"
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

この記事では **エラーを解決するだけでなく、なぜそうなるのかの仕組みも一緒に理解できる** ように書いています。
仕組みがわかると、今後同じようなエラーに自分で対処できるようになります。

**所要時間：約10分**

---

## 前提条件

- MacBook（Apple Silicon / M1以降）
- Sublime Text がインストール済み（`/Applications/Sublime Text.app` が存在する）

---

## なぜ `command not found` になるのか

ターミナルに `subl` と打つと、シェル（zsh）が「`subl` という命令はどこにある？」と探しに行きます。
このとき探しに行く場所のリストを **PATH** と呼びます。

```
PATH（探す場所のリスト）
─────────────────────────
1番目 /opt/homebrew/bin
2番目 /usr/local/bin
3番目 /usr/bin
4番目 /bin
...
```

Sublime Text の実行ファイルはここにあります。

```
/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl
```

この場所は PATH に含まれていないため、シェルが見つけられず `command not found` になります。

```
シェルが探す場所           subl の実際の場所
─────────────────       ─────────────────────────────────────────
/opt/homebrew/bin       /Applications/Sublime Text.app/   ← ここは探さない！
/usr/local/bin          └── Contents/SharedSupport/bin/
/usr/bin                        └── subl  ← 見つからない → command not found
/bin
```

---

## 解決の流れ

```
① 自分専用のコマンド置き場（~/bin）を作る
② subl のシンボリックリンクを ~/bin に置く
③ ~/bin を PATH に追加して反映する
```

---

## Step 1: コマンドを置くフォルダを作る

```zsh
mkdir ~/bin
```

`~/bin` は `/Users/あなたのユーザー名/bin` に相当します。

:::message
**なぜ既存の `/usr/bin` などを使わないのか？**
OS や Homebrew が管理する場所に手を入れると、誤操作で OS が壊れるリスクがあります。
`~/bin` は自分専用なので、ミスしても影響が自分だけに収まります。
:::

---

## Step 2: subl のシンボリックリンクを作る

シンボリックリンクとは、Windows の「ショートカット」・Mac の「エイリアス」に相当するものです。
本体を移動せずに、`~/bin` から呼び出せるようにします。

```zsh
ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" ~/bin/subl
```

```
~/bin/subl（ショートカット）
    └──→ /Applications/Sublime Text.app/.../subl（本体）
```

---

## Step 3: PATH に `~/bin` を追加して反映する

```zsh
echo 'export PATH=$PATH:$HOME/bin' >> ~/.zshrc
source ~/.zshrc
```

- `~/.zshrc` はターミナル起動時に自動で読み込まれる設定ファイルです
- `$PATH:$HOME/bin` は「既存の PATH の末尾に `~/bin` を追加」する書き方です
- `source ~/.zshrc` を忘れると、新しいターミナルを開くまで反映されません

:::message alert
`>>` と `>` を間違えないでください。
- `>>` → ファイルの末尾に**追記**（安全）
- `>` → ファイルを**上書き**（既存の設定が消える）
:::

---

## 動作確認

```zsh
subl .      # Sublime Text が開けば成功🎉
which subl  # /Users/あなたのユーザー名/bin/subl と表示されれば正常
```

---

## うまくいかない場合

**① `source ~/.zshrc` を実行したか？**
最も多いのはこの忘れです。

**② シンボリックリンクが正しく作れているか？**

```zsh
ls -la ~/bin/subl
```

以下のように表示されれば正常です。

```
lrwxr-xr-x  ~/bin/subl -> /Applications/Sublime Text.app/.../subl ✅
```

`No such file or directory` と出たら Step 2 を再実行してください。

**③ PATH に `~/bin` が入っているか？**

```zsh
echo $PATH | tr ':' '\n' | grep 'あなたのユーザー名/bin'
```

`/Users/あなたのユーザー名/bin` が表示されれば正常です。表示されない場合は Step 3 を再実行してください。

---

:::message
**他の方法について**
`/usr/local/bin` に直接シンボリックリンクを貼る方法もよく紹介されています。
どちらでも動作しますが、個人の Mac では `~/bin` の方が OS への影響がなく安全です。
[公式ドキュメント](https://www.sublimetext.com/docs/command_line.html) も参考にしてください。
:::

:::message
**Homebrew を使う場合**
`brew install` が便利な理由は、今回やった一連の手順（シンボリックリンクの作成・PATH への追加）を自動でやってくれるからです。
仕組みを理解しておくと、`brew` が使えない場面でも自分で対処できます。
:::
