---
title: "[論文メモ] 画像生成AIのパーソナライズ? Textual Inversion"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "機械学習", "stablediffusion"]
published: false
---

## An Image is Worth One Word: Personalizing Text-to-Image Generation using Textual Inversion

- 論文URL: https://arxiv.org/abs/2208.01618
- 会議: -
- GitHub: https://github.com/rinongal/textual_inversion/tree/main

### 読む動機

- niji journeyでパーソナライズ機能がリリースされ、StableDiffusionにおけるパーソナライズでいうと、Textual Inversionが思い浮かんだので精読しておこうと思ったため
    - （niji journeyのパーソナライズ機能でTextual Inversionが使われているかは不明）

https://twitter.com/nijijourney/status/1832189431094096253

### 要約

- Text-to-imageモデルは、特定のユニークな概念を自然言語のみで指示して生成するのが難しい
- ユーザが提供した3〜5枚の画像を、新しい”擬似単語”としてText-to-imageモデルのText EncoderのEmbedding空間で表現するよう学習する手法を提案
- この”擬似単語”は、自然言語の文章に組み込むことで、パーソナライズされた画像生成を直感的に補助する
    - 1つのEmbeddingで、ユニークで多様な概念を十分に捉えることが可能

![abst](/images/textual-inversion/abst.png)

### 背景

- 新たな概念をlarge scaleなモデル(e.g. Text-to-imageモデル)に学習させるのは難しい
    - 新たな概念が生まれるたびにデータセットを拡張しモデルを再学習させるのはコストがかかる
    - 少数のサンプルでfine-tuningするのはモデルの壊滅的な忘却を引き起こす
- 事前学習されたText-to-imageモデルの豊富な表現力を使いたかったので、Text-to-imageモデルで条件付けを行う部分であるText Encoderのword-embedding stageに目をつけた

### 提案手法

Textual Inversion

![architecture](/images/textual-inversion/architecture.png)

- 新しい概念（S∗）をText Embeddingとして表現し、Latent Diffusion Model (LDM) の損失を最小化するよう最適化を行う
    - 元のLDMの学習スキーマ (※備考参照) をそのまま用いている
- これにより、新たな概念を既存のモデルに統合することが可能になる

- 数理化
    - t: time step
    - $z_{t}$: tにおける潜在ノイズ
    - $\epsilon$: （スケールなしの）ノイズサンプル
    - $\epsilon_{\theta}$: denoising network
    - $c_{\theta (y)}$: 条件付けの入力yをベクトルへ写像するモデル

$$
v_* = \arg \min_{v} \mathbb{E}_{z \sim \mathcal{E}(x), y, \epsilon \sim \mathcal{N}(0,1), t} \left[ \| \epsilon - \epsilon_{\theta}(z_t, t, c_{\theta}(y)) \|_2^2 \right]

$$

### 有効性（定性評価）

- ImageVariations
    - 人間がキャプションした既存のSoTA手法との比較
    - 既存手法に比べて、新たな概念を捉えて出力できている
    - 既存手法は、特にCLIPに存在しないような概念だと苦戦している
        - 一方でCLIPに存在するような概念だと、良い出力をすることが多い

    ![result_image_variations](/images/textual-inversion/result_image_variations.png)

- Text-guided synthesis
    - （割愛）
- Style transfer
    - 画風も捉えられる

    ![result_style_transfer](/images/textual-inversion/result_style_transfer.png)

（定量評価も行っていたが割愛）

### 次に読むべき論文

- Clip
- Unet
- DiffusionModel
    - LDMs: https://openaccess.thecvf.com/content/CVPR2022/html/Rombach_High-Resolution_Image_Synthesis_With_Latent_Diffusion_Models_CVPR_2022_paper.html
    - DDPMs

### 備考

- LDM loss

$$
v_* = \mathbb{E}_{z \sim \mathcal{E}(x), y, \epsilon \sim \mathcal{N}(0,1), t} \left[ \| \epsilon - \epsilon_{\theta}(z_t, t, c_{\theta}(y)) \|_2^2 \right]

$$

### 感想

- niji journeyのパーソナライズ機能は本論文と同様にTextEncoderによる条件付けで実現していそう
    - 一方で、Textual Inversionのような処理を通すと出力画像の精度が落ちることが多いので、どうやって精度を保っているのか気になった
- GANのInversionから着想を得たらしいが、GANまわりは全く知見がないのでよくわからなかった
- 大量のコンテンツ内から手間をかけることなく目的のコンテンツを見つけるのは検索やレコメンドの分野であり、生成AIの時代には今までにないペースでコンテンツがでてくるようになるため、検索やレコメンドがより重要になってくるのではないかと思っている

### 参考

https://huggingface.co/docs/diffusers/main/en/training/text_inversion
