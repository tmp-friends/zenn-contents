---
title: "[論文メモ] LFRR"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習"]
published: true
---

## Latent Factor Models and Aggregation Operators for Collaborative Filtering in Reciprocal Recommender Systems

- 論文: [https://github.com/tangxyw/RecSysPapers/blob/main/Industry/Reciprocal/Latent Factor Models and Aggregation Operators for Collaborative Filtering in Reciprocal Recommender Systems.pdf](https://github.com/tangxyw/RecSysPapers/blob/main/Industry/Reciprocal/Latent%20Factor%20Models%20and%20Aggregation%20Operators%20for%20Collaborative%20Filtering%20in%20Reciprocal%20Recommender%20Systems.pdf)
- 会議: RecSys'19

### 読む動機

- 相互推薦タスクというものの存在を知り、興味を持ったため
- CyberAgentさんの日本国内向けサービスでの事例のため、参考になる箇所があればいいなと思ったたため

### 要約

- 相互(=user-to-user)推薦は一般的なuser-to-item推薦と手法が異なる
- 潜在因子モデルを用いた相互推薦の主要な手法について示す
- 相互推薦におけるSoTAを達成

### 背景

- 相互推薦の需要が高まっている
    - マッチングアプリ
    - 採用
    - SNS
- 相互推薦は双方向の興味の関係性を考慮する必要があり、難しい
    - ex. 不人気のユーザに人気のユーザを推薦してもダメ
- 提案されてきた手法
    - コンテンツベース
        - RECON
    - 協調フィルタにおける近傍推薦
        - RCF
        - ユーザに対する全ペアを計算することになるので、計算量が大きくなりがち

### 提案手法

### LFRR

潜在因子モデル(=行列分解を用いる)を採用した部分が新規性

![lfrr](/images/lfrr-in-reciprocal-recommender-systems/lfrr.png)

#### ex) 上図のマッチングアプリ

1. 以下の2方向の推薦スコアをSGDにより算出
    - Male-to-Female のスコア
    - Female-to-male のスコア
2. 1.で算出したそれぞれの推薦スコアに対し、集計関数(Aggregation Operators)から相互推薦スコアを算出
    - 候補としては以下の平均三兄弟
        - Arithmetic Mean（算術平均）
        - Geometric Mean（幾何平均）
        - Harmonic Mean（調和平均）

### 有効性

- データセット
    - マッチングアプリPairsにおける３ヶ月分の行動ログ
        - マジョリティに絞りたいなどで以下の制限
            - 東京周辺住み
            - 18 - 30歳
            - 10Like以上のユーザ

![lfrr-experiment-result](/images/lfrr-in-reciprocal-recommender-systems/lfrr-experiment-result.png)

### 感想・備考

- 一般的な推薦なら行列分解するのが主流だけど、相互推薦だと2019年までしてこなかったのに驚いた
- 性能評価などでLikeで閾値を設けるのは参考になるかもしれないと思った
- レコメンドの最適化関数にSGDを使ったことがなかったので勉強になった

### 参考

- [Python: レコメンドの行列分解を確率的勾配降下法で実装してMovieLens100Kに適用する](https://ohke.hateblo.jp/entry/2018/01/13/230000)
