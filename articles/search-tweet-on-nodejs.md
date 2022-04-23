---
title: "Twitter-API-v2ライブラリでTwitter検索してみる"
emoji: "😆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Node.js", "TypeScript", "Twitter", "Firebase", "Cloud Functions"]
published: true
---

## はじめに

Twitter-API-v2ライブラリから、[ツイート取得のExample](https://github.com/plhery/node-twitter-api-v2/blob/HEAD/doc/examples.md#stream-tweets-in-real-time)を試してみる

## インスタンス生成

- TwitterApiのインスタンスを生成
  - 現状TwitterAPIのv1とv2どちらも対応している
  - v1スタイル
    - access_keyなどの4つのkeyやtokenでの生成
  - v2スタイル
    - bearer-tokenの1つでの生成

- 今回はv2スタイルで生成してみた
(ちなみに、今後はv2が主流になっていくとTwitterは発表している)

```ts
const client = new TwitterApi('<YOUR-TWITTER-BEARER-TOKEN>');

// Read+Write level
const rwClient = client.readWrite;

// Read-only level
const roClient = client.readOnly;
```

## ツイートの取得

[公式Doc](https://github.com/plhery/node-twitter-api-v2/blob/HEAD/doc/examples.md#stream-tweets-in-real-time)よりexample

```ts
// Get and delete old rules if needed
const rules = await client.v2.streamRules();
if (rules.data?.length) {
  await client.v2.updateStreamRules({
    delete: { ids: rules.data.map(rule => rule.id) },
  });
}

// Add our rules
await client.v2.updateStreamRules({
  add: [{ value: 'JavaScript' }, { value: 'NodeJS' }],
});

const stream = await client.v2.searchStream({
  'tweet.fields': ['referenced_tweets', 'author_id'],
  expansions: ['referenced_tweets.id'],
});
// Enable auto reconnect
stream.autoReconnect = true;

stream.on(ETwitterStreamEvent.Data, async tweet => {
  // Ignore RTs or self-sent tweets
  const isARt = tweet.data.referenced_tweets?.some(tweet => tweet.type === 'retweeted') ?? false;
  if (isARt || tweet.data.author_id === meAsUser.id_str) {
    return;
  }

  // Reply to tweet
  await client.v1.reply('Did you talk about JavaScript? love it!', tweet.data.id);
});
```

それぞれの要素について見ていく

### 取得するルールの設定

#### 前回のルールが残っていたら削除する処理

```ts
// Get and delete old rules if needed
const rules = await client.v2.streamRules();
if (rules.data?.length) {
  await client.v2.updateStreamRules({
    delete: { ids: rules.data.map(rule => rule.id) },
  });
}
```
- streamRulesメソッドを辿ってみる
  - [GET /2/tweets/search/stream/rules](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream-rules)とつながっている

```ts
    /**
     * Return a list of rules currently active on the streaming endpoint, either as a list or individually.
     * https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream-rules
     */
    streamRules(options?: Partial<StreamingV2GetRulesParams>): Promise<StreamingV2GetRulesResult>;
    /**
     * Add or delete rules to your stream.
     * To create one or more rules, submit an add JSON body with an array of rules and operators.
     * Similarly, to delete one or more rules, submit a delete JSON body with an array of list of existing rule IDs.
     * https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/post-tweets-search-stream-rules
     */
    updateStreamRules(options: StreamingV2AddRulesParams, query?: Partial<StreamingV2UpdateRulesQuery>): Promise<StreamingV2UpdateRulesAddResult>;
```
- GET /2/tweets/search/stream/rulesのresponseの一例
  - 前回検索時の検索条件をreturn
```ts
  {
  "data": [
    {
      "id": "1165037377523306497",
      "value": "dog has:images",
      "tag": "dog pictures"
    },
    {
      "id": "1165037377523306498",
      "value": "cat has:images -grumpy"
    }
  ],
  "meta": {
    "sent": "2019-08-29T01:12:10.729Z"
  }
}
```
- updateStreamRulesメソッドを辿ってみる
  - [POST /2/tweets/search/stream/rules](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/post-tweets-search-stream-rules)とつながっている
  - streamRulesのPost版
    - streamRulesのaddやdeleteができる

#### 検索したいWordを追加する処理

```ts
// Add our rules
await client.v2.updateStreamRules({
  add: [{ value: 'JavaScript' }, { value: 'NodeJS' }],
});
```
- 条件として`JavaScript`もしくは`NodeJS`を含むツイートとしている

### 設定したWordで検索する

- 公式のexampleでは
  - `JavaScript`もしくは`NodeJS`が含まれているツイート
  - RTと自分がツイートしたもの以外

  にreplyを送っている


```ts
const stream = await client.v2.searchStream({
  'tweet.fields': ['referenced_tweets', 'author_id'],
  expansions: ['referenced_tweets.id'],
});
// Enable auto reconnect
stream.autoReconnect = true;

stream.on(ETwitterStreamEvent.Data, async tweet => {
  // Ignore RTs or self-sent tweets
  const isARt = tweet.data.referenced_tweets?.some(tweet => tweet.type === 'retweeted') ?? false;
  if (isARt || tweet.data.author_id === meAsUser.id_str) {
    return;
  }
```

- searchStreamメソッドを辿ってみる
  - [GET /2/tweets/search/stream](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream)とつながっている
    - Query parameterは色々設定できる
      - expansions
      - media.fields
      - tweet.fields など

- ツイートにreplyする処理は今回の趣旨とずれるので割愛

## 実行結果

- replyは迷惑すぎるので、console.log(tweet)にして確認してみる

```sh
>  {
>    data: {
>      author_id: '84208574',
>      id: '1517746807451037696',
>      text: 'Hello guys,\n' +
>        'Does anyone know where it is a company that has whatsapp api services?\n' +
>        'I only need to integrate with my backend (node + express) and use it as an outgoing message.\n' +
>        'I preferred pay as go (per message)\n' +
>        'Have a green check wa account\n' +
>        '\n' +
>        '#Nodejs #whatsappAPI #WebDevelopment'
>    },
>    matching_rules: [ { id: '1517746830268067840', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '731117954188775424',
>      id: '1517746817114554368',
>      text: 'Siempre habra un color para cada momento del dia, ahorita mismo estoy muy #5df497\n' +
>        '\n' +
>        ' #bot #nodejs #random #color #face #emoji #programacion #meme #api https://t.co/Zioq4MbP3B'
>    },
>    matching_rules: [ { id: '1517746830268067840', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '1499252873103695873',
>      id: '1517746858780626945',
>      text: 'うーいんだよ～\n' +
>        '\n' +
>        'これはオススメだね～\n' +
>        'だってプログラムの書き方や考え方が、 うーいんの中の人と同じ思想だからね～\n' +
>        '\n' +
>        'スラスラ読める JavaScript ふりがなプ ログラミング (ふりがなプログラミングシリーズ) https://t.co/qc0ljSyx80\n' +
>        '\n' +
>        '#プログラミング勉強中'
>    },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '69941978',
>      id: '1517746856687980544',
>      referenced_tweets: [ [Object] ],
>      text: "@_gulam_anas_ Thanks Anas for the feedback. Yeah I have considered people's feedbacks from the similar post I made last month. Based on those, I have added/rectified\n" +
>        '\n' +
>        '&gt; Other Skills\n' +
>        '&gt; Practice\n' +
>        '&gt; Frequently Asked Questions\n' +
>        '\n' +
>        'I also added links to HTML/CSS and JavaScript individual roadmaps.'
>    },
>    includes: { tweets: [ [Object] ] },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '1401391924754325512',
>      id: '1517746862173925377',
>      text: 'UDP is better in the COVID era since it avoids unnecessary handshakes.\n' +
>        '\n' +
>        '#programming #programmingjoke #programminghumor #Python #javascript #Java'
>    },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '1078992647933448192',
>      id: '1517746892800544769',
>      text: 'Change file name from backend to server https://t.co/NK0LynUzuM #github #JavaScript #SCSS #HTML #CSS'
>    },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '950931957860286464',
>      id: '1517746905190649856',
>      text: '【人気！】\n' +
>        'カテゴリー： プログラミング\n' +
>        '「ゲームを作りながら楽しく学べるHTML5+CSS+JavaScriptプログラミング［改訂版］ (Future Coders（NextPublishing）)」\n' +
>        '44%OFF / 821円\n' +
>        'https://t.co/ACHcL4ZSGa'
>    },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '1163939968559058944',
>      id: '1517746926191525888',
>      referenced_tweets: [ [Object] ],
>      text: '@sourav_code Well explained 👌'
>    },
>    includes: { tweets: [ [Object] ] },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
>  {
>    data: {
>      author_id: '13821792',
>      id: '1517746998015033344',
>      referenced_tweets: [ [Object] ],
>      text: 'Javaが出てきた時はOSを代替すると 言われて期待されたけど、アップレットとかまとも に動かなくてサーバーサイドにピボットした。\n' +
>        'ネスケのJavascriptは最初バグだらけで 使い物にならなくて、10年くらいたってやっとまと もに使えるようになった。\n' +
>        '技術が成熟するには時間がかかるね。 https://t.co/AY92UoXZqd'
>    },
>    includes: { tweets: [ [Object] ] },
>    matching_rules: [ { id: '1517746830268067841', tag: '' } ]
>  }
!  functions: Your function timed out after ~60s. To configure this timeout, see
      https://firebase.google.com/docs/functions/manage-functions#set_timeout_and_memory_allocation.
```

- Cloud Functions for Firebaseで試した結果、取得はできたが途中でタイムアウトしてしまった
  - 取得するツイートが多すぎる？
    - 取得するツイート数を調整すればちゃんと検索結果が取れるのかなと思う
  - Cloud Functionsのタイムアウト設定などは以下を参考にする
    - https://firebase.google.com/docs/functions/manage-functions#set_timeout_and_memory_allocation

- 検索文字の大文字小文字は気にしない模様

## おわりに

- Twitter-API-v2を使えば簡単にTwitterAPIを操作できる（ただしTwitterAPIの仕様を読み込む必要はある...）
- 今後はタイムアウトしないように調整しつつ、アプリ開発で使っていきたい

## 参考

- [Twitter-API-v2 Examples](https://github.com/plhery/node-twitter-api-v2/blob/HEAD/doc/examples.md#stream-tweets-in-real-time)
- Twitter API
  - [GET /2/tweets/search/stream/rules](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream-rules)
  - [POST /2/tweets/search/stream/rules](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/post-tweets-search-stream-rules)
  - [GET /2/tweets/search/stream](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream)
