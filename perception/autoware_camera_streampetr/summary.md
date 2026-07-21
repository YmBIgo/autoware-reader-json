# autoware_camera_streampetr

## 概要

`autoware_camera_streampetr` は、Autoware Universe の **Perception** モジュールに含まれる **カメラ画像のみを用いた3D物体検出** パッケージです。

従来のAutowareでは、3D物体検出はLiDARを利用することが一般的でしたが、このパッケージでは**複数台のカメラ画像だけ**から周囲の車両・歩行者・自転車などの3次元位置を推定できます。TensorRT による高速推論を利用しており、Autoware初のカメラ専用3D物体検出ノードとして実装されています。 :contentReference[oaicite:0]{index=0}

---

# StreamPETRとは

StreamPETR は、複数カメラの画像から3D物体検出を行うディープラーニングモデルです。

特徴は、**時間方向の情報（Temporal Information）** を利用して検出精度を向上させる点です。

通常の画像認識では1枚の画像だけを使って推論しますが、StreamPETRでは

- 前フレーム
- 現在のフレーム

の情報を利用することで、

- 遠くの物体
- 一時的に見えにくくなった物体
- 動いている物体

をより安定して検出できます。 :contentReference[oaicite:1]{index=1}

---

# Autowareでの役割

Perception パイプラインでは、おおよそ次のような流れになります。

```text
複数台のカメラ画像
        │
        ▼
autoware_camera_streampetr
        │
        ▼
3D物体検出
（車・歩行者・自転車など）
        │
        ▼
Planningなど後続モジュール
```

LiDARを使用せず、カメラだけで3D認識を行うため、カメラ主体の自動運転システムや研究用途でも利用できます。 :contentReference[oaicite:2]{index=2}

---

# 主な特徴

- カメラ画像のみで3D物体検出
- TensorRTによる高速推論
- 複数カメラに対応
- 時系列情報を利用した安定した認識
- ROS 2 ノードとしてAutowareへ統合

特に、複数カメラの画像が**順番に到着する環境**でも効率よく処理できるよう最適化されています。 :contentReference[oaicite:3]{index=3}

---

# 入力

主な入力は以下です。

- 各カメラの画像
  - `sensor_msgs/msg/Image`
  - `sensor_msgs/msg/CompressedImage`
- 各カメラのキャリブレーション情報
  - `sensor_msgs/msg/CameraInfo`

複数台（通常6台程度）のカメラ入力を扱うことを想定しています。 :contentReference[oaicite:4]{index=4}

---

# 出力

出力は

```text
DetectedObjects
```

です。

検出された物体には、

- 車両
- 歩行者
- 自転車
- バイク

などのクラス情報に加え、

- 3次元位置
- 向き
- サイズ

などの情報が含まれます。 :contentReference[oaicite:5]{index=5}

---

# 処理の流れ

内部では大まかに次のような流れで処理が行われます。

```text
画像入力
      │
      ▼
画像前処理
      │
      ▼
TensorRT推論
（StreamPETR）
      │
      ▼
後処理
(NMS・信頼度フィルタ)
      │
      ▼
DetectedObjects
```

推論モデルは大きく

- Backbone
- Position Embedding
- Detection Head

に分かれて構成されています。 :contentReference[oaicite:6]{index=6}

---

# 前提条件

このノードを利用するためには、

- カメラ同士の時刻同期
- 正しいカメラキャリブレーション
- TF（座標変換）の設定
- 歪み補正済み画像

などが必要になります。 :contentReference[oaicite:7]{index=7}

---

# 利用シーン

`autoware_camera_streampetr` は、

- カメラ主体の自動運転システム
- LiDARを搭載しない車両
- LiDARと組み合わせたセンサフュージョン
- 研究・実験用途

などで利用されます。

検出された3D物体は、その後の

- Tracking
- Prediction
- Planning
- 障害物回避

などのモジュールで活用されます。

---

# まとめ

- `autoware_camera_streampetr` は **カメラ画像のみで3D物体検出** を行うPerceptionパッケージ
- StreamPETRをTensorRTで高速実行するノード
- 複数カメラの画像から車両・歩行者・自転車などを3Dで認識する
- 時系列情報を利用することで、安定した検出性能を実現
- 検出結果はAutowareのTrackingやPlanningなど後続モジュールで利用される :contentReference[oaicite:8]{index=8}