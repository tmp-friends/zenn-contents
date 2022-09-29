---
title: "NestJSでのCORSエラーを解消する"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['NestJS', 'TypeScript', 'Web', 'http', 'Security']
published: false
---

開発時にNext.jsからNestJSへデータfetchするとCORSエラーが起きました

- cosoleでのエラー文

```
Access to fetch at '<Next.jsアプリのlocalhost>' from origin '<NestJSアプリのlocalhost>' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

解消まで時間がかかったので備忘録として残します


## CORSとは

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

## NestJSでCORSを許可する

ヘッダーには確かに`Access-Control-Allow-Origin`や`Access-Control-Allow-Headers`情報が付与されていない

![error-header](/images/resolve-nestjs-cors-error/error-header.jpg)

### Middleware

NestJSのMiddlewareを用いてResponseオブジェクトのヘッダーに情報を付与する

MiddlewareとはClientからのRequestがNestJSのRouteHandlerに到達する前に、RequestオブジェクトやResponseオブジェクトに介入できる仕組み

![nestjs-middleware](/images/resolve-nestjs-cors-error/nestjs-middleware.png)

```ts:cors.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class CorsMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.header('Access-Control-Allow-Origin', '*');
    res.header(
      'Access-Control-Allow-Headers',
      'Origin, X-Requested-With, Content-Type, Accept',
    );
    next();
  }
}
```

作成した`cors.middleware.ts`を`app.module.ts`に適用させる

```ts:app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CorsMiddleware } from './cors.middleware';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(CorsMiddleware).forRoutes('*');
  }
}
```

ヘッダーに`Access-Control-Allow-Origin`や`Access-Control-Allow-Headers`情報が付与されるようになった

![success-header](/images/resolve-nestjs-cors-error/success-header.jpg)

### その他の解決策

- app.enableCors()
  - NestJSアプリのbootstrap時に`enableCors`関数を適用させる方法
    - 調べたらよく出てくる解決策
  - 何故か上手くいかなかった
    - Next.jsでのリクエストの仕方の問題？
      - SWRを使ってfetchしている

## 参考

- CORSの仕組み

https://developer.mozilla.org/ja/docs/Web/HTTP/CORS

https://qiita.com/att55/items/2154a8aad8bf1409db2b

- NestJSでのCORS許可

https://qiita.com/daikiojm/items/4c2143a213332abf1170

- NestJSのMiddleware

https://docs.nestjs.com/middleware
