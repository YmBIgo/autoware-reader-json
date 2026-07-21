# autoware_cluster_merger

## 概要

`autoware_cluster_merger` は、Autoware Universe の **Perception** モジュールに含まれる **点群クラスタ（Point Cloud Clusters）の結合** を行うパッケージです。

LiDARなどから生成された複数のクラスタ群を1つにまとめ、後続の認識処理で扱いやすい形式に変換します。

このパッケージは高度なデータ関連付け（Data Association）は行わず、**複数の入力クラスタを単純に連結（Concatenate）する** シンプルな役割を持っています。 :contentReference[oaicite:0]{index=0}

---

# 役割

Autowareでは、異なるアルゴリズムや異なるLiDARからクラスタが生成されることがあります。

例えば、

- 前方LiDARのクラスタ
- 側方LiDARのクラスタ
- 異なるクラスタリングアルゴリズムの結果

などです。

`autoware_cluster_merger` は、それらを1つのトピックへまとめて、後続ノードが扱いやすいようにします。 :contentReference[oaicite:1]{index=1}

---

# Perceptionパイプラインでの位置付け

おおよそ次のような流れになります。

```text
LiDAR
    │
    ▼
クラスタリング
    │
    ├─────────────┐
    ▼             ▼
Cluster 0     Cluster 1
    │             │
    └──────┬──────┘
           ▼
autoware_cluster_merger
           │
           ▼
Merged Clusters
           │
           ▼
物体認識・Tracking
```

クラスタを統合することで、その後の認識・追跡モジュールは入力元を意識せずに処理できます。

---

# 主な特徴

- 複数のクラスタ入力に対応
- 入力クラスタを単純に結合
- 高速・軽量
- 点群データを変更しない
- ROS 2 ノードとして利用可能

データを加工するというより、「複数の入力をまとめるためのユーティリティノード」と考えると分かりやすいでしょう。 :contentReference[oaicite:2]{index=2}

---

# 入力

入力は2つのクラスタトピックです。

```text
input/cluster0
input/cluster1
```

型は

```text
tier4_perception_msgs/msg/DetectedObjectsWithFeature
```

となっています。 :contentReference[oaicite:3]{index=3}

---

# 出力

出力は

```text
output/clusters
```

で、型は入力と同じ

```text
tier4_perception_msgs/msg/DetectedObjectsWithFeature
```

です。

出力には、2つの入力クラスタがまとめられた結果が含まれます。 :contentReference[oaicite:4]{index=4}

---

# 処理の流れ

内部の処理は非常にシンプルです。

```text
Cluster0
      │
      ├──────┐
      │      │
Cluster1     │
      │      │
      ▼      ▼
クラスタを順番にコピー
        │
        ▼
Merged Clusters
```

データ同士の対応付けや重複除去は行わず、入力クラスタをそのまま連結します。 :contentReference[oaicite:5]{index=5}

---

# パラメータ

代表的なパラメータは以下です。

| パラメータ | 説明 |
|------------|------|
| `output_frame_id` | 出力トピックの `frame_id` を指定します（デフォルトは `base_link`）。 | :contentReference[oaicite:6]{index=6}

---

# 利用シーン

`autoware_cluster_merger` は、例えば次のような場面で利用されます。

- 複数LiDARのクラスタをまとめる
- 複数クラスタリング手法の結果を統合する
- 後続ノードへの入力を一本化する

単体で物体認識を行うパッケージではなく、Perceptionパイプラインのデータ整理を担当する補助的な役割を担っています。

---

# cluster_merger と object_merger の違い

名前が似ていますが、役割は異なります。

| パッケージ | 役割 |
|------------|------|
| **autoware_cluster_merger** | 点群クラスタを単純に結合する |
| **autoware_object_merger** | 検出された物体同士を対応付けし、1つの物体として統合する | :contentReference[oaicite:7]{index=7}

つまり、

- **Cluster Merger** は「データをまとめるだけ」
- **Object Merger** は「同じ物体かどうかを判断して統合する」

という違いがあります。

---

# まとめ

- `autoware_cluster_merger` は **点群クラスタを結合する** Perceptionパッケージ
- 複数のクラスタ入力を1つのトピックへまとめる
- データ関連付けや重複除去は行わず、単純に連結する
- 軽量で高速なデータ統合ノードとして利用される
- 後続の物体認識・Trackingモジュールへ統一されたクラスタデータを提供する :contentReference[oaicite:8]{index=8}