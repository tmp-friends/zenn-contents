---
title: "よく使う(主観)！TensorFlowの組み込み関数"
emoji: "🔢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "TensorFlow", "機械学習"]
published: false
publication_name: "dmmdata"
---

公式Docのリンクを貼っているので、詳しくはそちらから

## TensorFlow modules
### tf.where

https://www.tensorflow.org/api_docs/python/tf/where

- 概要
  - Condition配列の真偽をチェックし、Trueならx、Falseならyの値を用いる
- 引数
  - Condition
  - x
  - y

e.g. RaggendTensorであるinputsに対し、要素が0(=True)ならxのself.no_member_varを返し、要素が0でない(=False)ならそのままresを返す

```py
res = tf.where(
    condition=tf.expand_dims(inputs.row_lengths() == 0, axis=1),
    x=self.no_member_var,
    y=res,
)
```

### tf.gather

https://www.tensorflow.org/api_docs/python/tf/gather

- 概要
  - paramsの配列からindicesの配列に対応するインデックスの値を引っ張ってくる
- 引数
  - params
  - indices

![/images/tensorflow-modules/gather.png](/images/tensorflow-modules/gather.png)

### tf.einsum

https://www.tensorflow.org/api_docs/python/tf/einsum

- 概要
  - Einsteinの縮約記法を利用して、テンソルの要素ごとの操作を効率的に計算

e.g. tensor1の各要素とtensor2の対応する行の積([n]と[n, d]のテンソル同士の積)

```py
tf.einsum("n,nd->nd", tensor1, tensor2)
```

### tf.math.maximum

https://www.tensorflow.org/api_docs/python/tf/math/maximum

- 概要
  - xとyの要素ごとに大きい方を採用する
- 引数
  - x
  - y

e.g.

```py
x = tf.constant([0., 0., 0., 0.])
y = tf.constant([1., 3., 1., -1.])

tf.math.maximum(x, y)
```

```
tf.Tensor([1. 3. 1. 0.], shape=(4,), dtype=float32)
```

### tf.argsort

https://www.tensorflow.org/api_docs/python/tf/argsort

- 概要
  - sortした際のidxを返す
  - デフォルトは、order by asc

e.g.

```py
v = [1e4, 1e2, 1e10, 1e3]

tf.argsort(v)
```

```
<tf.Tensor: shape=(4,), dtype=int32, numpy=array([1, 3, 0, 2], dtype=int32)>
```

### tf.searchsorted

TODO:

https://www.tensorflow.org/api_docs/python/tf/searchsorted

- 概要
  - valuesがsorted_sequenceに入れるとしたらどこに入るかのidxを返す
- 引数
  - sorted_sequence
  - values

e.g.

```py
```

### tf.tensor_scatter_nd_add

https://www.tensorflow.org/api_docs/python/tf/tensor_scatter_nd_add

- 概要
  - tensorに対して、indicesに対応するupdatesの値を加算
- 引数
  - tensor
  - indices
  - updates

e.g.

```py
tensor = tf.ones([5], dtype=tf.int32)
indices = tf.constant([[1, 3]]) # 変更を加えたいtensor変数のindex
updates = tf.constant([100, 10]) # 加算したい値

updated = tf.tensor_scatter_nd_add(tensor, indices, updates)
```

```
<tf.Tensor: shape=(5,), dtype=int32, numpy=array([  1, 101,   1,  11,   1], dtype=int32)>
```

### tf.Variable

https://www.tensorflow.org/api_docs/python/tf/Variable

- 概要
  - Tensor型の変数のコンストラクタ

### tf.keras.initializers.GlorotUniform

https://www.tensorflow.org/api_docs/python/tf/keras/initializers/GlorotUniform

- tf.keras.initializers: 変数の初期化モジュール
- GlorotUniform: Glorotの一様分布
    - 原点周りにおける線形な活性化関数
    - nnのレイヤーの重み初期化で使われることが多い

e.g.

```py
tf.Variable(tf.keras.initializers.GlorotUniform()(shape=(1, 32)))
```

```
<tf.Variable 'Variable:0' shape=(1, 32) dtype=float32, numpy=
array([[ 0.14041537,  0.40557677, -0.15692988,  0.10684782,  0.24010688,
        -0.3836745 , -0.08835864, -0.37656   , -0.26274264,  0.09739405,
         0.1863839 ,  0.33400756,  0.38333488,  0.06080905, -0.40710193,
         0.17942119, -0.328488  ,  0.2535221 ,  0.1820457 , -0.41341117,
        -0.19261914,  0.24314821, -0.13418898,  0.08893508, -0.21338432,
        -0.2051562 ,  0.08110768,  0.33076704,  0.26542288,  0.22211951,
         0.2835554 , -0.03046271]], dtype=float32)>
```

**参考**

- https://nykergoto.hatenablog.jp/entry/2017/10/15/%E3%83%8B%E3%83%A5%E3%83%BC%E3%83%A9%E3%83%AB%E3%83%8D%E3%83%83%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E5%A4%89%E6%95%B0%E3%81%AE%E5%88%9D%E6%9C%9F%E5%8C%96%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6

### tf.keras.layers.Layer

https://www.tensorflow.org/api_docs/python/tf/keras/layers/Layer


## TensorFlow Text

https://www.tensorflow.org/text?hl=ja

- 概要
  - TensorFlowのテキスト処理ライブラリ
  - Tokenizerが用意されている

e.g.

```py
import tensorflow_text as tf_text

tf_text.WhitespaceTokenizer().tokenize("Attention is all you need.")
```

```
<tf.Tensor: shape=(5,), dtype=string, numpy=array([b'Attention', b'is', b'all', b'you', b'need.'], dtype=object)>
```
