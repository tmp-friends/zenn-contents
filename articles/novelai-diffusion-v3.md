---
title: "[論文メモ] NovelAI Diffusion V3 技術レポート"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "機械学習", "stablediffusion"]
published: false
---

## Improvements to SDXL in NovelAI Diffusion V3

- 論文URL: https://arxiv.org/abs/2409.15997v1
- 会議: -

### 読む動機

- NovelAI Diffusion V3がどのように学習させているのか (特に学習時の工夫やマシンリソースの観点で) 気になったから

### 要約

- SDXLをbaseにし、NovelAI Diffusion V3を学習させた際の技術レポート

### 背景

- 近年、Diffusionベースの画像生成モデルは急速に発展していて、特にOpen SourceであるStable Diffusionが大きな人気を集めている
- Stable Diffusionを拡張したSDXLは、精度の高い画像生成を可能にしている
- しかし、SDXLは改良の余地がある

### 提案手法

#### 工夫
- v-Prediction Parameterization
    - 後述するZero Terminal SNRを補助するため
    - 従来のε-predictionだとSNR=0 (=完全なノイズ) から画像を推論する方法をモデルに教えることができない
    - 一方、v-predictionだとε-predictionからx0-predictionへの切り替えも適切なタイミングで行うことができ、low SNRのtimestepでもモデルの予測が単純にならない
    - これにより、lossが安定し、高解像度画像生成時のcolor-shiftingが抑えられるようになった
- Zero Terminal SNR
    - 従来のnoise scheduleでは画像生成時に常に中程度の (不必要な) 明度が設定されてしまう
        - “純粋な信号” から “純粋なノイズ” のnoise scheduleにするべき
        - これは推論時に “純粋なノイズ” から画像を生成するのに弊害

        ![fig1](/images/novel-ai-diffusion-v3/fig1.png)

    - NAI v3では学習時のnoise scheduleにZero Terminal SNRを採用
        - “純粋なノイズ” で学習できる
        - Fig2のように黒一色の画像も生成できるようになる
            - =中程度の (不必要な) 明度が設定されない

        ![fig2](/images/novel-ai-diffusion-v3/fig2.png)

        - イラストの生成でもFig2と同様

        ![fig3](/images/novel-ai-diffusion-v3/fig3.png)

- Sampling from High-Noise Timesteps Improves High-Resolution Generation
    - 高解像度画像だとより多くのノイズが必要になる
    - 従来のnoise scheduleだと$\sigma_{max}$が14.6と小さい
        - カメラに近い手や足などの大きな部分の一貫性が保たれなくなることがある
    - (経験則として) キャンバス長が2倍になると$\sigma_{max}$も2倍にするべき

    ![fig5](/images/novel-ai-diffusion-v3/fig5.png)

- MinSNR
    - 拡散過程をmulti-task学習として捉え、各timestepで異なるlossの重み付けをした
    - low SNRのtimestep時に学習が集中することを避けた

#### データセット

600万枚のイラスト

#### 学習
256 x H100 クラスタ上で、約 75k H100 hours 学習

### 有効性

- NovelAI Diffusion V3は、従来のモデルより低いCFGスケール (3.5 ~ 5) で、関連性の高い一貫した画像を生成することができる
    - データセットのラベリングがより良くなっていることを示唆している

### 次に読むべき論文

- SDXL: https://arxiv.org/abs/2307.01952

### 備考

- SNR (=Signal-to-Noise Ratio): 信号とノイズの比率

- Lossについて
    - ε-prediction
        - モデルは元画像 (x0) に加えられたノイズ (ε) を予測
    - x0-prediction
        - 直接元画像 (x0) を予測
    - v-prediction
        - 元画像 (x0) とノイズ (ε) の重み付き和を予測

### 感想

- SNRの改善がメインっぽい
- NAIはInpaintingモデルの性能が良いイメージがあったが、ベースモデルに対しても色々と工夫していることを知れて面白かった
    - NovelAI Diffusion V3をベースモデルとして、FurryDiffusionV3, Director Tools, Inpaintingモデルなどを作っているそう
- 学習時のマシンリソースが膨大すぎる

### 参考

https://medium.com/@zljdanceholic/three-stable-diffusion-training-losses-x0-epsilon-and-v-prediction-126de920eb73
