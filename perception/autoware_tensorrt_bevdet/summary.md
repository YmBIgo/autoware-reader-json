# autoware_tensorrt_bevdet

## 概要

`autoware_tensorrt_bevdet` は、**複数のカメラ画像のみを用いて3次元物体検出（3D Object Detection）を行うBEVDetモデルを、TensorRTで高速に推論するモジュール**です。

BEVDet（Bird's Eye View Detection）は、車両の周囲に配置された複数台のカメラ画像を上空視点（BEV: Bird's Eye View）へ変換し、そのBEV特徴量から車両・歩行者・自転車などの3D物体を検出します。

このモジュールでは、学習済みのBEVDetモデルをTensorRTで実行することで、高速なリアルタイム推論を実現しています。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このモジュールでは主に以下の処理を行います。

- 複数カメラ画像を入力
- 画像をBEV（Bird's Eye View）特徴へ変換
- TensorRTによるBEVDet推論
- 3D物体（位置・サイズ・向き・クラス）の検出
- 検出結果をAutoware標準メッセージとして出力

LiDARは使用せず、**カメラのみで3D物体検出を行う**ことが特徴です。 :contentReference[oaicite:1]{index=1}

---

# 入力

主な入力は以下です。

| Topic | 型 | 内容 |
|-------|----|------|
| `~/input/topic_img_front_left` | `sensor_msgs/msg/Image` | 前左カメラ画像 |
| `~/input/topic_img_front` | `sensor_msgs/msg/Image` | 前方カメラ画像 |
| `~/input/topic_img_front_right` | `sensor_msgs/msg/Image` | 前右カメラ画像 |
| `~/input/topic_img_back_left` | `sensor_msgs/msg/Image` | 後左カメラ画像 |
| `~/input/topic_img_back` | `sensor_msgs/msg/Image` | 後方カメラ画像 |
| `~/input/topic_img_back_right` | `sensor_msgs/msg/Image` | 後右カメラ画像 |
| 各CameraInfo | `sensor_msgs/msg/CameraInfo` | カメラ内部・外部パラメータ |

通常は6台のサラウンドカメラを利用して車両周囲を認識します。 :contentReference[oaicite:2]{index=2}

---

# 出力

| Topic | 型 | 内容 |
|-------|----|------|
| `~/output/boxes` | `DetectedObjects` | 検出した3D物体 |
| `~/output_bboxes` | `MarkerArray` | デバッグ・可視化用マーカー |

検出された物体は、後続のトラッキングや経路計画で利用されます。 :contentReference[oaicite:3]{index=3}

---

# 処理の流れ

```text
Front Left Camera
Front Camera
Front Right Camera
Back Left Camera
Back Camera
Back Right Camera
          │
          ▼
Image Preprocessing
          │
          ▼
BEV Feature Generation
          │
          ▼
TensorRT BEVDet
          │
          ▼
DetectedObjects
```

---

# BEVDetとは

BEVDetは、複数のカメラ画像から**BEV（Bird's Eye View：俯瞰視点）特徴量**を生成し、その特徴量上で3D物体検出を行う手法です。

一般的な画像検出では各カメラ画像ごとに物体を検出しますが、BEVDetでは一度BEV空間へ変換してから認識を行うため、

- カメラ間の情報を統合しやすい
- 3D位置を直接推定できる
- 自動運転との相性が良い

という特徴があります。 :contentReference[oaicite:4]{index=4}

---

# 主な設定項目

代表的な設定項目は以下です。

| パラメータ | 内容 |
|------------|------|
| `precision` | TensorRT推論精度（FP16 / FP32） |
| `onnx_path` | ONNXモデルの保存場所 |
| `engine_path` | TensorRT Engineファイル |
| `debug_mode` | デバッグ表示の有効化 |

TensorRTのFP16推論を利用することで、高速なリアルタイム処理が可能です。 :contentReference[oaicite:5]{index=5}

---

# 他モジュールとの関係

BEVDetは、カメラベース認識パイプラインの前段で利用されます。

```text
Multiple Cameras
        │
        ▼
autoware_tensorrt_bevdet
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

検出された物体は、追跡（Tracking）、予測（Prediction）、経路計画（Planning）へ受け渡されます。 :contentReference[oaicite:6]{index=6}

---

# 関連モジュールとの違い

| モジュール | 特徴 |
|-----------|------|
| **autoware_tensorrt_bevdet** | カメラのみを用いたBEVベースの3D物体検出 |
| **autoware_tensorrt_bevformer** | Transformerと時系列情報を利用したBEV検出 |
| **autoware_bevfusion** | LiDARとカメラを融合したBEVベースの3D物体検出 |

BEVDetは構成が比較的シンプルで高速ですが、LiDARを利用しないため、環境によってはBEVFusionより認識精度が低下する場合があります。一方で、カメラのみで動作できるため、システム構成を簡素化できるという利点があります。 :contentReference[oaicite:7]{index=7}

---

# 制限事項

現在公開されている学習済みモデルは**nuScenesデータセット**で学習されているため、他の環境では十分な性能が得られない場合があります。

実運用では、自車両や対象環境に合わせて再学習したモデルを利用することが推奨されています。 :contentReference[oaicite:8]{index=8}

---

# まとめ

- 複数カメラ画像のみを利用する3D物体検出モジュール
- BEVDetアルゴリズムをTensorRTで高速実行
- Bird's Eye View特徴を生成して3D物体を検出
- `DetectedObjects` を出力し、TrackingやPlanningへ接続
- カメラのみで動作するため構成がシンプルで高速な認識モジュール