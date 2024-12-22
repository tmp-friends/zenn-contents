---
title: "Embedding を可視化して NN の学習結果を定性的に評価する"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "機械学習", "scikitlearn"]
published: true
publication_name: "dmmdata"
---

## はじめに

Embedding を可視化することで NN の学習結果を定性的に評価することができます。
この記事では、実際の活用事例とともに紹介します。

## 背景

Two-tower レコメンドモデル [^1] において、日付情報の Embedding (= DayEmbedding) を導入したい。
この DayEmbedding が上手く学習できているかを確認したい。

## 実装

scikit-learn の TSNE クラス [^2] を用いることで、簡単にn次元の情報を2次元へ射影することができます。
その後、射影した情報を2次元座標へ plot します。

具体的な実装は以下になります。

```py
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
```

```py:t-SNE による次元削減
# day_embedding: np.ndarray
tsne = TSNE(n_components=2, random_state=42)
day_embedding_2d = tsne.fit_transform(day_embedding)
```

```py:plot
plt.figure(figsize=(12, 8))
plt.scatter(
    day_embedding_2d[:, 0],
    day_embedding_2d[:, 1],
    c=np.arange(day_embedding.shape[0]),
    cmap='viridis'
)
plt.colorbar(label='Day Index')
plt.title('t-SNE Visualization of Day Embeddings')
plt.xlabel('t-SNE Component 1')
plt.ylabel('t-SNE Component 2')
plt.show()
```

## 出力結果

Day index (=日付) が右から左へのグラデーションとなっており、学習が上手くいってそうなことがわかります。

![result](/images/visualize-embedding/result.png)

一方で、下図は上図に比べてあまり学習が上手くいっていない例です。最近の日付に対してなんらか偏っていそうなことがわかります。

![result-failure](/images/visualize-embedding/result-failure.png)

## おわりに

scikit-learn の TSNE クラスを用いることでサクッと次元削減ができ、学習結果の定性評価に活用することができました。
今後の展望として、ItemEmbedding に対しカテゴリーで色分けしたり、下図のようにサムネイル画像として可視化できると便利かと思いました。

![item-embedding](/images/visualize-embedding/item-embedding.png)*画像引用元: Graph Convolutional Neural Networks for Web-Scale Recommender Systems[^3], Figure 6*

## 参考

https://qiita.com/g-k/items/120f1cf85ff2ceae4aba

https://lvdmaaten.github.io/publications/papers/JMLR_2008.pdf

[^1]: [Scaling deep retrieval with TensorFlow Recommenders and Vertex AI Matching Engine](https://cloud.google.com/blog/products/ai-machine-learning/scaling-deep-retrieval-tensorflow-two-towers-architecture?hl=en)
[^2]: [scikit-learn API Reference > TSNE](https://scikit-learn.org/1.5/modules/generated/sklearn.manifold.TSNE.html)
[^3]: [Graph Convolutional Neural Networks for Web-Scale Recommender Systems](https://arxiv.org/abs/1806.01973)
