# autoware_lidar_centerpoint

## 概要

`autoware_lidar_centerpoint` は、**LiDAR点群から3次元物体を検出するAIベースのPerceptionモジュール**です。

CenterPointと呼ばれる3D物体検出アルゴリズムを採用しており、LiDAR点群から

- 車
- トラック
- バス
- 自転車
- 歩行者

などの物体を高精度に検出します。AutowareではTensorRTを用いて高速に推論を実行できるよう実装されています。 :contentReference[oaicite:0]{index=0}

---

# 役割

このモジュールの役割は、

> **LiDAR点群から3D物体を直接検出すること**

です。

従来のように

- 点群をクラスタリングして
- 物体を推定する

のではなく、ニューラルネットワークが点群全体を解析し、物体の位置や種類を推定します。

```
LiDAR
   │
   ▼
PointCloud
   │
   ▼
CenterPoint
   │
   ├── 車
   ├── 歩行者
   ├── 自転車
   └── バス
```

---

# なぜ必要？

LiDAR点群だけでは、

- 点の集まりが何の物体なのか
- どこまでが1つの物体なのか

を判断するのは容易ではありません。

CenterPointではディープラーニングを利用することで、

- 物体の中心位置
- 大きさ
- 向き
- 種類

を一度に推定できます。

そのため、

- 検出精度の向上
- 遠距離物体の認識
- 複雑な環境での認識性能向上

が期待できます。 :contentReference[oaicite:1]{index=1}

---

# 特徴

このモジュールの特徴は次のとおりです。

- CenterPointアルゴリズムを採用
- PointPillarsベースのネットワークを使用
- TensorRTによる高速推論
- ONNX形式の学習済みモデルを利用
- GPU上でリアルタイム推論が可能

Autowareでは前処理・後処理をROSノード側で実装し、AIモデル本体のみをONNXモデルとして実行します。 :contentReference[oaicite:2]{index=2}

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
前処理
（Voxel化）
    │
    ▼
CenterPoint推論
    │
    ▼
3D Bounding Box生成
    │
    ▼
Detected Objects
```

1. LiDAR点群を入力
2. 点群をVoxel（PointPillars）形式へ変換
3. CenterPointモデルで推論
4. 3Dバウンディングボックスを生成
5. DetectedObjectsとして出力

---

# 入力

主な入力は次のとおりです。

| トピック | 型 | 内容 |
|-----------|----|------|
| `~/input/pointcloud` | `sensor_msgs/msg/PointCloud2` | LiDAR点群 |

入力はLiDAR点群のみです。 :contentReference[oaicite:3]{index=3}

---

# 出力

主な出力は次のとおりです。

| トピック | 内容 |
|-----------|------|
| `~/output/objects` | 検出された3D物体（DetectedObjects） |

出力には、

- 物体クラス
- 位置
- 大きさ
- 向き

などの情報が含まれます。 :contentReference[oaicite:4]{index=4}

---

# Perception内での位置付け

```
LiDAR
    │
    ▼
PointCloud
    │
    ▼
CenterPoint
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

LiDAR点群から3D物体を検出し、TrackingやPredictionへ受け渡す重要な役割を担います。

---

# 学習済みモデル

Autowareでは複数の学習済みモデルが提供されています。

- CenterPoint
- CenterPoint Tiny

これらはONNX形式で提供され、初回起動時にTensorRTエンジンへ変換されます。

学習には主に以下のデータセットが利用されています。

- nuScenes
- Argoverse 2
- TIER IVの内部データセット :contentReference[oaicite:5]{index=5}

---

# Apollo Instance Segmentationとの違い

| CenterPoint | Apollo Instance Segmentation |
|-------------|------------------------------|
| 3D物体検出AI | 点群セグメンテーションAI |
| 3Dバウンディングボックスを出力 | ラベル付き点群クラスタを出力 |
| CenterPointアルゴリズム | Apollo由来アルゴリズム |
| PointPillarsベース | CNNベースの点群セグメンテーション |

どちらもLiDARのみを利用しますが、CenterPointは**物体検出**、Apollo Instance Segmentationは**点群のインスタンス分割**を主目的としています。 :contentReference[oaicite:6]{index=6}

---

# まとめ

- LiDAR点群から3D物体を検出するPerceptionモジュール
- CenterPointアルゴリズムを採用
- PointPillarsベースのネットワークを利用
- TensorRTによる高速推論に対応
- 検出結果をTrackingやPredictionへ受け渡す重要な物体検出モジュール