# autoware_euclidean_cluster

## 概要

`autoware_euclidean_cluster` は、LiDARなどから取得した点群を**物体ごとのグループ（クラスタ）**に分割するためのパッケージです。

物体検出の前処理として利用されることが多く、「どの点群が同じ物体に属しているか」を距離情報を利用して判定します。

Autowareでは、このクラスタから物体の大きさや位置を推定し、後続の追跡（Tracking）や経路計画（Planning）へ渡します。 :contentReference[oaicite:1]{index=1}

---

## 主な役割

LiDARが取得する点群は、周囲のすべての物体が混ざった状態になっています。

例えば、

- 車両
- 歩行者
- 自転車
- ガードレール
- 標識

などの点が一つの点群として入力されます。

`autoware_euclidean_cluster` は、近くに存在する点同士をまとめることで、

「この点群は1台の車」

「こちらは歩行者」

というように、物体単位へ分割します。 :contentReference[oaicite:2]{index=2}

---

## どのような処理を行うのか

基本的な流れは次のようになります。

```text
入力点群
    │
    ▼
点同士の距離を計算
    │
    ▼
近い点をグループ化
    │
    ▼
クラスタ生成
    │
    ▼
DetectedObjectsWithFeature
```

各クラスタが、1つの物体候補になります。 :contentReference[oaicite:3]{index=3}

---

## 利用できるクラスタリング方式

このパッケージでは、主に3種類の方式が利用できます。

### euclidean_cluster

最も基本的なクラスタリング方法です。

点同士のユークリッド距離（直線距離）を利用してクラスタを作成します。

近くにある点は同じ物体として扱われ、一定距離以上離れた点は別の物体として分割されます。

シンプルで分かりやすく、多くの場面で利用されます。 :contentReference[oaicite:4]{index=4}

---

### voxel_grid_based_euclidean_cluster

大量の点群を高速に処理するための方式です。

最初に点群をVoxel（小さな立方体）へまとめ、その重心だけを使ってクラスタリングを行います。

処理の流れは次のようになります。

```text
点群
   │
Voxel化
   │
Voxel重心を計算
   │
Euclidean Clustering
   │
元の点群へ戻す
```

点数が多い環境でも計算量を削減できるため、大規模な点群処理に適しています。 :contentReference[oaicite:5]{index=5}

---

### label_based_euclidean_cluster

Semantic Segmentation によってラベル付けされた点群を対象としたクラスタリングです。

例えば、

- 車
- 歩行者
- 自転車

などのラベルごとにクラスタリングを行うため、異なる種類の物体が混ざりにくくなります。

近年のAIベースの点群認識と組み合わせて利用されます。 :contentReference[oaicite:6]{index=6}

---

## 入出力

### 入力

- `input`
  - `sensor_msgs/msg/PointCloud2`
  - 地面除去後などの点群

### 出力

- `output`
  - `DetectedObjectsWithFeature`
  - クラスタリングされた物体

- `debug/clusters`
  - 色分けされたクラスタ点群（デバッグ・可視化用） :contentReference[oaicite:7]{index=7}

---

## 主なパラメータ

代表的な設定項目は次のとおりです。

| パラメータ | 内容 |
|------------|------|
| `tolerance_m` | 同じクラスタとみなす距離 |
| `min_points_per_cluster` | 最小点数 |
| `max_cluster_size` | 最大点数 |
| `use_height` | 高さ方向（Z）もクラスタリングに利用するか |

これらを調整することで、小さすぎるノイズや大きすぎるクラスタを除外できます。 :contentReference[oaicite:8]{index=8}

---

## Autoware内での位置づけ

このパッケージは、LiDAR認識の初期段階で利用されます。

```text
LiDAR点群
     │
     ▼
Ground Segmentation
     │
     ▼
Euclidean Cluster
     │
     ▼
Shape Estimation
     │
     ▼
Object Tracking
     │
     ▼
Prediction
```

クラスタリングによって「物体候補」を作成し、その後の形状推定や追跡処理へつなげます。

---

## 利用するメリット

このパッケージを利用することで、

- 点群を物体単位へ分割できる
- LiDARのみで物体候補を生成できる
- 高速なVoxel版クラスタリングを利用できる
- AIでラベル付けされた点群にも対応できる
- 後段の物体認識や追跡処理の入力を生成できる

といったメリットがあります。

---

## まとめ

`autoware_euclidean_cluster` は、LiDAR点群を距離に基づいて物体ごとに分割するためのパッケージです。

用途に応じて、

- **euclidean_cluster**（基本方式）
- **voxel_grid_based_euclidean_cluster**（高速版）
- **label_based_euclidean_cluster**（ラベル付き点群向け）

の3種類のクラスタリング方式を利用できます。

AutowareのLiDAR認識パイプラインにおいて、物体候補を生成する重要な役割を担っています。

---

## 参考

- Autoware Universe Documentation
  - https://autowarefoundation.github.io/autoware_universe/main/perception/autoware_euclidean_cluster/
- Autoware Universe
  - https://github.com/autowarefoundation/autoware_universe/tree/main/perception/autoware_euclidean_cluster