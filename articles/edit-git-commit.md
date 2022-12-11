---
title: "[Git] コミットの単位を編集したい時に使っているコマンド"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['git']
published: true
---

## はじめに

PRを出す前など、コミットの単位をいい感じにまとめ直したくなる時がよくあると思います

そんな時に私がよく使っているコマンドを紹介します

## コマンド

- `git rebase -i origin <branch名>`
- `git reset HEAD`

### git rebase -i origin <branch名>

オプションの説明
- `i`オプション: interactive(=対話)モードを使う
- `origin <branch名>`: 編集対象が、現在のブランチで作ったコミットのみに限定できる
  - もちろん`HEAD~`のように指定しても大丈夫です

実行すると以下のようなinteractiveモードが開くと思います

```sh:interactiveモード
pick 05191fc first commit
pick 05120dd second commit
pick 05dc5b2 third commit

# Rebase 82f0447..05dc5b2 onto 05191fc
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

ここで以下の3つをよく使うことが多いです

- `r(reward)`: コミット名を変える
- `f(fixup)`: コミットをひとつ前のものにまとめる
- コミットの順番を変える(conflictしないようであれば)

#### ex. `third commit`を`first commit`に含めたいとき

以下のようにするだけでできます

- `third commit`を`first commit`の次になるようcommitの順番を変える
- `third commit`の`pick`を`f`にする

```sh:interactiveモード
pick 05191fc first commit
f 05dc5b2 third commit
pick 05120dd second commit

# Rebase 82f0447..05dc5b2 onto 05191fc
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

補足)`rebase -i`の基本的な使い方はこちらの記事などをご覧ください

https://www.karakaram.com/git-rebase-i-usage/

### git reset HEAD

- 実装した変更はそのままにcommitをなくすことができます
  - commitの単位をまとめ直したいときに使うことが多いです

- HEADの後ろに`~`をn個つけることで対象にしたいcommitをn個指定できます

補足)`reset`の基本的な使い方はこちらの記事などをご覧ください

https://www-creators.com/archives/1069

---

## おまけ

過去のコミットに手をつけなくていい時

- `git add -p`
- `git commit --amend`

### git add -p

通常の`git add`ではファイル単位でしか取り扱えません
しかし、`p`オプションをつけることで、より細かいコードのまとまり単位で取り扱うことができます

### git commit --amend

`git add`でstageに乗せたものを現在のcommitに含めることができます

## おわりに

commitの単位が汚いと
- PRを見る側に負担を強いてしまう
- 他の開発者や未来の自分にとってもcommitで行っている変更がわかりにくい

ということになるので、できる限りわかりやすくした方がいいですね

