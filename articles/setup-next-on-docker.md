---
title: "DockerでNext.jsの環境構築をする"
emoji: "👍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "typescript", "docker", "eslint", "prettier"]
published: true
---

## はじめに

DockerでNext.jsの環境構築を行いました

- Next.js
  - TypeScript
  - ホットリロード機能
  - ESLintの設定
  - Prettierの設定

**最終的なディレクトリ構成**

```
.
├── .vscode
│   └── .settings.json
├── src
|   ├── <Next.jsのコード>
|   ├── .eslintrc.json
│   └── .prettierrc
├── .dockerignore
├── docker-compose.yml
└── Dockerfile
```

## Next.js on Docker

### Dockerfile

```Dockerfile:Dockerfile
FROM node:lts-buster-slim

WORKDIR /app
```

### docker-compose.yml

```yml:docker-compose.yml
version: '3'
services:
  app:
    build:
      context: .
    tty: true
    volumes:
      - ./src:/app
    ports:
      - "3000:3000"
    command: sh -c "npm run dev"
```

- volumes
  - srcディレクトリ内にNext.jsアプリを作成していくため、`./src:/app`としています

### create-next-app

```
docker-compose build

docker-compose run --rm app sh -c 'npx create-next-app src --typescript'

docker-compose up
```

これでアプリの雛形をsrcディレクトリ内に作成でき、サーバーを起動できていることが確認できました

### ホットリロードの設定

Next.jsのホットリロード機能を有効にするため、docker-compose.ymlで環境変数を設定しました

```diff yml:docker-compose.yml
version: '3'
services:
  app:
    build:
      context: .
    tty: true
    volumes:
      - ./src:/app
    ports:
      - "3000:3000"
+   environment:
+     - WATCHPACK_POLLING=true
    command: sh -c "npm run dev"
```

- [watchpack](https://github.com/webpack/watchpack)
  - webpackに含まれているライブラリ
  - fileを監視する役割

- ちなみに、最初はnext.config.jsに以下のような`watchOptions`を記載するやり方を真似したのですが、採用しませんでした

```js:next.config.js
module.exports = {
  webpackDevMiddleware: config => {
    config.watchOptions = {
      poll: 800,
      aggregateTimeout: 300,
    }
    return config
  },
}
```

採用しなかった理由としては以下です

- コード変更から一定時間たたないとホットリロードによる変更が開始しない（`poll: <秒数>`のため）
- ホットリロードが完了するのに数秒かかる（再コンパイルが発生するため？）

## リンター・フォーマッターの設定

そもそもなぜESLintだけでなくPrettierも使用するのか、といったことに疑問を持ったのですが、以下の記事が参考になりました

https://qiita.com/soarflat/items/06377f3b96964964a65d

まとめると、

- ESLintで整形しきれない部分をPrettierが補える
- ESLintよりPrettierの方が手軽で確実に整形できる

とのことでした

### ESLintの設定

- ESLint
- Next.jsでのESLint

をインストールします

```
docker-compose run --rm app npm i --save-dev eslint eslint-config-next
```

- 上記コマンドを実行すると対話形式で.eslintrc.jsonが作成されました
  - 以下の記事を参考にしました

https://yumegori.com/vscode_react_typescript_eslint_prettier

#### ESLintファイルの修正


- Prettierを導入するために追記
  - Prettierとのルールが重複する部分はESLint側で無効化
- ESLintでエラーが出る箇所を修正
  - ファイルでReactを必ずimportするというルールを無効化
- importするライブラリなどをアルファベット順にする
```diff json:.eslintrc.json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "plugin:react/recommended",
    "standard",
+   "prettier" // eslintとprettierの衝突回避
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["react", "@typescript-eslint"],
  "rules": {
+   "react/react-in-jsx-scope": "off", // v17からReactをimport不要になった
+   // importをアルファベット順にする
+   "import/order": [
+     "error",
+     {
+       "alphabetize": {
+         "order": "asc"
+       }
+     }
+   ]
  }
}
```

### Prettierの設定

- prettier
- eslint-config-prettier

をインストールします

```
docker-compose run --rm app npm i --save-dev prettier eslint-config-prettier
```

- 上記コマンドを実行した後、.prettierrcをルートディレクトリに作成します
  - Prettierの設定一覧は以下の記事を参考にしました

https://www.mitomex.blog/prettier-options/

Prettierの設定は完全に個人の好みですが、載せておきます

```json:.prettierrc
{
  "singleQuote": false,
  "jsxSingleQuote": false,
  "semi": false,
  "trailingComma": "all",
  "arrowParens": "always"
}
```

### VSCodeでのリンター・フォーマッターの設定

拡張機能のインストールと./.vscode/settings.jsonでの設定を行いました

- 拡張機能
  - ESLint
    - リアルタイムでESLintでのエラーを表示
    - 保存と同時に修正(settings.jsonに追記)
  - Prettier
    - 保存と同時に修正(settings.jsonに追記)

```json:./.vscode/settings.json
{
  // ESLint
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  // Prettier
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
}
```

## おわりに

Next.jsでの基本的な環境構築はできたかなと思います。
アプリを作るたびに上記の手順をいちいち行っていくのは面倒くさいので、テンプレート化したいなと思いました。

## 参考

- Next.js on Docker
  - https://zenn.dev/yuki_tu/articles/01c7963eeb2876
- ホットリロード(TypeScript)
  - https://qiita.com/SDTakeuchi/items/694fe7545396246da573
- ESLint&Prettier
  - https://qiita.com/soarflat/items/06377f3b96964964a65d
  - https://yumegori.com/vscode_react_typescript_eslint_prettier
  - Prettierの項目一覧
    - https://www.mitomex.blog/prettier-options/
