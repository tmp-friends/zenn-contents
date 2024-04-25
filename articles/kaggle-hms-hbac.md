---
title: "Kaggle-HMSコンペ 上位解法まとめ(+参加振り返り)"
emoji: "🦆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kaggle"]
published: true
publication_name: "dmmdata"
---

## はじめに

2024/01/08~2024/04/08に開催されていたKaggleのHMS - Harmful Brain Activity Classificationコンペに参加していました。

https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification

https://twitter.com/temple_c_tech/status/1777488184903086260

復習として上位解法をまとめつつ、自身の取り組みについての振り返りを書いていきたいと思います。

## コンペ概要

重症の患者から記録された脳波信号(EEG)から、有害な脳活動を検出・分類する

- Data
    - 以下の2種類
        - 脳波信号(EEG)
            - 50s
        - 脳波スペクトログラム(Spectrograms): 脳波信号を短時間フーリエ変換(STFT)したもの
            - 10m (中央の50sは脳波信号と同じもの)
    - 詳しく: https://www.kaggle.com/code/suzukisatsuki/hms-eda
- Label
    - 以下の6種類
        - SZ: seizure(発作)
        - LPD: lateralized periodic discharges(側方化周期性放電)
        - GPD: generalized periodic discharges(汎発性周期性放電)
        - LRDA: lateralized rhythmic delta activity(側方化リズミックデルタ活動)
        - GRDA: generalized rhythmic delta activity(汎発性リズミックデルタ活動)
        - other

![example-data](/images/kaggle-hms-hbac/example-data.png)

- アノテーション
    - 専門家グループによって行われた
    - 手作業のため、セグメントによって専門家間で一致したり、そうでなかったりする
- 評価指標
    - KL Divergence: 観測データの分布の情報エントロピー - 推定モデルの情報エントロピー
    - 詳しく: https://qiita.com/shuva/items/81ad2a337175c035988f
- Submission
    - eeg_idに対し、6つの検出対象の確率


## 上位解法まとめ

### Model
#### 2D Model

- 使用されていた主なモデル
    - ConvNeXt
    - InceptionNeXt
    - TinyViT
    - SwinTransformer
- [timm](https://github.com/huggingface/pytorch-image-models)

- [1st](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492560)の入力処理
    - 脳波信号(EEG)を並べて2D画像にし処理する
        - 双極性モンタージュの18の信号を3つの異なる区間で切り出して、resizeして1つの画像として連結
        - 双極性モンタージュの18の信号をCWT(Continuous Wavelet Transform)
    - 1D畳み込みを使用して、特徴マップを取得 -> 縦に結合して、2DCNNの入力にする
        - 特徴マップをそのままGRUに入力し、アンサンブルの入力にもできる
    - SuperletsによるCWT
        - 参考: https://zenn.dev/bilzard/articles/stft-wavelet-superlet

#### 3D Model

[2nd](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492254)のユニークな解法

- 使用されていた主なモデル
    - X3D
- 脳波スペクトログラム(Spectrograms)をSTFT
- 3DCNNでは、channelの位置に敏感であるため、今回のLabel(LPD, GPDなど)に対して効いたそう

### Train

#### Augmentation

- XYmasking
- Mixup
- Mirror(左脳と右脳のデータを反転)

など

#### 2stage training

- アノテーター(専門家)が確信をもって判断しているものと、そうでないものにデータセットを分けて学習
    - コンペ期間中は、この2stage trainingだと学習データにOverfitしてしまいshakeが起こるのでは、という議論が起こっていた
    - テストデータも学習データと同じような分布を取っていた かつ 同じような分布であると予想できたため、shakeは起こらなかった
    - 詳しく: https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492262
#### PseudoLabeling

[8th](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492482)

- Labelのついている数が10未満のものに対して、PseudoLabeling

#### Ensemble

- 単純な重み付け
    - [3rd](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492471)はNNで最適な重み付けの値を探索している
- [1st](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492560)は非線形回帰
    - 今回のコンペのデータが学習データに対してOverfitしてもCVとPublicLBとの相関が維持される特性があったため

#### Loss

- KLDivergenceLoss
    - コンペ中の公開Notebookでは主にこの損失関数が用いられていた
    - 評価指標がKLDivergenceのため
- entmax
    - softmaxに比べ、スパースな出力ができる
    - softmaxだとすべてのLabelに対する出力が0ではなくなってしまう
    - 多くの学習データのLabelには0のクラスがあるため、採用

### その他

- [3rd](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification/discussion/492471)のCVの切り方がよく練られていた

## 参加してみての振り返り

### Keep

- 継続してコンペに取り組むことができた
    - 以前に出たコンペでは、公開notebookを写経して1subで終了みたいなことが多かった
    - 継続できるようになったのは、以下の点が考えられそう
        - ローカル環境でKaggleをできるようにして、開発体験が向上した
        - 転職してフルリモートになり、可処分時間が増えた
        - 転職先での強いKagglerの存在
- メダルを取ることができた
    - 『Kaggleで勝つ データ分析の技術』やDiscussionを読んで、PseudoLabelingを使った解法を実装することができた

### Problem

- 過去の類似コンペの解法について調べる量が足りてなかった
- 画像系のコンペだったので、ほぼ知見がなく苦戦した

### Try

- 過去の類似コンペの上位解法は読むようにする
- 業務で扱っているテーブルデータやNLPのほうがとっつきやすいと思うので、その領域のコンペを中心に参加していくようにする
- ContinuousDeployment(?)を整備する
    - GitHubにソースコードをpushしたらGHAでKaggleDatasetを反映させる仕組みを作りたい

## おわりに

- 上位解法を何個か読んだが、モデルへの入力周りは全然ピンとこなかった...
- 継続して取り組めば、メダルくらいは取れることがわかったのが一番の収穫だった
- PrivateLB公開時に普段の生活からは考えられないくらいの脳汁が出るので、今後もやっていきたいと思う

## 参考

テンプレートとして参考にさせていただきました
https://zenn.dev/yume_neko/articles/5bc998cfd8cd05
