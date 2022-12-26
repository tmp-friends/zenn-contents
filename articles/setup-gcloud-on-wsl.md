---
title: "WSL2でgcloud CLIをインストールする"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GCP", "WSL2"]
published: true
---

## 事象

- WSL2でgcloud CLIのセットアップをしようした
- `gcloud init`からWebブラウザ上で認証を行おうとした際、`アクセスをブロック：認証エラーです`の画面が表示されてしまった

## 結論

- 公式Docにブラウザのないマシンでgcloud CLIを承認する手順が書いてあった

https://cloud.google.com/sdk/docs/authorizing?hl=ja#run_gcloud_auth_login

1. WSL2上で`gcp init --no-browser`

2. `gcloud auth login --remote-bootstrap="https://accounts.google.com/o/oauth2/auth?...&token_usage=remote"`といったコマンドが表示されるので、Windowsのコマンドプロンプト上で貼り付けて実行

3. localhostが立ち上がり以下のようなURLが生成されるので、それをWSL2の`Enter the output of the above command:`の後に貼り付ける
`https://localhost:8085/?state=&...prompt=consent`

