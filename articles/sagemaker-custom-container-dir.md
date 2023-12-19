---
title: "SageMakerにおけるカスタムコンテナと推し構成"
emoji: "🧠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "sagemaker", "機械学習"]
published: true
---

この記事は[MLOps Advent Calendar 2023](https://qiita.com/advent-calendar/2023/mlops)の19日目の記事です。

### はじめに

SageMakerで機械学習の運用を行うにあたり、コンテナのカスタマイズが必要になってくることが往々にしてあると思います。

私自身SageMakerにおけるカスタムコンテナをネット上で調べてみたところ、
- あまり記事がない
- 運用まで見据えた部分まで言及されていることが少ない

ということもあり、備忘録もかねてSageMakerでのカスタムコンテナまわりの記事を書いてみます。

本記事では以下の内容について書きます。

- SageMakerにおける学習の流れ
- カスタムコンテナ実装パターン
- SageMakerでの運用における推し構成

### SageMakerにおける学習の流れ

ざっくりですが記載します。

- 学習コードの種類
    - HelperCode
        - SageMakerSDKからTrainingCodeを呼び出す
    - TrainingCode
        - 実際に学習を行うコード
        - ECRからDockerイメージを指定して、インスタンスを起動し処理を実行
        - (本記事で扱う"カスタムコンテナ"の部分)

- 学習の流れ
    - HelperCode → TrainingCode
        - ハイパーパラメータなどの情報と共にトレーニングジョブ開始のリクエストを送る
    - TrainingCode(ECR) → TrainingCode
        - TrainingCodeが格納されたDockerイメージを用いてインスタンスが起動
    - TrainingData(S3) → TrainingCode
        - S3にある学習関連データがインスタンスに配置される
    - TrainingCode
        - インスタンス内で学習処理が実行される
    - TrainingCode → ModelArtifacts(S3)
        - 学習済みモデルをS3へ保存

![sagemaker_architecture](/images/sagemaker-custom-container-dir/sagemaker-architecture.png)

### カスタムコンテナ実装パターン

TrainingCodeのコンテナをカスタマイズしたい、となった際に以下の3パターンのやり方が考えられます。

1. AWSが提供しているコンテナイメージを拡張する方法
2. 独自のコンテナイメージにSageMakerToolkitをインストールする方法
3. スクラッチでコンテナイメージを作成する方法

3.にいくにつれ、カスタム性があがり(ポジティブ要素)、手間がかかります(ネガティブ要素)

![sagemaker_custom_containers_pattern](/images/sagemaker-custom-container-dir/sagemaker_custom_containers_pattern.png)

以下の記事にそれぞれのパターンが詳しく記載されています↓

https://aws.amazon.com/jp/blogs/news/sagemaker-custom-containers-pattern-training/

- 個人的な意見
    - 2.のパターンを用いるのが、運用という観点でやりやすいのかなと思います
    - 1.だとAWSが事前にECRに登録しているDockerイメージを使うことになる
        - ライブラリの追加と独自のTrainingCodeの実行くらいしかできない
    - 3.だと自分で作成したDockerイメージを用いることができるが、Job実行時に渡されるHyperParameterやディレクトリパスの取得を行うコードまで実装する必要があり、コストが高い
    - その点2.では、自分で作成したDockerfileを用いることができる かつ SageMakerToolkitというライブラリを用いることである程度コードを簡素化できる という理由から推しています
        - https://github.com/aws/sagemaker-training-toolkit

    ```py:SageMakerToolkitによるコード例
    from sagemaker_training import environment

    env = environment.Environment()

    # get the path of the channel "training" from the `inputdataconfig.json` file
    training_dir = env.channel_input_dirs["training"]

    # get a the hyperparameter "training_data_file" from `hyperparameters.json` file
    file_name = env.hyperparameters["training_data_file"]

    # get the folder where the model should be saved
    model_dir = env.model_dir
    ```

#### [補足] カスタムコンテナ内部のディレクトリ構造

3パターンありますが、実質どの程度スクラッチで書くかの違いであり、コンテナ内部のディレクトリ構造は以下のように同じです。

```
/opt/ml
├── input
│   ├── config
│   │   ├── hyperparameters.json
│   │   └── resourceConfig.json
│   └── data
│       └── <channel_name>
│           └── <input data>
├── model
│
├── code
│
├── output
│
└── failure
```

- `/opt/ml/input/config`
    - SageMakerAPIで指定した、HyperParameterやマシンリソース(ex. ml.t3.xlarge 1台 など)の設定ファイルが置かれる
- `/opt/ml/input/data`
    - SageMakerAPIでinputとして指定したS3パス
    - 指定したS3パスがmountされる
    - TrainingData置き場
- `/opt/ml/model`
    - SageMakerAPIでoutputとして指定したS3パス
    - 指定したS3パスがmountされる
    - ModelArtifacts置き場
- `/opt/ml/code`
    - TrainingCode(実行コード)置き場
    - Dockerイメージbuild時にCOPYしておく
- `/opt/ml/output`
    - モデル以外で保存したいファイル置き場
    - 指定したS3パスがmountされる
- `/opt/ml/failure`
    - 失敗時のログファイル置き場
    - S3パスがmountされる

上記はTrainingJobでの構成であり、ProcessingJobなどになると少し異なります


### SageMakerでの運用における推し構成

以上を踏まえ、SageMakerで運用するときの構成について考えてみます。
エンドポイントを立てるところまでは考えておらず、バッチ実行の想定です。

- 概観
    - HelperCode（を含めたもの）を`pipeline`、TrainingCodeを`container`としています
    - 運用上でHelperCodeとTrainingCodeでそれぞれDockerイメージを作る必要が出てくると思うので、切り分けた形をとるようにします
    - データやモデル(成果物)はS3をmountするため、リポジトリには含めません
        - コード、テストコード、スクリプトを用意するだけなので、シンプルな構成となります
        - ローカルではSageMakerのローカルモード実行を使います
- `code/container`
    - TrainingCodeの部分
    - `scripts/docker/container`によってDockerイメージを作ります
        - 実行コードである`code/container/${モデル名}`を`/opt/ml/code`にCOPYして配置しておきます
    - カスタムコンテナ実装パターンは`2. 独自のコンテナイメージにSageMakerToolkitをインストールする方法`をとります
        - TrainingCodeのDockerイメージにSageMakerToolkitライブラリをインストールしておきます
    - モデル名でディレクトリを区切ることで、モデルのバージョン管理がしやすくなります
- `code/pipeline`
    - HelperCodeを含めた部分
    - `scripts/docker/pipeline`によってDockerイメージを作ります
    - `batches`配下にバッチのコードを置いておき、実行させていく
    - SageMakerSDKからJobを発行する部分は、`sagemaker_client.py`で行うようにします

```
${リポジトリ名}/
├── code/
│   ├── container/ # TrainingCode
|   │   └── ${モデル名}/
|   │       ├── jobs/
|   │       │   ├── preprocess.py # ProcessingJob
|   │       │   ├── training.py # TrainingJob
|   │       │   ├── evaluation.py # ProcessingJob
|   │       │   └── inference.py # ProcessingJob
|   │       ├── utils/
|   │       ├── poetry.lock
|   │       └── pyproject.toml
│   └── pipeline/ # HelperCode
│       ├── configs/ # 設定ファイル
│       ├── batches/
│       │   ├── collection/ # データ収集処理バッチ
│       │   ├── preprocess/ # 前処理バッチ
│       │   ├── training/ # 学習処理バッチ ※TrainingJob発行
│       │   ├── evaluation/ # 性能評価処理バッチ ※ProcessingJob発行
│       │   ├── inference/ # 推論処理バッチ ※ProcessingJob発行
│       │   └── ...
│       ├── utils/
│       │   ├── aws/
│       │   │   ├── s3_client.py # S3の取り扱い
│       │   │   └── sagemaker_client.py # SageMakerAPIの取り扱い
│       │   ├── db/
│       │   │   ├── enum/
│       │   │   ├── db_adapter.py
│       │   │   └── schema.py # ORMのスキーマ定義
│       │   └── config_reader.py # configファイルの読み込み
│       ├── poetry.lock
│       └── pyproject.toml
├── scripts/
│   ├── docker/ # Dockerスクリプト置き場
│   │   ├── container/
|   |   │   ├── Dockerfile
|   |   │   ├── build.sh
|   |   │   └── push.sh
│   │   └── pipeline/
|   |       ├── Dockerfile
|   |       ├── build.sh
|   |       └── push.sh
│   ├── local_run/ # ローカルで実行するスクリプト
│   │   ├── local_mode.sh # SageMakerのlocalモード実行
│   │   └── pipeline.sh
│   └── test/ # unittestのためコンテナ内に入るスクリプト
│       ├── container.sh
│       └── pipeline.sh
└── tests/ # UnitTest
```

### おわりに

SageMakerでコンテナをカスタマイズし、運用する構成例について書きました。

SageMakerはマネージドにやってくれるのもあり便利な半面、少しカスタマイズしようとすると手こずる部分も大きいなと思います。


### 参考

https://docs.aws.amazon.com/sagemaker/latest/dg/docker-containers.html

https://docs.aws.amazon.com/sagemaker/latest/dg/how-it-works-training.html

https://aws.amazon.com/jp/blogs/news/sagemaker-custom-containers-pattern-training/

https://nsakki55.hatenablog.com/entry/2022/06/11/194809

https://www.slideshare.net/KenichiroNishioka/amazon-sagemaker-252717393
