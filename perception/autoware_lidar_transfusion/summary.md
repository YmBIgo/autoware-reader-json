# autoware_lidar_transfusion

## 概要

`autoware_lidar_transfusion` は、LiDAR点群から車両・歩行者・自転車などの3次元物体を検出するためのパッケージです。

Autowareの認識（Perception）モジュールにおいて、LiDARのみを用いた3D物体検出を担当し、検出した物体の位置・大きさ・向き・種類を後続の追跡（Tracking）や経路計画（Planning）へ渡します。

内部では **TransFusion** をベースとしたニューラルネットワークを使用し、推論には **TensorRT** を利用することでGPU上で高速に実行できるようになっています。:contentReference[oaicite:0]{index=0}

---

# TransFusionとは

TransFusionは、Transformerを利用した3D物体検出アルゴリズムです。

元々はLiDARとカメラ画像を融合する手法として提案されましたが、AutowareではLiDARのみを入力として利用する構成が採用されています。

特徴として、

- 点群全体から物体候補を推定する
- Transformerによって物体同士の関係を学習できる
- 車両・歩行者・自転車などを高精度に検出できる

といった利点があります。:contentReference[oaicite:1]{index=1}

---

# 主な役割

`autoware_lidar_transfusion` の役割は次のとおりです。

- LiDAR点群をニューラルネットワーク入力へ変換
- TensorRTによる高速推論
- 3D物体検出
- 検出物体のクラス分類
- 検出結果のNMS（重複除去）
- ROSメッセージとしてDetectedObjectsを出力

---

# 処理の流れ

```text
LiDAR点群
      │
      ▼
前処理
（Voxel化）
      │
      ▼
TransFusion推論
      │
      ▼
物体候補生成
      │
      ▼
NMS
      │
      ▼
DetectedObjects出力
```

---

# 1. 点群入力

ROS2の

```
sensor_msgs/msg/PointCloud2
```

を受信します。

最低限必要なフィールドは

- x
- y
- z
- intensity

です。:contentReference[oaicite:2]{index=2}

---

# 2. 前処理

入力点群をニューラルネットワークが扱いやすいようにVoxel化します。

Voxelとは、

> 空間を小さな立方体に区切ったもの

であり、

```
□□□□□□□□
□□□□□□□□
□□□□□□□□
```

のように3次元空間を格子状に分割して特徴量を計算します。

---

# 3. TensorRTによる推論

前処理後のデータをTensorRTで実行します。

推論では

- 車
- トラック
- バス
- 自転車
- 歩行者

などの物体を検出します。:contentReference[oaicite:3]{index=3}

---

# 4. 後処理

推論結果には重複した物体候補が含まれるため、

- Score Threshold
- NMS（Non-Maximum Suppression）

によって不要な候補を除去します。:contentReference[oaicite:4]{index=4}

---

# ROS2ノード

主要ノードは

```
lidar_transfusion_node
```

です。

Launchファイル

```
launch/lidar_transfusion.launch.xml
```

から起動します。:contentReference[oaicite:5]{index=5}

---

# 入力

| Topic | 型 | 内容 |
|-------|----|------|
| `~/input/pointcloud` | `sensor_msgs/msg/PointCloud2` | LiDAR点群 |

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `~/output/objects` | `autoware_perception_msgs/msg/DetectedObjects` | 検出した3D物体 |

またデバッグ用として

- preprocess時間
- inference時間
- postprocess時間
- pipeline latency
- cyclic time

なども出力します。:contentReference[oaicite:6]{index=6}

---

# TensorRTエンジン生成

ONNXモデルからTensorRTエンジンを生成できます。

```bash
ros2 launch autoware_lidar_transfusion lidar_transfusion.launch.xml build_only:=true
```

生成後はTensorRT Engineを利用して高速推論を行います。:contentReference[oaicite:7]{index=7}

---

# 学習済みモデル

Autowareでは

- ONNX形式の学習済みモデル

を利用します。

モデルはmmdetection3dを用いて学習されており、TensorRTへ変換して推論を実行します。:contentReference[oaicite:8]{index=8}

---

# ディレクトリ構成

```text
autoware_lidar_transfusion/
├── config/         # パラメータ
├── include/        # ヘッダ
├── launch/         # Launchファイル
├── lib/            # TensorRT・CUDA関連
├── schema/         # パラメータスキーマ
├── src/            # ノード実装
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

# Autoware内での位置付け

Perception全体で見ると、

```text
LiDAR
   │
   ▼
autoware_lidar_transfusion
   │
   ▼
DetectedObjects
   │
   ▼
Multi Object Tracker
   │
   ▼
Prediction
   │
   ▼
Planning
```

という流れになります。

物体検出を担当する中心的なモジュールの一つです。

---

# まとめ

`autoware_lidar_transfusion` は、LiDAR点群から3D物体を検出するためのパッケージです。

TensorRTによる高速推論とTransFusionアルゴリズムを組み合わせることで、高精度かつリアルタイムな3D物体検出を実現しています。

Autowareでは、LiDARベースの認識モジュールとして、車両・歩行者・自転車などを検出し、その結果を後続の追跡・予測・経路計画へ受け渡す重要な役割を担っています。:contentReference[oaicite:9]{index=9}

---

# 参考

- Autoware Universe Documentation（autoware_lidar_transfusion）:contentReference[oaicite:10]{index=10}
- TransFusion: Robust LiDAR-Camera Fusion for 3D Object Detection with Transformers
- https://github.com/open-mmlab/mmdetection3d