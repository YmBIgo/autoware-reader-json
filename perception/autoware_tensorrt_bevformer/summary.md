# autoware_tensorrt_bevformer 概要

## 概要

`autoware_tensorrt_bevformer` は、**BEVFormer** を TensorRT 上で高速に実行するための Autoware の Perception モジュールです。

BEVFormer は、車両に搭載された複数台のカメラ画像から **Bird's Eye View（BEV：俯瞰視点）** を生成し、その BEV 上で 3D 物体検出を行う深層学習モデルです。

Autoware では、このモデルを **TensorRT に最適化**することで、GPU 上でリアルタイムに近い速度で推論できるようになっています。:contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- 複数カメラ画像を入力する
- TensorRT で BEVFormer を高速推論する
- 車・歩行者・自転車などの3D物体を検出する
- 検出結果を Autoware の DetectedObjects として出力する

LiDAR を使用せず、**カメラのみで周囲の3D環境を認識できる**ことが大きな特徴です。:contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
複数カメラ画像
        │
        ▼
画像特徴抽出
        │
        ▼
BEVFormer
（Transformer）
        │
        ▼
BEV特徴生成
        │
        ▼
3D物体検出
        │
        ▼
DetectedObjects
```

---

# 入力

主に6台のサラウンドカメラ画像を入力します。

例

- Front
- Front Left
- Front Right
- Back
- Back Left
- Back Right

画像は ROS2 の `sensor_msgs/msg/Image` として受信します。:contentReference[oaicite:2]{index=2}

---

# 出力

主な出力は

- `autoware_perception_msgs/msg/DetectedObjects`

です。

検出された物体には

- 種類（車・歩行者など）
- 位置
- 向き
- サイズ
- 信頼度

などの情報が含まれます。:contentReference[oaicite:3]{index=3}

---

# 特徴

- カメラのみで3D物体検出が可能
- Transformer を利用した高精度な認識
- 時系列情報を利用して認識精度を向上
- TensorRT により高速推論
- Autoware の Perception パイプラインへ組み込み可能:contentReference[oaicite:4]{index=4}

---

# TensorRT を利用する理由

BEVFormer は Transformer ベースの大規模モデルであり、そのままでは推論負荷が高くなります。

そこで TensorRT を利用することで

- GPU向け最適化
- FP16 推論
- CUDA カーネル最適化
- 推論速度向上
- メモリ使用量削減

などが実現され、実車で利用しやすくなります。:contentReference[oaicite:5]{index=5}

---

# TensorRT フォルダについて

このパッケージには `TensorRT/` ディレクトリがあり、BEVFormer を TensorRT 上で動作させるための **カスタムプラグイン** が実装されています。

代表例として

- Grid Sampler
- Multi-scale Deformable Attention
- Modulated Deformable Conv2d
- Rotate
- BEV Pool
- Multi-Head Attention

などがあります。

これらは TensorRT 標準では扱えない演算を高速に実行するためのプラグインです。:contentReference[oaicite:6]{index=6}

---

# 利用される場面

`autoware_tensorrt_bevformer` は

- カメラのみの3D物体検出
- 自動運転車の周辺環境認識
- GPU を用いたリアルタイム推論

などで利用されます。

---

# 他モジュールとの関係

```text
Camera
   │
   ▼
autoware_tensorrt_bevformer
   │
   ▼
DetectedObjects
   │
   ▼
Object Tracker
   │
   ▼
Prediction
   │
   ▼
Planning
```

検出した物体は、後続のトラッキング・予測・経路計画モジュールへ渡され、自動運転システム全体で利用されます。

---

# まとめ

`autoware_tensorrt_bevformer` は、BEVFormer を TensorRT に最適化して実行するための Perception モジュールです。

主な役割は、**複数カメラ画像から Bird's Eye View を生成し、3D物体検出を高速に実行すること**です。TensorRT のカスタムプラグインを活用することで、Transformer ベースの高精度な認識モデルを実車で扱える速度まで高速化している点が特徴です。:contentReference[oaicite:7]{index=7}