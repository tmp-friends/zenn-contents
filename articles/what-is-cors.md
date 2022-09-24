---
title: "CORSとは何なのか"
emoji: "🥴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['http']
published: false
---

## CORS

- CORS(Cross-Origin Resource Sharing) = オリジン間リソース共有
  - 追加のHTTPヘッダーを使用して、あるオリジンで動作しているアプリケーションに、
異なるオリジンにあるリソースへのアクセス権を与えるようにブラウザに指示を出すための仕組み
  - 異なるオリジンに対して、アクセスを制限することで以下のような脆弱性を防げる
    - XSS
    - CSRF

- オリジン =  プロトコル + ドメイン + ポート番号
  - https://foo.com:443 = https:// + foo.com + 443

- オリジン間HTTPリクエスト
  - 異なるオリジンにあるリソースへリクエストするときに実行する

### 例

`https://foo.com`で提供されているフロントエンドから、`XMLHttpRequest`を使用して、`https://bar.com`で提供されているバックエンドAPIへリクエストを行うような場合

- ブラウザはスクリプトによって開始されるオリジン間HTTPリクエストを制限している
  - `XMLHttpRequest`や`FetchAPI`は同じオリジンのリソースのみリクエストを行うことができる(=同一オリジンポリシー)
  - それ以外のオリジンからの場合は正しいCORSヘッダーを含んでいる必要がある

### HTTPヘッダーでのやりとり

- リクエストヘッダー
  - `Origin`が含まれており、ここで判定をする

```sh
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

- レスポンスヘッダー
  - `Access-Control-Allow-Origin`でアクセスできるドメインを制御
  - 以下のレスポンスヘッダーでは、すべてのドメインからアクセスできることを示している

```sh
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML データ…]
```

## 参考

https://developer.mozilla.org/ja/docs/Web/HTTP/CORS

https://qiita.com/att55/items/2154a8aad8bf1409db2b
