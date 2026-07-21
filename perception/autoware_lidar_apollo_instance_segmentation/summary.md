# autoware_lidar_apollo_instance_segmentation

## 概要

`autoware_lidar_apollo_instance_segmentation` は、**LiDARの点群から物体を検出・分割（インスタンスセグメンテーション）するPerceptionモジュール**です。

Apollo（Apollo Auto）のCNNベースの物体検出アルゴリズムを利用し、LiDAR点群を解析して、

- 車
- トラック
- 自転車
- 歩行者

などの障害物を検出します。検出された各物体は、それぞれ独立したクラスタとして出力されます。 :contentReference[oaicite:0]{index=0}

---

# 役割

このモジュールの役割は、

> **LiDAR点群を物体ごとに分割し、物体の種類を推定すること**

です。

通常の点群は大量の3D点の集まりですが、このモジュールではCNNを利用して

- どの点が同じ物体なのか
- その物体が何であるか

を推定します。

```
LiDAR
   │
   ▼
PointCloud
   │
   ▼
Apollo Instance Segmentation
   │
   ├── 車
   ├── 歩行者
   ├── 自転車
   └── トラック
```

---

# なぜ必要？

LiDARの生データは単なる点の集合であり、

- どこまでが1台の車なのか
- どれが歩行者なのか

は分かりません。

このモジュールではCNNを利用することで、

- 点群を物体単位に分割
- 物体の種類を推定
- 後続のTrackingやPredictionで扱いやすいデータへ変換

を行います。 :contentReference[oaicite:1]{index=1}

---

# 特徴

このモジュールの特徴は次のとおりです。

- Apollo由来のCNNベース物体検出アルゴリズムを採用
- LiDARのみで物体検出が可能
- 点群をクラスタごとに分類
- 各クラスタにラベル（車・歩行者など）を付与
- 学習済みモデルを利用して推論を実施

なお、このパッケージには**学習機能は含まれておらず**、Apolloが提供する学習済みモデルを使用します。ビルド時に必要なモデルが自動でダウンロードされます。 :contentReference[oaicite:2]{index=2}

---

# 処理の流れ

基本的な処理は以下のようになります。

```
LiDAR
    │
    ▼
PointCloud
    │
    ▼
CNN推論
    │
    ▼
点群セグメンテーション
    │
    ▼
クラスタ生成
    │
    ▼
ラベル付け
    │
    ▼
Detected Objects
```

1. LiDAR点群を入力
2. CNNで各点の特徴を推論
3. 同じ物体に属する点をクラスタ化
4. 物体の種類を推定
5. ラベル付きクラスタとして出力

---

# 入力

主な入力は次のとおりです。

| トピック | 型 | 内容 |
|-----------|----|------|
| `input/pointcloud` | `sensor_msgs/PointCloud2` | LiDAR点群 |

入力はLiDAR点群のみです。 :contentReference[oaicite:3]{index=3}

---

# 出力

主な出力は次のとおりです。

| トピック | 内容 |
|-----------|------|
| `output/labeled_clusters` | ラベル付き点群クラスタ |

出力には、

- 点群クラスタ
- 物体クラス（車・歩行者など）

が含まれ、後続のTrackingモジュールなどで利用されます。 :contentReference[oaicite:4]{index=4}

---

# Perception内での位置付け

```
LiDAR
    │
    ▼
PointCloud
    │
    ▼
Apollo Instance Segmentation
    │
    ▼
Detected Objects
    │
    ▼
Object Tracking
    │
    ▼
Prediction
```

LiDARから取得した点群を物体単位へ変換し、TrackingやPredictionへ渡す役割を担います。

---

# 対応LiDAR

公式ドキュメントでは、以下のVelodyneシリーズ向け学習済みモデルが提供されています。

- Velodyne VLP-16
- Velodyne HDL-64
- Velodyne VLS-128

また、Velodyne 32など他のLiDARでも一定の精度で利用可能とされています。 :contentReference[oaicite:5]{index=5}

---

# まとめ

- LiDAR点群から物体を検出・分割するPerceptionモジュール
- Apollo由来のCNNベースアルゴリズムを採用
- 点群を物体単位のクラスタへ分割し、ラベルを付与
- 学習済みモデルを利用して推論を実施
- TrackingやPredictionへ渡すための重要な前処理モジュール