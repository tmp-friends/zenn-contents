---
title: "VSCode RemoteContainerを導入する"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker", "VS Code"]
published: true
---

## 前提

- Docker環境を構築できていること
- VSCodeをインストールしていること

私が試した環境は以下になります

https://zenn.dev/temple_c_tech/articles/setup-next-on-docker

## RemoteDevelopment拡張機能のインストール

VSCodeのマーケットプレイスから

- Remote Development

をインストール

Remote Developmentには
- Remote - WSL
- Remote - Containers
- Remote - SSH

が含まれていて便利です

## コンテナ内に入る

1. 左下の`><`マークをクリック
2. `Reopen in Container`を選択

## RemoteContainer内での拡張機能

- グローバルにインストールしていない拡張機能はdevcontainer.jsonの`extensions`に記述することで使用可能になります
- 拡張機能の歯車マークから`Add to devcontainer.json`をクリックすると楽に追加できます

![](/images/setup-remote-container/plugin-on-remotecontainer.png)

```json:.devcontainer/devcontainer.json
// Add the IDs of extensions you want installed when the container is created.
"extensions": [
  "dbaeumer.vscode-eslint",
  "esbenp.prettier-vscode",
  "syler.sass-indented",
  "PKief.material-icon-theme",
  "AbhijoyBasak.nestjs-files",
  "eg2.vscode-npm-script",
  "christian-kohler.path-intellisense",
  "timitejumola.chakra-ui-doc",
  "ms-azuretools.vscode-docker"
]
```
## おわりに

- 何度か耳にしたVSCodeのRemoteContainer機能を試してみました
  （こんなに簡単にできると思ってなかったです）
- dockerで環境構築する際に、RemoteContainre機能を使うと、Docker環境をローカル環境のように使用できて良いなと思いました
  - `docker-compose run --rm <アプリ名>`をコマンドにいちいちつけなくてもよい
    など

## 参考

- https://penpen-dev.com/blog/vscode-remote-container-toha/
- 拡張機能
  - https://www.sun-m.co.jp/blog/tips/414.html
