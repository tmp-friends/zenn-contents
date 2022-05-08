---
title: "TypeScript×FirebaseでTwitterBotを制作した"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "Firebase", "Twitter"]
published: true
---

## 制作物

毎朝7:30に東京の天気予報をつぶやくTwitterBotを制作しました。

TypeScriptをFirebase CloudFunctions上で定期実行させることで実現しています。

https://twitter.com/OzoraOtenkiBot

https://twitter.com/OzoraOtenkiBot/status/1507484991546142721

GitHubでコードを公開しています！

https://github.com/tmp-friends/ozora-otenki-bot-ts

### 背景

『アイカツ！』というアニメで、大空あかりちゃんがお天気キャスターとして活躍します。それをBotとして作りました。
朝に東京の天気もわかるし、推しの画像も見られるので一石二鳥です。

実はこのBotは3年前からRubyで作成しHeroku上で運用していたのですが、今回作り直しました。
理由としては、以下です。
* [HerokuでSSL証明書が必須](https://devcenter.heroku.com/ja/articles/ssl-endpoint)となったらしく、毎月課金しないとBotが動かない状態にあったから
* TypeScriptの勉強がてら作るには最適なレベル感だったから

### 仕様

* Pub/Subを用いて毎朝7:30に定期実行
* [天気予報API](https://weather.tsukumijima.net/)から、今日の東京の天気予報をGET
* TwitterAPIを用いて、画像付きツイート

## ポイント

今回制作した上での個人的なポイントです。

* TypeScriptの導入と使い方
* Firebaseでローカル実行環境を整える
* Pub/Subを用いて、定期実行する
* 天気予報APIから天気データをGETする
* TwitterAPIを用いて、ツイートする

それぞれ詳しく見ていきます。

### TypeScriptの導入と使い方

#### 導入

FirebaseでのTypeScript導入にあたり、[Firebase公式Doc](https://firebase.google.com/docs/functions/typescript)を参考にしました。

とはいえ、導入はとても簡単で、

`firebase init`でのプロジェクト初期化時に言語をTypeScriptに選択するだけでした。

#### 使い方

使い方としては、TypeScriptで書かれたコードをJavaScriptにトランスパイルして実行するとのこと。

`package.json`にコマンドが書いてあるので、その部分を読めば大丈夫でした。

ex)
* トランスパイルしたいとき
→functions配下で`npm run build`

* デプロイしたいとき
→functions配下で`npm run deploy`

**(Firebase Emulatorsでローカル実行したいときは、少し複雑なため後述)**

>  "scripts": {
    "lint": "eslint --ext .js,.ts .",
    "fix": "eslint --ext .js,.ts . --fix",
    "build": "tsc",
    "serve": "npm run build && firebase emulators:start --only functions",
    "shell": "npm run build && firebase functions:shell",
    "start": "npm run shell",
    "deploy": "firebase deploy --only functions",
    "logs": "firebase functions:log"
  },

### Firebaseでローカル実行環境を整える
* Pub/Sub
* Firebase Cloud Functions

上記を同時にFirebase Emulatorsでローカル実行する方法が少し複雑でした。

#### 準備
`Firebase init`からEmulatorをルートディレクトリにインストールします
Emulatorの質問事項で選択するのは以下です。
* Functions
* PubSub

定期的に実行する関数を用意する

```ts
import * as functions from "firebase-functions";

const runtimeOpts = {
  timeoutSeconds: 180,
  memory: "512MB" as const,
};

export const twitterBot = functions
    .runWith(runtimeOpts)
    // 2分おきに実行される
    .pubsub.schedule("*/2 * * * *")
    .onRun(async (_context) => {
      try {
        console.log("test");
      } catch (e) {
        console.error(e);
      }
    });
```

#### ローカル実行
エミュレーターを起動

```
firebase emulators:start
```

このままだと定期実行されないので、shellから手動で関数を実行します
（一旦実行したい関数をデプロイしないといけなかったかもですが、そこら辺の手順を忘れてしまいました...動かない方はデプロイしてみてください。）

```
firebase functions:shell
firebase > twitterBot()
```

#### ハマったこと

エミュレーター起動時にJavaをインストールしてとのエラーが出た
```
firebase emulators:start
```

pubsub-debug.log

```sh
ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH.

Please set the JAVA_HOME variable in your environment to match the
location of your Java installation.
```

下記の記事を参考にJavaをインストールしました

https://qiita.com/BlackMagician/items/7fd102a4ee8b3831cc44

"java.exe"で新たにターミナルが開かれ、テスト実行できるようになりました

```sh
Implementation may be incomplete or differ from the real system.
3 16, 2022 9:06:47 午前 com.google.cloud.pubsub.testing.v1.Main main
情報: IAM integration is disabled. IAM policy methods and ACL checks are not supported
3 16, 2022 9:06:48 午前 io.gapi.emulators.netty.NettyUtil applyJava7LongHostnameWorkaround
情報: Unable to apply Java 7 long hostname workaround.
3 16, 2022 9:06:48 午前 com.google.cloud.pubsub.testing.v1.Main main
情報: Server started, listening on 8085
3 16, 2022 9:06:48 午前 io.gapi.emulators.grpc.GrpcServer$3 operationComplete
情報: Adding handler(s) to newly registered Channel.
3 16, 2022 9:06:51 午前 io.gapi.emulators.grpc.GrpcServer$3 operationComplete
情報: Adding handler(s) to newly registered Channel.
3 16, 2022 9:06:51 午前 io.gapi.emulators.netty.HttpVersionRoutingHandler channelRead
情報: Detected HTTP/2 connection.
```

よくよく見ると、
GCP公式Docのローカルでのテスト要件にJavaが必要なことが書かれていました...w

https://cloud.google.com/pubsub/docs/emulator?hl=ja

### Pub/Subを用いて、定期実行する

**定期実行処理**

/functions/src/index.ts

処理の起点となる箇所です。

CloudFunctionsで定期実行する方法は以下を参考にしました。

https://firebase.google.com/docs/functions/schedule-functions?hl=ja

> 指定した時刻に実行されるように関数をスケジュール設定する場合は、functions.pubsub.schedule().onRun() を使用します。

timezoneはデフォルトでアメリカに設定されていたので、日本との時差16hを考慮し指定しました。

https://citizen.jp/support-jp/manual/terms/deeper_05c.html

```ts
import * as functions from "firebase-functions";
import {tweetWeatherInfo} from "./core";

const runtimeOpts = {
  timeoutSeconds: 180,
  memory: "512MB" as const,
};

export const twitterBot = functions
    .runWith(runtimeOpts)
    // 16h遅い(or 8h進んでいる)時間を指定
    .pubsub.schedule("30 15 * * *")
    .onRun(async (_context) => {
      try {
        await tweetWeatherInfo();
      } catch (e) {
        console.error(e);
      }
    });
```

---

余談ですが、
スケジュール実行する関数とメイン処理を行う関数は別にしてみました。
（最近、コードの美しさにこだわって試行錯誤しています...）

https://note.com/cyberz_cto/n/n26f535d6c575


**メイン処理**

/functions/src/core/index.ts

```ts
import {TwitterAPI} from "../lib/twitterAPI";
import {WeatherAPI} from "../lib/weatherAPI";

export const tweetWeatherInfo = async (): Promise<void> => {
  const weatherInfo =
    await new WeatherAPI(`${process.env.TOKYO_CODE}`).getWeatherInfo();
  console.log(weatherInfo);

  await new TwitterAPI().tweet(weatherInfo);
};
```

### 天気予報APIから天気データをGETする

/functions/src/lib/weatherAPI.ts

Axiosを使用し、天気予報APIとやりとりを行います。

https://github.com/axios/axios

下記のAxiosで使用されるプロパティの型定義を行いAPI通信を行いました。
* AxiosRequestConfig
* AxiosResponse
* AxiosError

```ts
import axios, {AxiosError, AxiosRequestConfig, AxiosResponse} from "axios";
import {Weather} from "../model/types/weather";

/**
 * 天気予報APIとやりとりするクラス
 */
export class WeatherAPI {
  private static axiosRequestConfig: AxiosRequestConfig;
  readonly BASE_URL = "https://weather.tsukumijima.net/api/forecast/city";

  /**
   * 天気データを取得したい都市を設定
   * 都市コード一覧: https://weather.tsukumijima.net/primary_area.xml
   * @param {string} cityCode 取得したい都市のコード
   */
  constructor(cityCode: string) {
    WeatherAPI.axiosRequestConfig = {
      url: `${this.BASE_URL}/${cityCode}`,
      method: "GET",
    };
  }

  /**
   * 天気予報APIから天気データを取得する
   * 天気予報API: https://weather.tsukumijima.net/
   * @param {string} cityCode 取得したい都市のコード
   * @return {Weather} {今日の天気, 今日の最高気温}
   */
  public getWeatherInfo = async (): Promise<Weather> => {
    let todayTelop = "";
    let todayMaxTemperature = "";

    await axios(WeatherAPI.axiosRequestConfig)
        .then((res: AxiosResponse) => {
          todayTelop =
            WeatherAPI.extractTodayTelop(res);
          todayMaxTemperature =
            WeatherAPI.extractTodayMaxTemperature(res);
        }).catch((e: AxiosError) => {
          console.log(e);
        });

    return {
      todayTelop,
      todayMaxTemperature,
    };
  };
  // ...
```

### TwitterAPIを用いて、ツイートする

/functions/src/lib/twitterAPI.ts

TwitterAPIをたたく際にはTwitter-API-v2というライブラリを使いました。

TwitterAPIのv1,v2共に対応しており、かなり使いやすかったです。

https://www.npmjs.com/package/twitter-api-v2

**twitterClientの作成**


```ts
import TwitterApiV2 from "twitter-api-v2";
import {Weather} from "../model/types/weather";

/**
 * Twitter-API-v2ライブラリを用いて、TwitterAPIとやりとりするクラス
 */
export class TwitterAPI {
  private static twitterClient: TwitterApiV2;
  // __dirname = ozora-otenki-bot-ts/functions/lib/lib
  private static imagesFolderPath = "./src/assets/images";

  /**
   * twitterClientを生成する
   */
  constructor() {
    TwitterAPI.twitterClient = new TwitterApiV2({
      // /functions/.env
      // HACK:undefined型を回避するため変数展開
      appKey: `${process.env.APP_KEY}`,
      appSecret: `${process.env.APP_SECRET}`,
      accessToken: `${process.env.ACCESS_TOKEN}`,
      accessSecret: `${process.env.ACCESS_SECRET}`,
    });
  }
```

---

**ツイート**

```ts
  /**
   * ツイートを実行する
   * @param {Weather} weatherInfo
   * @return {void}
   */
  public tweet = async (
      weatherInfo: Weather
  ): Promise<void> => {
    // メディアツイートはv1を推奨とのこと
    // WARN: 同じ内容をツイートするとAPIの仕様によりエラー
    const text = TwitterAPI.createTweetText(weatherInfo);
    const mediaId = await TwitterAPI.generateMediaId();

    await TwitterAPI.twitterClient.v1.tweet(text, {media_ids: mediaId});
  };
```

---

**画像付きツイート**

下記の手順で画像付きツイートができました。

1. ライブラリの`uploadMedia()`メソッドからmediaIdを生成
2. ライブラリの`tweet()`メソッドの引数にmediaIdをつける

```ts
  /**
   * 画像をUploadしたときに生成したmediaIdを返す
   * @return {string} mediaId
   */
  private static generateMediaId = async (): Promise<string> => {
    const randomId =
      TwitterAPI.generateRandomNumber(`${process.env.NUMBER_OF_IMAGES}`)
          .toString();
    const imagesPath = `${TwitterAPI.imagesFolderPath}/otenki${randomId}.jpg`;

    return TwitterAPI.twitterClient.v1.uploadMedia(imagesPath);
  };
```

## おわりに

はじめてTypeScriptを使ってみたのですが、型定義の部分がとても楽しくもあり、変なハマり方もしました...w

とりあえずTypeScriptをさわってみようというモチベーションで始めたので、言語自体の理解が全然できていないなと感じています。
次にやることとしては、動画教材もしくは書籍で体系的にTypeScriptを学ぼうと思います。

これからNuxt3が正式リリースされたりするので、Nuxt3.tsでWebアプリをどんどん作っていきたいなと思います！

## 参考

* TypeScript
  * https://typescriptbook.jp/overview/static-type
  * https://future-architect.github.io/typescript-guide/class.html
* TwitterBot
  * https://zenn.dev/ryo_kawamata/articles/github-trending-bot
* ローカル実行
  * https://zenn.dev/fukutan/articles/44e59aff1c76dc
  * https://cloud.google.com/pubsub/docs/emulator?hl=ja
* 定期実行
  * https://firebase.google.com/docs/functions/schedule-functions?hl=ja
  * https://qiita.com/Styled-Panda/items/03e229a1c4a52de07c97
* Axios
  * https://github.com/axios/axios
  * https://zenn.dev/mkt_engr/articles/axios-req-res-typescript
* TwitterAPI
  * https://www.npmjs.com/package/twitter-api-v2
  * https://github.com/plhery/node-twitter-api-v2/blob/HEAD/doc/examples.md
