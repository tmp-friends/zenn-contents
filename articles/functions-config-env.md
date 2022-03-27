---
title: "FirebaseCloudFunctionsで環境変数を設定する"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Firebase", "Cloud Functions"]
published: false
---

FirebaseCloudFunctionsで環境変数を設定するにあたり、以下の2通りの手段が提供されています。
* .envファイルで設定する
* CLIで設定する

## .envファイルで設定する

functions/ 配下に`.env`ファイルを作成し、キーを追加するだけでできます。

https://firebase.google.com/docs/functions/config-env?hl=ja#env-variables

### 本番環境での環境変数の確認方法

デプロイするとGCPのCloudFunctionsの変数タブから確認できます

![](/images/functions-config-env/production_env.jpg)

## CLIで設定する

こちらの方法は実際に使用していないので自分用のメモです。

set

```sh
$ firebase functions:config:set twitter.app_key="YOUR_APP_KEY"
```

get

```sh
$ firebase functions:config:get
{
  "twitter": {
    "app_key": "YOUR_APP_KEY"
  }
}
```

## おわりに

CloudFunctionsなら簡単に環境変数を設定できることがわかりました。
CloudFunctions以外で環境変数を設定する場合は、dotenvなどを使用するといいかなと思います。

## 参考
* https://firebase.google.com/docs/functions/config-env?hl=ja#env-variables
* https://blog.katsubemakito.net/firebase/functions-environmentvariable
