---
title: "Twitter Recommendation Algorithm"
emoji: "🐦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "github"]
published: false
---

## はじめに

Twitterのレコメンドアルゴリズムが公開されました

https://github.com/twitter/the-algorithm

https://github.com/twitter/the-algorithm-ml

公開されてから二日間ほどコードを追っていたので、一部にはなりますがまとめようと思います

## 全体像

https://blog.twitter.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm

- データ量
  - 5億ツイート/日

- レコメンドシステム構成
  - Multi-stage
    1. Candidate Sources(=候補生成)
    2. Ranking(=ランク付け)
    3. Heuristics,Filters,and ProductFeatures(=フィルタリング)

![/images/twitter-recommendation-algorithm/open-algorithm.png.img.fullhd.medium.png](/images/twitter-recommendation-algorithm/open-algorithm.png.img.fullhd.medium.png)

### 補足) Multi-stageレコメンドシステム

ステージを複数用意し、効率的にレコメンドするアイテムを絞っていくシステムの構成

以下のRecsys2022のスライド資料などがわかりやすいです

https://librerank-community.github.io/slides-recsys22-tutorial-neuralreranking.pdf

## Candidate Sources

- 概要
  - この層ではアイテムを大雑把に絞ることが目的なので、精度よりスピードが求められる
  - 具体的には、数億あるアイテムの中から1500までに絞るそう
  - ScalaとたまにPython


- データ
  - In-Network-Sources: フォローしているユーザ
  - Out-of-Network-Sources: フォローしていないユーザ

In-Network-SourcesとOut-of-Network-Sourcesを1:1でレコメンドする

- Candidate Sourcesでの代表的なシステム
  - In-Network-Sources
    - Real Graph
      - ユーザ間のエンゲージメントの可能性を予測するモデル
      - https://github.com/twitter/the-algorithm/tree/main/src/scala/com/twitter/interaction_graph
  - Out-of-Network-Sources
    - GraphJet
      - フォローしていないのに、あるツイートが自分に関連するかどうかを判断するのが難しい
      - -> 2つのアプローチをとっている
        1. 自分がフォローしている人達がどんなツイートをしたか
        2. 自分と似たような'いいね'をしている人たちは他にどんなツイートに'いいね'しているか
    - SimClusters


## Ranking

- LIGHT RANKER
- HEAVY RANKER
    - [https://github.com/twitter/the-algorithm-ml/tree/main/projects/home/recap](https://github.com/twitter/the-algorithm-ml/tree/main/projects/home/recap)

### 補足) torchrecとは

[https://pytorch.org/torchrec/index.html](https://pytorch.org/torchrec/index.html)

PyTorchが提供する大規模なDeepLearningレコメンドシステムを構築する際に必要な機能を提供するライブラリ

具体的に以下のことが簡単にできるようになる

- multi-device
- multi-node
- data-parallelism
- model-parallelism

### HEAVY RANKER

Candidate Sourcesを経て1500件程度に絞られたツイートの中でRankingをつける

#### 入力特徴量

https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/FEATURES.md

#### モデル

- ****MaskNet: Introducing Feature-Wise Multiplication to CTR Ranking Models by Instance-Guided Mask****
    - [https://arxiv.org/abs/2102.07619](https://arxiv.org/abs/2102.07619)
        - CTRの推定を行うには複雑な高次の特徴をモデルを用いて捉える必要がある
        - DNN(=DeepNeuralNetwork)モデル(ex. FNN, DeepFM, xDeepFM)は高次の特徴を捉えるのに使われている
        - しかし、一部共通の特徴量を捉えるためにfeed-forward層が非効率なことがわかっている
        - そこで、instance-guided maskというembedding層とfeedforward層の両方で要素ごとの積を計算する処理を提案
        - MaskNetではDNNモデルにMaskBlockというレイヤー正規化、instance-guided mask、feed-forward層を組みあわせた構造を導入してみた
        - 結果、DeepFMやxDeepFMなどの性能を超えSOTA
    - PyTorchでの再現実装
        - [https://github.com/xue-pai/FuxiCTR](https://github.com/xue-pai/FuxiCTR)
    - TwitterでのMaskNetの実装部分
        - https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/model/mask_net.py

## おわりに

- 情報量が多すぎてRanking部分しかろくに見れていないですが、他も見ていきたいです
- torchrecの存在は知っていたのですが、ネット上に情報がなさすぎたので、Twitterで使われていて驚きました
- Twitterレベルの大規模なシステムのレコメンドの実装が見られるのは貴重ですし、面白いので是非ご覧になってみてください


