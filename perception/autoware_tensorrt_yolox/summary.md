# autoware_tensorrt_yolox 概要

## 概要

`autoware_tensorrt_yolox` は、**YOLOX を TensorRT 上で高速に実行するための Autoware の Perception モジュール**です。

カメラ画像から車両・歩行者・自転車などの物体を検出するほか、利用するモデルによっては**セマンティックセグメンテーション**や**信号機検出**にも対応しています。TensorRT を利用することで、GPU 上で高速な推論を実現しています。 :contentReference[oaicite:0]{index=0}

---

# 主な役割

このパッケージの役割は以下のとおりです。

- カメラ画像から物体を検出する
- TensorRT による高速推論を行う
- 2Dバウンディングボックスを生成する
- 検出結果を後続の認識モジュールへ渡す
- モデルによってはセマンティックセグメンテーションや信号機検出も行う :contentReference[oaicite:1]{index=1}

---

# 処理の流れ

```text
カメラ画像
      │
      ▼
画像前処理
      │
      ▼
YOLOX
(TensorRT)
      │
      ▼
後処理
(NMSなど)
      │
      ▼
2D物体検出結果
```

---

# 入力

主な入力はカメラ画像です。

| 入力 | 型 |
|------|----|
| カメラ画像 | `sensor_msgs/msg/Image` |

YOLOX は画像1枚ごとに推論を行います。 :contentReference[oaicite:2]{index=2}

---

# 出力

モデルによって出力内容が異なりますが、主な出力は以下のとおりです。

- `DetectedObjectsWithFeature`
- 検出結果を描画した画像
- セマンティックセグメンテーションマスク（対応モデルのみ）
- カラー化したセグメンテーション画像（対応モデルのみ） :contentReference[oaicite:3]{index=3}

---

# 検出できる物体

標準的なモデルでは以下のような物体を検出できます。

- 車
- トラック
- バス
- 歩行者
- 自転車
- バイク

また、交通信号機検出用のモデルへ切り替えることで、

- 車両用信号
- 歩行者用信号

の検出にも利用できます。 :contentReference[oaicite:4]{index=4}

---

# 特徴

- YOLOX ベースの高速物体検出
- TensorRT によるGPU最適化
- FP32・FP16・INT8推論に対応
- 前処理をGPUで実行可能
- セグメンテーションや信号機検出にも対応（使用モデルによる） :contentReference[oaicite:5]{index=5}

---

# TensorRT を利用する理由

YOLOX はリアルタイム性に優れた物体検出モデルですが、自動運転ではさらに高速な処理が求められます。

TensorRT を利用することで、

- GPU向け最適化
- FP16・INT8 推論
- 推論時間の短縮
- メモリ使用量の削減

などが可能となり、高いフレームレートで物体検出を実行できます。 :contentReference[oaicite:6]{index=6}

---

# TensorRT エンジン

このパッケージでは、指定した ONNX モデルを初回起動時に TensorRT エンジンへ変換します。

生成された `.engine` ファイルは保存され、次回以降は再利用されるため、起動時間を短縮できます。 :contentReference[oaicite:7]{index=7}

---

# 他モジュールとの関係

```text
Camera
   │
   ▼
autoware_tensorrt_yolox
   │
   ▼
DetectedObjectsWithFeature
   │
   ▼
2D-3D Fusion
   │
   ▼
Object Tracker
   │
   ▼
Prediction
```

カメラ画像から検出した物体は、後続の物体融合・追跡・予測モジュールで利用されます。

---

# ノード構成

このパッケージでは主に以下のようなノードが利用されます。

- `tensorrt_yolox_node`
  - 実際に YOLOX の推論を行うメインノード
- `yolox_single_image_inference_node`
  - 単一画像での推論や動作確認向けのノード

通常の Autoware の認識パイプラインでは、`tensorrt_yolox_node` が中心となって利用されます。

---

# 利用される場面

`autoware_tensorrt_yolox` は、

- カメラ画像からの2D物体検出
- 信号機検出
- セマンティックセグメンテーション（対応モデル）
- LiDARとの認識融合の前段処理

などで利用されます。

---

# まとめ

`autoware_tensorrt_yolox` は、**YOLOX を TensorRT 上で高速に実行するための物体検出モジュール**です。

主な役割は、**カメラ画像から車両・歩行者・自転車などをリアルタイムで検出し、その結果を後続の認識モジュールへ提供すること**です。また、使用するモデルによってはセマンティックセグメンテーションや信号機検出にも対応しており、Autoware のカメラ認識を支える重要なコンポーネントとなっています。 :contentReference[oaicite:8]{index=8}