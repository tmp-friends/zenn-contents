---
title: "Linux環境でDockerコンテナ内にuserで入る"
emoji: "🐧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Linux, Docker]
published: true
---

## 問題点

- Linux環境でDockerコンテナを起動して、`exec`コマンドでコンテナ内に入るとroot権限になってしまう

- コンテナに入った状態で、ファイルを以下のように作成すると当然ながら所有権がrootになってしまう
```sh
nest g controller <ファイル名>
```

- （user権限で使用している）Vimなどでのエディタで編集しようとしてもファイルがreadonlyとなってしまうため、編集できなくなるという問題に直面

## 解決策

Dockerコンテナにuser権限で入るようにすることで解決
`-u`オプションでユーザを指定できる

```sh
docker exec -u $(id -u $USER):$(id -g $USER) -it <コンテナ名> /bin/sh
```

指定するユーザについてはLinux上で`id`コマンドを実行することで確認できる


```sh
uid=1000(<ユーザ名>) gid=1000(<ユーザ名>) groups=1000(<ユーザ名>) ...
```

dockerコマンドではuidとgidを指定している

## 参考

https://qiita.com/Nahuel/items/d11d169ba7c40b413e6b

https://uxmilk.jp/8530
