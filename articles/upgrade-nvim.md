---
title: "UbuntuでNvimを最新にする"
emoji: "🐧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim"]
published: true
---

## 経緯

- WSL2のUbuntuで開発している
- `sudo apt install neovim`で最新バージョンが入らなかった
  - 一部のPluginでNvimのバージョンが古いためエラーが出ていた
- 最新版にしよう

## 手順

- `sudo apt install neovim`で入れたNvimを削除する
  - `sudo apt remove neovim`

- AppImageの手法でNvimをインストールする
→最新のNvimがインストールできる
  - https://github.com/neovim/neovim/wiki/Installing-Neovim#appimage-universal-linux-package

## おわりに

- `sudo apt install neovim`では古いバージョンがインストールされてしまうので気を付ける
  - Nvimのversionを指定してインストールするやり方は探せばありそうだが、AppImageの手法で入れちゃった方が早そうだったのでそうした
