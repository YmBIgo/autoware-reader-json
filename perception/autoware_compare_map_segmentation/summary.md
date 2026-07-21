# autoware_compare_map_segmentation

## 概要

`autoware_compare_map_segmentation` は、Autoware Universe の **Perception** モジュールに含まれる **地図を利用した点群フィルタリング** パッケージです。

LiDARから取得した点群を、あらかじめ作成されたHDマップ（点群地図やElevation Map）と比較し、**地図上に存在する静的な物体を除去**します。

これにより、車両・歩行者などの**動的物体だけを抽出**しやすくなり、後続の物体検出や追跡の精度向上につながります。 :contentReference[oaicite:0]{index=0}

---

# 役割

LiDARで取得した点群には、

- 路面
- 建物
- ガードレール
- 標識
- 縁石

など、地図にも登録されている静的な構造物が多数含まれます。

これらは障害物検出には不要な場合が多いため、`autoware_compare_map_segmentation` はHDマップと比較し、

- 地図と一致する点
- 地図に存在しない点

を判定して、静的な点群を除去します。 :contentReference[oaicite:1]{index=1}

---

# Perceptionパイプラインでの位置付け

Perceptionでは、おおよそ次のような流れになります。

```text
LiDAR点群
      │
      ▼
autoware_compare_map_segmentation
      │
      ▼
静的物体を除去
      │
      ▼
動的物体のみの点群
      │
      ▼
クラスタリング
      │
      ▼
物体認識・Tracking
```

このパッケージは、物体検出そのものではなく、**前処理（Preprocessing）** を担当します。

---

# 主な特徴

- HDマップを利用して静的物体を除去
- 動的物体を抽出しやすくする
- 複数のフィルタ方式を提供
- PointCloud2をそのまま出力
- ROS 2ノードとして利用可能

地図を利用することで、建物や道路設備などを毎回検出対象にしなくて済み、後続処理の負荷や誤検出を減らすことができます。 :contentReference[oaicite:2]{index=2}

---

# 主なフィルタ

`autoware_compare_map_segmentation` には、用途に応じて複数の比較方法が用意されています。

## Distance Based Compare Map Filter

最もよく利用されるフィルタです。

入力点群とHDマップの点群を比較し、

- 地図上の点に十分近い点
- 地図と一致すると判断できる点

を除去します。

KD-Treeによる近傍探索を利用するため、高速に比較できます。 :contentReference[oaicite:3]{index=3}

---

## Compare Elevation Map Filter

Elevation Map（高さ地図）を利用するフィルタです。

各点の高さをElevation Mapと比較し、

高さの差が小さい点を除去します。

路面付近の点を効率よく取り除く用途に適しています。 :contentReference[oaicite:4]{index=4}

---

## Voxel Based Compare Map Filter

点群をVoxel（小さな立方体）へ分割して比較します。

Voxel単位で処理するため、

- 大規模な点群
- 広い地図

でも比較的高速に処理できます。 :contentReference[oaicite:5]{index=5}

---

# 入力

フィルタによって多少異なりますが、代表的な入力は以下です。

- LiDAR点群
  - `sensor_msgs/msg/PointCloud2`
- HD PointCloud Map
- Elevation Map（フィルタによる）
- Lanelet情報（フィルタによる）

使用するフィルタによって必要な地図情報が異なります。 :contentReference[oaicite:6]{index=6}

---

# 出力

出力は

```text
sensor_msgs/msg/PointCloud2
```

です。

地図上に存在する静的点群が除去され、動的物体を中心とした点群が出力されます。 :contentReference[oaicite:7]{index=7}

---

# 処理の流れ

内部では概ね次のような処理を行います。

```text
LiDAR点群
      │
      ▼
HDマップ読み込み
      │
      ▼
各点を比較
      │
      ▼
地図と一致する点を除去
      │
      ▼
残った点群を出力
```

フィルタ方式によって比較方法は異なりますが、「地図に存在する静的な点を取り除く」という目的は共通しています。 :contentReference[oaicite:8]{index=8}

---

# 利用シーン

`autoware_compare_map_segmentation` は、

- 建物を除去したい
- 路面を除去したい
- 静的障害物を除去したい
- 動的物体だけを検出したい

といった場面で利用されます。

後続の

- クラスタリング
- 物体検出
- Tracking

などの入力として使用されます。

---

# 他のセグメンテーションとの違い

| パッケージ | 主な役割 |
|------------|----------|
| **autoware_compare_map_segmentation** | HDマップを利用して静的物体を除去 |
| **autoware_ground_segmentation** | 点群形状から地面を推定して除去 |
| **autoware_euclidean_cluster** | 点群をクラスタへ分割 |
| **autoware_detected_object_validation** | 検出結果の妥当性を確認 |

`autoware_compare_map_segmentation` の特徴は、**センサデータだけではなくHDマップを利用してフィルタリングすること**です。 :contentReference[oaicite:9]{index=9}

---

# まとめ

- `autoware_compare_map_segmentation` は **HDマップを利用した点群フィルタリング** パッケージ
- LiDAR点群と地図を比較し、建物・ガードレール・路面などの静的物体を除去する
- Distance Based、Elevation Map、Voxel Basedなど複数の比較手法を提供
- 動的物体だけを抽出しやすくし、後続のクラスタリングや物体認識の精度向上に貢献する
- Perceptionパイプラインでは、物体検出前の重要な前処理として利用される :contentReference[oaicite:10]{index=10}