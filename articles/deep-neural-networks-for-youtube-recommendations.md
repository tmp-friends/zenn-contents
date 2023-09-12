---
title: "[論文メモ] Deep Neural Networks for YouTube Recommendations"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習"]
published: true
---

## Deep Neural Networks for YouTube Recommendations

- 論文URL: https://static.googleusercontent.com/media/research.google.com/ja//pubs/archive/45530.pdf

### 読む動機

- DNN(=DeepNeuralNetwork)を用いたレコメンドシステムを構築したいため
- Youtubeのレコメンドシステムについて気になったため
  - 産業界での使われ方の部分

### 要約

- 2stageのDNNからなるレコメンドシステムを構築し、性能向上を実現した
  - candidate generation: 候補生成
  - ranking: ランキング

![two-stage](/images/deep-neural-networks-for-youtube-recommendations/two-stage.jpg)

### 背景

- Youtubeでは以下の3つの観点を重要視している
  - Scale: 膨大なユーザベースとコーパスを扱うには高度に専門化された分散学習アルゴリズムと効率的なServingシステムが不可欠
  - Freshness: 1秒間に何時間もの動画がアップロードされるので、それらの取り扱い
  - Noise: ユーザ行動はスパース性と様々な外的要因に左右され、推論が困難

### 提案手法

2stage recommendation system

1. candidate generation
2. ranking

#### 1. candidate generation

- 膨大なYoutubeコーパスを、ユーザに関連する可能性のある数百の動画に絞り込むstage
- 置き換える前はMatrixFactorization(=MF)だった箇所
  - DeepLearningによるMFの非線形一般化ととらえることができる

![candidate](/images/deep-neural-networks-for-youtube-recommendations/candidate.jpg)

**問題設定**

レコメンドを以下のように多クラス分類としてとらえる

- ユーザ$U$とコンテキスト$C$に基づき、コーパス$V$の数百万の動画$i$（クラス）の中から、時刻$t$の特定の動画$w_{t}$を正確に分類すること
- $P(w_{t} = i|U,C) = \dfrac {e^{v_{i}u}}{\Sigma_{j\in V}e^{v_{j}u}}$
  - $u$: ユーザ×コンテキストの高次のEmbedding
  - $v_{j}$: 各候補動画のEmbedding

**データ**

- ユーザの閲覧履歴(implicit feedback)を主に用いる
- implicit feedbackが桁違いに多く、explicit feedbackが極端に少ないため

**効率化**

- クラス数が数百万~とあるので、データの背景にある分布”candidate sampling”から重みづけをするNegativeSamplingを行う
  - 代替アプローチとして知られる階層型ソフトマックスでは精度がでなかった
- 最近傍探索アルゴリズムを使用した推論
  - 推論時には、softmax層からの尤度は必要なくなるため

**モデル**

- BoWモデルを参考にしている
- SequentialなEmbeddingで各ユーザの視聴動画IDを表現する
  - 密にするために、平均化を行う
  - 誤差逆伝播によって、他の特徴量も考慮した学習が行われる
- DNNでMFを近似することにより、特徴量を容易にモデルに追加できるようになった

**学習のコツ**

- ランダムな一点の視聴を予測するより、ユーザの視聴を予測する方が性能が上がった
  - ユーザは広く人気のあるものから動画を見たり、とてもニッチなものを見たりと非対称な消費を行うため
  - 多くの協調フィルタでは、これができていない


#### 2. ranking

- 目的関数に従って、各動画にスコアを割り当てることで精度の高いレコメンドを行うstage
- candidate generationで絞られたので、多くの動画とユーザを表す特徴量を用いることができる
- 視聴時間で重みづけを行うことで効果的なレコメンドとなった
  - サムネ詐欺のようなコンテンツがレコメンドで出にくくなった
  - weighted logistic generation

![ranking](/images/deep-neural-networks-for-youtube-recommendations/ranking.jpg)

### 有効性

Youtubeで効果が出たそう

### 次に読むべき論文

- Youtubeのレコメンドシステムに関する論文
  - Latent Cross: Making Use of Context in Recurrent Recommender Systems
  - Top-K Off-Policy Correction for a REINFORCE Recommender System

### 感想

- DNNによるレコメンドということで有名な論文で、読みたかったので良かった
- たくさんの種類の説明変数を用いていて、これを眺めるだけで面白い(視聴履歴, 検索履歴, 地理的情報, 年齢, 性別, etc...)
- DeepLearningでMFを近似するというアイディアに基づいている、という部分をもっと深堀ろうと思った
- 次に視聴するアイテムを予測する(=時系列データを扱う)ということで気を付けないといけない点などありそうだなと思った
  - リークとか？
  - TODO: 時系列データの取り扱いについて、書籍などで調べる
- Rankingで視聴時間に対して重みづけを行うのが、レコメンドで最大化したい指標への気持ちを込めていて、参考になった
- レコメンドをシステムに組み込む際には、精度だけでなく大規模データに対する効率性なども気にする必要があることがわかった

### 参考

- https://medium.com/eureka-engineering/youtube-recommender-algorithm-survey-341a3aa1fbd6


