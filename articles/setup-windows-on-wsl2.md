---
title: "Windows + WSL2でUbuntuの開発環境構築"
emoji: "️🐧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Windows, WSL2, Linux]
published: true
---

## はじめに

WSL2を用いてWindows上でLinux環境を使えるようにしました

また、追加で

- NeoVim
- Docker

も設定しました

## WSL2

### 概要

- Windows Subsystem for Linux2
  - Windows上でLinuxが使える
- ハイパーバイザ型の仮想環境
  - ホストOSそのものを仮想化する仕組み
  - ホスト型(例えば、VirtualBox)より性能劣化が低い


### 環境構築

- ここら辺の記事を読みUbuntuをセットアップ
  - 何も設定せずとも`wsl2`コマンドがPowershellで使えたので、Ubuntuのインストールはかなり簡単だった

https://qiita.com/amenoyoya/items/ca9210593395dbfc8531

---

## NeoVimの環境構築

- Ubuntuを入れた時点でVimは使えるようになっている
- NeoVimも使えるようにする
  - 公式Docを読みつつ、Linux版のパッケージを入れる
    - https://github.com/neovim/neovim/wiki/Installing-Neovim
- NeoVimの環境設定としては、現状coc.nvimとフォントのみOSでの設定が必要

https://github.com/tmp-friends/nvim-settings

### coc.nvim

LSPで用いるcoc.nvimのためにnodejsをいれる

```sh
sudo apt install nodejs npm
sudo apt install yarn
```

#### nodejsのバージョンが古いとのエラーが出る

確認してみたところ、現在(2022/07)での最新版はv18

```sh
$ node -v
v10.19.0
```

パッケージを使用して最新版のnodejsをいれる

```sh
sudo npm install n -g
```

```sh
sudo n latest
```

確認

```sh
$ node -v
v18.5.0
```

aptでインストールしたnodejs, npmは不要なため削除

```sh
sudo apt purge nodejs npm
```

#### CocInstall coc-tsserver時にRequestがTimeoutする

- issueがあがっていた
  - https://github.com/neoclide/coc-tsserver/issues/25

- 他のcoc拡張は正常にインストールできるのでts-serverのファイルサイズが大きすぎるのが原因っぽい

- wikiの[coc拡張にはvimのプラグインマネージャを使用する](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions#use-vims-plugin-manager-for-coc-extension)を参考にすることで解決
  - init.vimに`Plug 'neoclide/coc-tsserver', {'do': 'yarn install --frozen-lockfile'}
`を追記

### フォントのインストール

フォルダやファイルの左横にアイコンを表示させるための設定

- NerdfontをWindows上でDL
  - 今回はNerdfontのDoidSansMonoのみ欲しかったので、[ここのページ](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/DroidSansMono/complete)から直接DLした
- Windowsの設定から 個人用設定>フォント でDLしたフォントを追加
- ターミナルでフォントを設定する

![](/images/setup-wsl2/dev-icon.png)

https://qiita.com/park-jh/items/4358d2d33a78ec0a2b5c

## 参考

https://wonwon-eater.com/coc-javascript-error/

https://qiita.com/seibe/items/36cef7df85fe2cefa3ea

---

## Dockerの環境構築

- Docker Desktop for Windowsがインストール済みのため、以下の記事を参考にしてUbuntuでも使用できるようにする
  - Settings>Resources>WSL INTEGRATION からUbuntuをオンにする

https://www.aise.ics.saitama-u.ac.jp/~gotoh/DockerOnWindowsWithWSL2.html

- WSL2 + Dockerのネットワークについては以下の記事が分かりやすかった

https://qiita.com/amenoyoya/items/8992fc1659e07b4bb7d6

## おわりに

- WSL2を使えば簡単にWindowsでLinux環境が作れました
- 環境がLinuxだと開発する上でなにかと楽だし、Linuxの勉強にもなるのでいいかもしれません


